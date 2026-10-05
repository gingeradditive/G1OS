# G1OS

Raspberry Pi OS (Bookworm arm64) image, Raspberry Pi 4 only, for the Ginger G1 pellet 3D printer, forked from MainsailOS (`upstream` remote). The repo contains only bash build modules run in chroot by **CustoPiZer** (not CustomPiOS), plus static config files. Application code (klipper4pellet, klipperscreen4pellet, G1-Configs, …) lives in other repos, cloned at build time from HEAD.

Read before fixing bugs:
- `docs/architecture.md`: build pipeline, module execution order, on-device layout
- `docs/modules.md`: per-module reference
- `docs/known-issues.md`: catalogued bugs (B01…) with file/line and proposed fix; update it when fixing one
- `docs/debugging.md`: on-device and CI diagnostics

Conventions:
- Base branch `develop`; Conventional Commits (git-cliff generates CHANGELOG, never edit it by hand).
- Modules run as root with `set -xe`, `source /common.sh` and `source /files/00-config`; use `$BOOT_PATH` (= `/boot/firmware` on Bookworm), never hard-coded `/boot`.
- Files under `/home/$BASE_USER` must end up owned by `$BASE_USER` (use `sudo -u` or `chown`).
- Any new `/home/pi` path must also be handled in `modules/raspberry/files/postrename`.
- No local test suite: validate via the `build.yml` workflow artifact on real hardware.
