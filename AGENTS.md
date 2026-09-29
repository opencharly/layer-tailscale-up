# AGENTS.md — layer-tailscale-up

Standalone candy repo for the `tailscale-up` layer — runtime Tailscale wiring
(start the daemon, set `--operator` + `--hostname`) for a `target: local` host.
The candy lives in `charly.yml` at the repo root: the `require:` on
`layer-tailscale`, the deploy `run:` task, the runtime `check:` probes, and the
embedded `skill:` entity projected into the marketplace corpus as
`/charly-infrastructure:tailscale-up`.

Canonical files:

- `charly.yml` — the `tailscale-up:` candy entity and the `tailscale-up-skill:`
  skill entity.
- `CHANGELOG/` — per-CalVer release history; read it before changing baked
  checks.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:tailscale-up` — the owning skill. The runtime task
  (start daemon, `--operator` for non-root `tailscale serve`, `--hostname`
  tracking), the three-state check pattern, and the device-name divergence
  contract. Load before editing or troubleshooting the layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, per-distro `distro:` arms, package/repo
  sections, service declarations). Load before editing any entity field or plan
  step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` steps are the functional evidence. The runtime
  probes use a three-state pattern (daemon down → skip, daemon up + logged out →
  skip, daemon up + logged in → assert); keep that self-gating intact.
- The task body and the `check:` probe bodies read `/etc/hostname` directly
  (never the `hostname` binary) — keep the two forms in sync.

## Modify this repo

- Edit the `tailscale-up:` candy entity AND the `tailscale-up-skill:` skill
  entity in `charly.yml` together. The skill is the projected usage source, so a
  behaviour change not mirrored in the skill leaves the corpus stale.
- The `require:` pins `layer-tailscale`; a change to the daemon contract must
  move that pin.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
