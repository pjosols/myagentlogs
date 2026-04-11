---
layout: post
title: "One Codebase, Eight Lambda Images, One Lockfile"
date: 2026-04-11
description: Structuring Lambda container images in a monorepo with uv dependency groups.
---

Eight Lambda functions in one repo. Each gets its own container image with only the dependencies it needs. One `pyproject.toml`, one `uv.lock`.

## Dependency Groups

`uv` supports [dependency groups](https://docs.astral.sh/uv/concepts/dependencies/#dependency-groups) — named sets of packages in a single `pyproject.toml`:

```toml
[dependency-groups]
base = ["boto3>=1.34", "awslambdaric>=3.0"]

api = [
    {include-group = "base"},
    "fastapi>=0.115", "PyJWT>=2.10", "stripe>=8.0",
]

authorizer = [
    {include-group = "base"},
    "PyJWT>=2.10",
]

video-processing = [{include-group = "base"}]
```

One lockfile resolves all groups together.

## Dockerfile

Every function uses the same pattern. Only the group name and handler path change:

```dockerfile
FROM python:3.13-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
WORKDIR /build
COPY pyproject.toml uv.lock ./
RUN uv export --frozen --only-group video-processing --no-dev > requirements.txt \
    && pip install --no-cache-dir -r requirements.txt --target /install

FROM python:3.13-slim
WORKDIR /function
COPY --from=builder /install ./
COPY backend/v1/video-processing/ ./
COPY backend/v1/wshtlib/ ./wshtlib/
ENTRYPOINT ["/usr/local/bin/python", "-m", "awslambdaric"]
CMD ["index.lambda_handler"]
```

`uv export --only-group video-processing` produces a minimal requirements file for that function only.

## wshtlib

```dockerfile
COPY backend/v1/wshtlib/ ./wshtlib/
```

`wshtlib` is a shared observability library — structured logging, request context, CloudWatch metrics, Lambda warming. Stdlib only, no external dependencies, so it doesn't belong in any dependency group. Just copied into every image.

## Image Sizes

The authorizer image is ~80MB. The API image is ~180MB. They share a base layer in ECR.
