---
layout: post
title: "Deploying Lambda container images from Apple Silicon"
date: 2026-04-11
description: Three things that will silently break when you push Lambda container images from an M-series Mac.
---

Lambda rejects container images built on Apple Silicon in two ways: wrong architecture, wrong manifest type. Both look like the same error.

```
InvalidParameterValueException: The image manifest, config or layer media type
for the source image ... is not supported.
```

## Problem 1: Multi-arch manifest list

Docker Desktop on Apple Silicon pushes an OCI manifest list by default, even with `--platform linux/amd64`. Lambda rejects it.

What Docker pushes:

```
application/vnd.oci.image.index.v1+json
```

What Lambda accepts:

```
application/vnd.docker.distribution.manifest.v2+json
application/vnd.oci.image.manifest.v1+json
```

Fix: add `--provenance=false` to every `docker build` call.

```bash
docker build --platform linux/amd64 --provenance=false -t my-image .
```

## Problem 2: Immutable ECR tags

If ECR repos have `image_tag_mutability = "IMMUTABLE"` (a reasonable default for prod), you can't overwrite `latest`. The push succeeds silently but the old manifest stays tagged.

Fix: set `MUTABLE` in Terraform, or delete the old image first.

```hcl
resource "aws_ecr_repository" "api" {
  name                 = "my-api"
  image_tag_mutability = "MUTABLE"
}
```

## Problem 3: Terraform creates Lambdas before images exist

If you run `terraform apply` before pushing images, Lambda creation fails. The functions don't get created. Running `deploy_lambdas.sh` afterward fails too because the functions don't exist yet to update.

Fix: push images first, then apply. Or use a `null_resource` with a `local-exec` provisioner to enforce ordering — but that's messy. Easier to just document the sequence:

```
1. terraform apply (ECR repos created, Lambda creation fails — expected)
2. deploy_lambdas.sh (images pushed)
3. terraform apply (Lambdas created successfully)
4. deploy_lambdas.sh (Lambdas updated with latest image)
```

Annoying but predictable once you know it.
