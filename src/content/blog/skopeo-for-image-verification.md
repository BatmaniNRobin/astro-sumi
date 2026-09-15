---
title: "Skopeo for Image Verification"
description: "Using Docker Alternative Skopeo and Cosign for Image Verification"
pubDate: 2026-05-12
updatedDate: 2026-05-12 # optional
tags: ["docker", "kubernetes", "podman", "tech", "skopeo", "cicd", "pipelines", "devops", "cosign", "gitlab", "github"]
draft: false # drafts are visible in dev, dropped from builds
---

# **Skopeo + Cosign for image verification in your registry workflow**

Published for Platform / SRE teams  |  ~800 words | Part 2

If you haven’t read part 1 of this 3 part series: 

Skopeo in CI/CD

## **Why image signing matters for SRE teams**

A container registry is only as trustworthy as the images it holds. Without signing, there is no cryptographic guarantee that the image you pull matches the one your build pipeline produced. Skopeo handles the transport; Cosign handles the attestation.

## **Tools**

- skopeo — copy and inspect images across registries
- cosign (Sigstore) — sign, verify, and attach attestations to OCI artifacts
- rekor (Sigstore) — (optional) transparency log for signatures (sigstore.dev)

## **Installing Cosign**

# Linux (amd64)

`curl -sSfL https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64 -o /usr/local/bin/cosign && chmod +x /usr/local/bin/cosign`

# Verify the cosign binary itself (bootstrap trust)

`cosign verify-blob --certificate-identity=... cosign-linux-amd64`

# macOS

`brew install cosign`

---

## **Workflow: sign after build, verify before deploy**

The signing and verification steps are separate. (co)Sign in your build pipeline; verify in your admission controller or deploy pipeline.

**Step 1: Generate a key pair (or use keyless)**

```bash
# Key-based (store private key in a secret manager)
cosign generate-key-pair

# Produces: cosign.key (private), cosign.pub (public)
# Keyless (uses OIDC — works well with GitHub Actions, GKE Workload Identity)
# No key management needed; identity is bound to the OIDC token
```

---

**Step 2: Copy the image with Skopeo, then sign**

```bash
IMAGE=registry.internal.example.com/app:$GIT_SHA

# Copy the image into your registry
skopeo copy \
  --authfile /tmp/auth.json 
  docker://build-cache.internal/app:$GIT_SHA \
  docker://$IMAGE

# Pin to the digest (sign the digest, not the mutable tag)
DIGEST=$(skopeo inspect --format '{{.Digest}}' docker://$IMAGE)

# Sign
cosign sign --key cosign.key registry.internal.example.com/app@$DIGEST
```

---

**Tip:** Always sign the digest, not the tag. Tags are mutable. Signing a digest gives you a tamper-evident guarantee tied to specific image content.

---

**Step 3: Verify before deploying**

```bash
IMAGE=registry.internal.example.com/app:$GIT_SHA

DIGEST=$(skopeo inspect --format '{{.Digest}}' docker://$IMAGE)

cosign verify \
  --key cosign.pub \
  registry.internal.example.com/app@$DIGEST
  
# Non-zero exit code = verification failed = block the deploy
```

---

## **Keyless signing in GitHub Actions**

Keyless mode uses GitHub's OIDC token to bind the signature to the workflow identity. No secrets to manage.

```bash
- name: Sign Image
  env:
    COSIGN_EXPERIMENTAL: '1'   # enables keyless
  run: |
    DIGEST=$(skopeo inspect --raw \
      docker://ghcr.io/${{ github.repository }}:${{ github.sha }} \
      | jq -r '.config.digest')

    cosign sign \
      ghcr.io/${{ github.repository }}@$DIGEST
  
- name: Verify image
  env:
    COSIGN_EXPERIMENTAL: '1'
  run: |
    cosign verify \
      --certificate-identity-regexp 'https://github.com/${{ github.repository }}' \
      --certificate-oidc-issuer 'https://token.actions.githubusercontent.com' \
      ghcr.io/${{ github.repository }}@$DIGEST
```

---

## **Attaching SBOMs as OCI attestations**

```bash
# Generate SBOM with syft
syft registry.internal.example.com/app:$GIT_SHA -o spdx-json > sbom.spdx.json

# Attach as attestation
cosign attest --key cosign.key \
  --predicate sbom.spdx.json \
  --type spdxjson \
  registry.internal.example.com/app@$DIGEST

# Copy image + attestations to another registry
skopeo copy --all \
  docker://registry.internal.example.com/app:$GIT_SHA \
  docker://prod-registry.example.com/app:$GIT_SHA
# --all includes the cosign signature and attestation OCI tags
```

---

## **Enforcing verification in Kubernetes**

Use Kyverno or Sigstore Policy Controller to reject pods whose images are unsigned or have an invalid signature:

```bash
# Kyverno ClusterPolicy snippet
rules:
  - name: verify-image-signature
    match:
      resources:
        kinds: [Pod]
    verifyImages:
      - imageReferences:
          - 'registry.internal.example.com/*'
        attestors:
          - entries:
              - keys:
                  publicKeys: |-
                    -----BEGIN PUBLIC KEY-----
                    <your cosign.pub content>
                    -----END PUBLIC KEY-----
```

---

### Stay tuned for part 3: Skopeo in Air-Gapped environments

```markdown
---
title: "Skopeo in CI/CD"
description: "Using Docker Alternative Skopeo"
pubDate: 2026-04-26
updatedDate: 2026-04-26 # optional
tags: ["docker", "kubernetes", "podman", "tech", "skopeo", "cicd", "pipelines", "devops", "cosign"]
draft: false # drafts are visible in dev, dropped from builds
---
```