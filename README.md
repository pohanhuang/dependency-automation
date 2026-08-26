# Dependency Automation

Shared Renovate configuration for repositories that want consistent dependency update rules.

## What This Repo Does

This repository centralizes Renovate settings so other repositories can reuse the same update policy instead of maintaining their own rules.

Current defaults include:

- **Security-only updates**: Only dependencies with known vulnerabilities will be updated
- All non-security updates (patch, minor, major) are disabled by default via the `security:only-security-updates` preset
- OSV vulnerability database integration enabled via `osvVulnerabilityAlerts`
- Disable some risky or manually managed updates such as Helm, Go major bumps, and `github.com/rancher/lasso`
- Keep `Longhorn` and `kubevirt` updates disabled on release branches only
- Group related dependencies into a single PR:
  - `golang toolchain` (the `go` directive in `go.mod` plus the `golang` / `registry.suse.com/bci/golang` Dockerfile stages)
  - `k8s + rancher dependencies` (`k8s.io/`, `github.com/rancher/`, `github.com/k3s-io/`), with a PR note to check Longhorn compatibility
  - A `SUSE BCI base images` group is defined but **currently disabled** (`enabled: false`), so BCI base images are not bumped

## Merge Policy

- All vulnerability PRs are labelled `needs-review`, wait for a `minimumReleaseAge` of 3 days, and get `harvester/dev` as reviewer
- **Patch** security updates are then **auto-merged** (squash) via `vulnerabilityAlerts.force`
- **Minor** and **major** security updates require manual review

## Other Inherited Defaults

Consuming repositories also inherit:

- `dependencyDashboard: false` — no dependency dashboard issue is created
- `recreateWhen: "always"` — closing a Renovate PR does not suppress it; the PR is recreated
- `prHourlyLimit: 0` / `prConcurrentLimit: 0` — no cap on open or hourly PRs
- `ignorePaths`: `deploy/**` and `**/charts/**`
- `lockFileMaintenance: enabled` — this is the one PR source that is not vulnerability-driven
- Labels: `dependencies` (gomod / dockerfile / pip), `ci` (github-actions), `needs-review` (vulnerability PRs)
- Semantic commits (`chore(deps)`, `chore(ci)`), `:gitSignOff`, merge-confidence age badges
- `postUpdateOptions`: `gomodTidy`, `gomodVendor`
- Schedule `at any time`, timezone `Asia/Taipei`

## Supported Managers

- `gomod`
- `dockerfile`
- `github-actions`
- `pip_requirements`

## Branch Policy

The shared config sets `baseBranchPatterns` to `master`, `main`, `v1.7`, `v1.8`, `v1.9`. Every consuming repository scans all five; there is nothing to opt into. Branches that do not exist in a repository are simply skipped — that is what the ❌ cells in the support matrix below mean.

- `main` / `master`: security-only updates, at any update type (subject to the global exclusions above — Helm, Go majors, `rancher/lasso`)
- Release branches matching `/^v\d+(\.\d+)+/`, additionally:
  - Major and minor updates are disabled via `packageRules`
  - `Longhorn` (`github.com/longhorn/`) and `kubevirt` (`kubevirt.io/`) Go modules are disabled
  - Note that `packageRules` do not constrain vulnerability-driven updates — see below

### Forcing Patch-Only Updates on Release Branches

**Problem**: `osvVulnerabilityAlerts` ignores `major.enabled: false`, and standard `packageRules` cannot block major updates for OSV vulnerability fixes ([renovate#42760](https://github.com/renovatebot/renovate/issues/42760)).

**Solution**: Use `vulnerabilityAlerts.force.packageRules` with `allowedVersions` to restrict updates to the current minor version. `allowedVersions` is templated as `~{{major}}.{{minor}}.0`, which resolves against the **currently installed version**, so a repo on `v1.7` pinned to `harvester` 1.7.x gets `~1.7.0`.

**Limitation**: this clamp is applied to `github.com/harvester/harvester` only. A major security bump of any other dependency is still possible on a release branch. Extend `vulnerabilityAlerts.force.packageRules` if another package needs the same treatment.

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

Validate changes before merging — consumers of a preset never receive Renovate's automatic config-migration PRs, so problems here surface only as warnings in their logs:

```sh
npx --package renovate -- renovate-config-validator --strict renovate.json
```
