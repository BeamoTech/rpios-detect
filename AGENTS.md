# AI agent instructions — rpios-detect

`AGENTS.md` is the sole guide for all agents.

Read-only CLI that answers whether a MicroSD card, mounted boot volume, directory, or `.img` contains **Raspberry Pi OS**. Firmware files are not enough.

## CI, cost and documentation

Read `~/dev/AGENTS.md` for shared checkout, cost, documentation and storage rules.

- Iterate locally; run the applicable full local gate before release. Hosted
  CI is only for necessary final public/customer production verification,
  never routine work, draft PRs, previews or unshipped instruction/doc maintenance. Local
  scripts named `ci` remain local; do not push merely to trigger CI.
- Run the fewest required hosted jobs. Reuse only evidence for the exact final
  SHA, artifacts and config; revalidate after changes. Fix every candidate/gate
  failure and material warning, then rerun until all applicable checks pass.
  Pending, canceled, blocked, timed out and unexpected skips are not passes;
  path skips require workflow evidence. Never weaken tests/coverage or retry blindly.
- Check automatic triggers and gate publication on successful verification.
  Preserve required statuses, branch protection, scheduled security/ops checks,
  native acceptance and approvals; record proof and verify after deployment.
  No hosted CI means retain local/manual gates. Changes to automation or
  publication need task authority. Avoid duplicate providers/runs.

**Project gate:** Use local pytest and safe file-backed fixtures during development. Before
public CLI distribution, require the applicable final Actions/native adapter
checks; a platform fixture is not physical-card acceptance. Agent guide edits
need no manual hosted run. Detection and CI remain read-only, without any
physical media write or firmware mutation.

## Safety

Never write to disks. Mounts must be read-only. Destructive argv (`dd`, `mkfs`, `diskutil erase*`, read-write `mount`) is refused in `safety.py` before exec. Do not weaken that guard. Do not treat internal system disks as SD cards unless the user passed that path explicitly. Watch-mode eject is unmount/eject only (identity re-check immediately before); never format, wipe, or `diskutil erase*`.

## Layout

- `src/rpios_detect/detect.py` — pure evidence → verdict. No subprocess.
- `src/rpios_detect/evidence.py` — rule table. Add OS negative-markers here.
- `src/rpios_detect/snapshot.py` / `fs.py` — host-agnostic file view.
- `src/rpios_detect/probe_*.py` — macOS diskutil / Linux lsblk / Windows.
- `src/rpios_detect/image.py` / `fat.py` — read-only `.img` parse.
- `src/rpios_detect/watch.py` — continuous insert-scan-eject station (`rpiv` / `rpios-detect watch`).
- `src/rpios_detect/ui.py` — flicker-free TTY station screen (redraw only on state change).
- `src/rpios_detect/session.py` — saved station session for `rpiv --resume` / `--clear` / `--status`.
- `src/rpios_detect/eject.py` — cross-platform unmount/eject after a verdict.
- `docs/DETECTION.md` — human rules. Keep it in sync with the matcher table.

Raspberry Pi OS boot filenames are `issue.txt`, `cmdline.txt`, `config.txt`, `bootcode.bin`, `start.elf`, `kernel8.img`, `kernel_2712.img`, `LICENCE.broadcom`. Do not “fix” those to Debian generic names.

## Gate

```bash
python3 -m pip install -e ".[dev]"
python3 -m pytest
```

No physical card in CI. Actions uses Blacksmith; local pytest remains required. A hosted run that never starts supplies no test evidence and does not waive a required check.

Never call firmware-only media `raspberry_pi_os`. If unsure, return `unknown`.

Keep project instructions in `AGENTS.md`.
