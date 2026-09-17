---
title: "Skopeo in CI/CD"
description: "Using Docker Alternative Skopeo"
pubDate: 2026-04-26
# updatedDate: 2026-04-26 # optional
tags: ["docker", "kubernetes", "podman", "tech", "skopeo", "cicd", "pipelines", "devops", "cosign", "gitlab", "github"]
draft: false # drafts are visible in dev, dropped from builds
---

# Why I replaced Docker pull with Skopeo in CI/CD

Published for Platform / SRE teams | ~800 words | Part 1

---

### The Problem

Running docker pull in CI requires a Docker daemon — which means either a privileged container, a DinD (Docker-in-Docker) sidecar, or an alternative VM-based runner. Each of those is a serious footgun: attack surface, layer cache headaches, and socket-mount nightmares.

Skopeo sidesteps all of it. It talks directly to registry APIs over HTTPS. No daemon, no root, no socket, while being super lightweight. 

### Where Skopeo comes in

Skopeo is a CLI tool for operating on container images and registries without requiring a local container runtime. It can:

- Copy images between registries (docker://, oci://, oci-archive://, dir://, docker-daemon://)
- Inspect remote image manifests and configs without pulling layers
- Delete tags from registries that support the Docker Registry API v2
- Sign and verify images

## **Core workflow: copy instead of pull**

The skopeo copy command is the workhorse. Instead of pulling to a local daemon and re-pushing, you move images registry-to-registry directly:

```bash
# Mirror an upstream image to your internal registry
skopeo copy \
  docker://docker.io/library/nginx:1.25 \
  docker://registry.internal.example.com/mirrors/nginx:1.25

# Authenticate with separate credentials per registry
skopeo copy \
  --src-creds user:token \
  --dest-creds cibot:$CI_REGISTRY_TOKEN \
  docker://upstream.io/app:latest \
  docker://registry.internal.example.com/app:latest

# Copy all tags (useful for full mirrors)
skopeo copy --all \
  docker://docker.io/library/alpine \
  docker://registry.internal.example.com/mirrors/alpine
```

## **Authentication: using a credentials file**

For pipelines that interact with many registries, maintain a single auth config file rather than passing --src-creds and --dest-creds everywhere:

```bash
# Generate auth config from existing docker login sessions
skopeo login --authfile /tmp/auth.json registry.internal.example.com

# Use it in any subsequent command
skopeo copy \
  --authfile /tmp/auth.json \
  docker://upstream.io/app:latest \
  docker://registry.internal.example.com/app:latest

# The file format is standard Docker config.json
# Safe to mount as a Kubernetes Secret
```

---

**Tip**: Mount your registry credentials as a Kubernetes Secret of type kubernetes.io/dockerconfigjson and pass the path to --authfile. No env var juggling.

---

Its lightweight and ability to move registry-registry allows for uses in CICD workflows and container instances meant to leverage without the use of a docker socket, a VM, or any additional attack surfaces.

Making Skopeo a very useful tool for CICD image handling!

Stay Tuned for Part 2 of this 3-part series on Skopeo! 🚀