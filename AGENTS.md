# AGENTS.md — vm-k3s-server

Standalone candy repo for the `k3s-server` layer — a single-node k3s control
plane with ServiceLB, Traefik v2, and local-path-provisioner enabled by default.
The candy lives in `charly.yml` at the repo root: the `require:` deps
(`layer-k3s`, `layer-k3s-kernel`, `plugin-kube`), the `secret_require:`
(`K3S_CLUSTER_TOKEN`) and `env_accept:` (`K3S_SERVER_HOSTNAME`,
`K3S_KUBECONFIG_SERVER`), the `artifact:` kubeconfig publication, the config +
systemd-unit `plan:` steps, the `check:` assertions, and the embedded `skill:`
entity projected into the marketplace corpus as
`/charly-infrastructure:k3s-server`.

Canonical files:

- `charly.yml` — the `k3s-server:` candy entity and the `k3s-server-skill:`
  skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:k3s-server` — the owning skill. The control-plane
  config, the token/hostname contract, the kubeconfig artifact, and
  verification. Load before editing or troubleshooting the layer.
- `/charly-infrastructure:k3s` — the base candy that installs the `k3s` binary
  this layer requires.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, `secret_require:` / `env_accept:`,
  `artifact:`, and service declarations). Load before editing any entity field
  or plan step.
- `/charly-check:check` — the check-verb reference for the deploy-scope
  `kube:` steps (served out-of-process by `candy/plugin-kube`; there is no host
  `charly check kube` command).

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- Validate the manifest with `charly box validate` at the repo root.
- The candy's `plan:` `check:` steps are the functional evidence; the live
  deploy-scope steps prove a `Ready` node, the Traefik IngressClass, the
  local-path StorageClass, and all three addon rollouts.

## Modify this repo

- Edit the `k3s-server:` candy entity AND the `k3s-server-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a
  behaviour change not mirrored in the skill leaves the corpus stale.
- Keep the kubeconfig `artifact:` registration (`register: kubeconfig`) — it is
  what dispatches the post-retrieve merge into `~/.kube/config`.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- The authoritative landing mechanics are `/charly-internals:git-workflow` and
  the umbrella `AGENTS.md` in `opencharly/opencharly`; this signpost does not
  restate them.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time).
