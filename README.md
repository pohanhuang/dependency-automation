# Dependency Automation

Shared Renovate configuration for repositories that want consistent dependency update rules.

## What This Repo Does

This repository centralizes Renovate settings so other repositories can reuse the same update policy instead of maintaining their own rules.

Current defaults include:

- **Security-only updates**: Only dependencies with known vulnerabilities will be updated (need manual review)
- All non-security updates (patch, minor, major) are disabled by default
- OSV vulnerability database integration enabled via `osvVulnerabilityAlerts`
- Disable some risky or manually managed updates such as Helm and Go major bumps
- Keep `Longhorn` and `kubevirt` updates disabled on release branches only
- Group related dependencies into a single PR, such as:
  - `golang toolchain`
  - `SUSE BCI base images`
  - `k8s + rancher dependencies`

## Supported Managers

- `gomod`
- `dockerfile`
- `github-actions`
- `pip_requirements`

## Branch Policy

- `main` and `master`: normal update flow
- Release branches matching `v*`: not scanned by default; if a repository opts in via `baseBranchPatterns`, only patch and security updates are enabled, and `Longhorn` / `kubevirt` remain blocked

### Forcing Patch-Only Updates on Release Branches

**Problem**: `osvVulnerabilityAlerts` ignores `major.enabled: false`, and standard `packageRules` cannot block major updates for OSV vulnerability fixes ([renovate#42760](https://github.com/renovatebot/renovate/issues/42760)).

**Solution**: Use `vulnerabilityAlerts.force.packageRules` with `allowedVersions` to restrict updates to the current minor version (e.g., `~1.7.0` on the `v1.7` branch).

## Usage

In a repository that uses this shared config, extend it from `renovate.json`:

```json
{
  "extends": [
    "github>harvester/dependency-automation"
  ]
}
```

Add repo-specific overrides locally if needed.

## Repository Support Matrix

**Legend:**
- ✅ Supported
- ❌ Not supported

| Repo | Support main / master | Support 1.7 | Support 1.8 | Support 1.9 |
|------|----------------------|-------------|-------------|-------------|
| [harvester/harvester](https://github.com/harvester/harvester/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/network-controller-harvester](https://github.com/harvester/network-controller-harvester/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/node-manager](https://github.com/harvester/node-manager/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/node-disk-manager](https://github.com/harvester/node-disk-manager/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/networkfs-manager](https://github.com/harvester/networkfs-manager/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/harvester-load-balancer](https://github.com/harvester/harvester-load-balancer/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/pcidevices](https://github.com/harvester/pcidevices/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/eventrouter](https://github.com/harvester/eventrouter/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/vm-import-controller](https://github.com/harvester/vm-import-controller/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/vm-dhcp-controller](https://github.com/harvester/vm-dhcp-controller/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/seeder](https://github.com/harvester/seeder/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/csi-driver-lvm](https://github.com/harvester/csi-driver-lvm/pulls) | ✅ | ❌ | ❌ | ❌ |
| [harvester/terraform-provider-harvester](https://github.com/harvester/terraform-provider-harvester/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/storage-validator](https://github.com/harvester/storage-validator/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/forklift-packaging](https://github.com/harvester/forklift-packaging/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/upgrade-responder](https://github.com/harvester/upgrade-responder/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/harvester-csi-driver](https://github.com/harvester/harvester-csi-driver/pulls) | ✅ | ❌ | ❌ | ❌ |
| [harvester/go-common](https://github.com/harvester/go-common/pulls) | ✅ | ❌ | ❌ | ❌ |
| [harvester/cloud-provider-harvester](https://github.com/harvester/cloud-provider-harvester/pulls) | ✅ | ❌ | ❌ | ❌ |
| [harvester/docker-machine-driver-harvester](https://github.com/harvester/docker-machine-driver-harvester/pulls) | ✅ | ❌ | ❌ | ❌ |
| [harvester/rancherd](https://github.com/harvester/rancherd/pulls) | ✅ | ❌ | ❌ | ❌ |
| [harvester/addons](https://github.com/harvester/addons/pulls) | ✅ | ✅ | ✅ | ✅ |
| [harvester/upgrade-toolkit](https://github.com/harvester/upgrade-toolkit/pulls) | ✅ | ❌ | ✅ | ✅ |
| [harvester/harvester-mcp-server](https://github.com/harvester/harvester-mcp-server/pulls) | ✅ | ❌ | ❌ | ❌ |

## Main Config

The shared Renovate configuration lives in [renovate.json](./renovate.json).
