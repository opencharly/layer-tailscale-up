# layer-tailscale-up

Runtime Tailscale wiring for a `target: local` host — start the daemon and set
`--operator` + `--hostname`.

The `tailscale-up` candy is the runtime-config sibling of
[`layer-tailscale`](https://github.com/opencharly/layer-tailscale), which it
`require:`s, so any image composing this layer carries the `tailscale` CLI and
`tailscaled` daemon it drives. On a `target: local` host its single deploy task
starts `tailscaled.service`, sets `--operator` (uid 1000) so a user-systemd
`ExecStartPost` can run `tailscale serve` without sudo, and keeps the tailnet
device name in sync with `/etc/hostname` across hostname changes.

Every step self-gates on `systemctl is-active tailscaled`, so the layer is a
no-op in image-build / bootc-assembly contexts where systemd is not PID 1.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `tailscale-up` |
| Requires | `layer-tailscale` (the daemon + CLI) |
| Packages | none (config-only layer) |
| Service | none (sets state on the existing `tailscaled.service`) |
| Install files | `charly.yml` (one `run:` task) |

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
my-host-profile:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-tailscale-up:v2026.271.1737'
```

Then, on the target host:

```bash
sudo tailscale up                                   # authenticate the daemon
tailscale debug prefs | grep -E '"OperatorUser"|"Hostname"'
tailscale status | head -2
```

The candy's `plan:` asserts the composed CLI at `/usr/bin/tailscale`, the daemon
at `/usr/sbin/tailscaled`, `tailscale version` running offline, and — on a
logged-in tailnet member — that `tailscale debug prefs` succeeds without sudo
with a non-empty `OperatorUser`, that prefs `Hostname` tracks the system
hostname, and that the registered device name matches it. Each runtime probe
pass-skips when the daemon is down or logged out.

## Layout

- `charly.yml` — the `tailscale-up:` candy entity (the `require:` on
  `layer-tailscale`, the deploy `run:` task, the runtime `check:` probes) and the
  embedded `tailscale-up-skill:` skill entity.
- `CHANGELOG/` — per-CalVer release history.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:tailscale-up`
- `/charly-infrastructure:tailscale` — the install-side companion this layer requires
- `/charly-local:local-deploy` — the `target: local` execution model
- `/charly-local:local-spec` — `local.yml` template authoring; canonical consumer `local.charly-cachyos`
- `/charly-core:deploy` — the `charly.yml` `tunnel: tailscale` mechanism that consumes the `--operator` permission
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
