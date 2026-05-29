---
layout: post
title: "MCP OAuth with Cognito behind CloudFront"
date: 2026-05-29
description: How to get Claude Desktop connecting to a private MCP server via OAuth when Cognito doesn't support DCR and Lambda has no internet egress.
---

Getting Claude Desktop to authenticate against a private MCP server on AWS took longer than expected. This covers what actually works.

## The constraints

- MCP server runs on Lambda behind API Gateway and CloudFront
- Auth is Cognito (user pool, hosted UI)
- Lambda is in a private VPC with no NAT gateway
- `mcp-remote` is the only way to connect Claude Desktop to a remote HTTP MCP server

## What `mcp-remote` expects

`mcp-remote` follows the MCP OAuth spec strictly:

1. Hits `/mcp/` → gets 401 with `WWW-Authenticate: Bearer resource_metadata=...`
2. Fetches the resource metadata → finds the authorization server
3. Fetches `/.well-known/oauth-authorization-server` → finds OAuth endpoints
4. POSTs to `/register` (DCR) → gets client credentials
5. Opens browser to `/authorize` → user authenticates
6. Exchanges code at `/token` → gets access token
7. Uses token for all subsequent requests

Two problems: Cognito doesn't support DCR, and Claude Desktop ignores external `authorization_endpoint`/`token_endpoint` from discovery metadata and insists on using the MCP server's own domain for all OAuth endpoints.

## The fix: OAuth proxy on your own domain

Make your domain act as the Authorization Server, proxying to Cognito behind the scenes.

**FastAPI routes:**

```python
@app.get("/.well-known/oauth-authorization-server")
async def oauth_as_metadata():
    return {
        "issuer": "https://your-domain.com",
        "authorization_endpoint": "https://your-domain.com/authorize",
        "token_endpoint": "https://your-domain.com/token",
        "registration_endpoint": "https://your-domain.com/register",
        "token_endpoint_auth_methods_supported": ["client_secret_post", "none"],
        ...
    }

@app.get("/authorize")
async def authorize(request: Request):
    # Redirect to Cognito hosted UI with all params forwarded
    return RedirectResponse(f"{cognito_domain}/oauth2/authorize?{request.url.query}")

@app.post("/register")
async def register(request: Request):
    # Return pre-configured Cognito app client credentials
    # No dynamic client creation — same credentials for all registrations
    return JSONResponse({"client_id": ..., "client_secret": ..., ...}, status_code=201)
```

**`/.well-known/oauth-protected-resource/mcp`** must list only your domain in `authorization_servers` — not the Cognito issuer URL. If Cognito's URL appears there, `mcp-remote` will try DCR against Cognito, fail, and exit.

## The `/token` problem

Lambda has no outbound internet. The `/token` handler can't proxy to `your-pool.auth.region.amazoncognito.com/oauth2/token`. The solution is to bypass Lambda entirely for this route.

Add Cognito's hosted UI as a CloudFront origin with an origin path of `/oauth2`:

```hcl
origin {
  origin_id   = "cognito-hosted-ui"
  domain_name = "${cognito_domain}.auth.${region}.amazoncognito.com"
  origin_path = "/oauth2"

  custom_origin_config {
    https_port             = 443
    origin_protocol_policy = "https-only"
    origin_ssl_protocols   = ["TLSv1.2"]
  }
}
```

Route `/token` to that origin instead of API Gateway:

```hcl
ordered_cache_behavior {
  path_pattern     = "/token"
  target_origin_id = "cognito-hosted-ui"
  allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
  cache_policy_id  = data.aws_cloudfront_cache_policy.caching_disabled.id
  ...
}
```

`POST /token` from `mcp-remote` hits CloudFront, which rewrites it to `POST /oauth2/token` at Cognito's hosted UI. Lambda never sees it.

## Cognito app client

`generate_secret = true` is required. Cognito's OIDC discovery only advertises `client_secret_basic` and `client_secret_post` — public clients (no secret) silently fail the token exchange.

`mcp-remote` derives its callback port from a hash of the server URL. For a given URL it's deterministic. Find it in Claude's MCP logs ("Using automatically selected callback port: XXXXX") and add it to Cognito's allowed callback URLs:

```hcl
callback_urls = [
  "https://claude.ai/api/mcp/auth_callback",
  "https://claude.com/api/mcp/auth_callback",
  "http://localhost:XXXXX/oauth/callback",
  "http://127.0.0.1:XXXXX/oauth/callback",
]
```

## FastMCP on Lambda

Two Lambda-specific issues:

**DNS rebinding protection.** FastMCP validates the `Host` header. Behind CloudFront → API Gateway, the host Lambda sees is the API Gateway's internal domain, not your public domain. Disable the check:

```python
mcp = FastMCP(
    "name",
    transport_security=TransportSecuritySettings(
        enable_dns_rebinding_protection=False
    ),
    ...
)
```

**Stateless sessions.** FastMCP's default mode maintains a session across requests. Lambda creates a new event loop per invocation — the session from `initialize` (200) doesn't exist when `notifications/initialized` arrives (400). Enable stateless mode:

```python
mcp = FastMCP("name", stateless_http=True, ...)
```

**Session manager warm-start.** FastMCP's `StreamableHTTPSessionManager` has a `_has_started` guard that prevents re-running in the same event loop. On warm Lambda invocations a new event loop is created but the module-level manager instance is reused. Reset the flag in the FastAPI lifespan:

```python
@asynccontextmanager
async def lifespan(app):
    mcp.session_manager._has_started = False
    async with mcp.session_manager.run():
        yield
```

## Token verification without internet

API Gateway's JWT authorizer validates the Cognito token before the request reaches Lambda. Lambda doesn't need to re-verify — just decode claims:

```python
class CognitoTokenVerifier(TokenVerifier):
    async def verify_token(self, token: str) -> AccessToken | None:
        try:
            claims = jwt.decode(
                token,
                algorithms=["RS256"],
                options={"verify_signature": False, "verify_aud": False},
            )
            ...
            return AccessToken(token=token, client_id=..., scopes=..., expires_at=...)
        except Exception:
            return None
```

## Claude Desktop config

```json
{
  "mcpServers": {
    "your-server": {
      "command": "npx",
      "args": ["mcp-remote", "https://your-domain.com/mcp/"]
    }
  }
}
```

Claude Desktop doesn't support remote HTTP MCP servers natively. `mcp-remote` is the standard proxy. On first connection it opens a browser for auth. The token is cached and refreshed against the refresh token (Cognito default: 30 days).
