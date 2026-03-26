# Secret Manager GitOps Repository

This repository contains SOPS-encrypted Kubernetes secrets managed by the [Secret Manager](https://github.com/Naikelin/secret-manager) application.

## ⚠️ Security Notice

All secrets in this repository are encrypted using [SOPS](https://github.com/mozilla/sops) with Age encryption. The encrypted files are safe to commit to Git.

**DO NOT** commit unencrypted secrets or the Age private key to this repository.

## Repository Structure

```
namespaces/
├── development/
│   └── secrets/
│       ├── test-secret-final.yaml    # SOPS encrypted
│       └── test-secret-e2e.yaml      # SOPS encrypted
├── staging/
│   └── secrets/
└── production/
    └── secrets/
```

## How It Works

1. **Secret Manager** publishes secrets to this repo (SOPS encrypted)
2. **FluxCD** monitors this repo for changes
3. **FluxCD** decrypts secrets using the Age key stored in K8s
4. **FluxCD** applies secrets to the Kubernetes cluster

## Drift Detection

The Secret Manager continuously monitors for drift between:
- **Git** (this repository) - Source of Truth
- **Kubernetes** (live cluster) - Actual State

When drift is detected, you can:
- **Sync from Git**: Overwrite K8s with Git version (undo manual changes)
- **Import to Git**: Accept K8s changes and update Git (promote manual changes)
- **Mark Resolved**: Acknowledge drift without action

## SOPS Configuration

Secrets are encrypted using Age with the public key configured in `.sops.yaml` at the root of this repository.

**Encryption format:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: development
type: Opaque
data:
  KEY: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
```

## FluxCD Kustomization

FluxCD monitors this repository using a Kustomization resource:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: development-secrets
  namespace: flux-system
spec:
  interval: 1m
  path: ./namespaces/development/secrets
  prune: true
  sourceRef:
    kind: GitRepository
    name: secrets-repo
  decryption:
    provider: sops
    secretRef:
      name: sops-age
```

## Related

- **Main Project**: [Naikelin/secret-manager](https://github.com/Naikelin/secret-manager)
- **SOPS**: [mozilla/sops](https://github.com/mozilla/sops)
- **FluxCD**: [fluxcd.io](https://fluxcd.io)

## License

This repository is part of the Secret Manager project. See the main repository for license information.
