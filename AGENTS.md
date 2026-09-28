# AI agent instructions — rpios-detect

`AGENTS.md` is the sole project instruction file for all coding agents.

Read-only CLI that answers whether a MicroSD card, mounted boot volume, directory, or `.img` contains **Raspberry Pi OS**. Firmware files are not enough.

## CI, cost and documentation

- Prefer [Google Cloud Build](https://cloud.google.com/build/pricing) for
  eligible CI only when the billing account's verified free minutes cover the
  entire forecast run with headroom. Use [AWS CodeBuild](https://aws.amazon.com/codebuild/pricing/)
  likewise for qualifying small on demand builds. Check shared usage, machine
  eligibility, and logging, storage and network costs. If neither free
  allowance safely covers the run, use [Blacksmith](https://docs.blacksmith.sh/blacksmith-runners/overview)
  for supported GitHub Actions or the project's required native provider.
- Keep existing required checks until a replacement proves equivalent
  statuses for the exact commit, native coverage, security and release gates. Follow
  provider mandates in each project and avoid duplicate runs. For suitable
  optional compute, assess [Cloud Run](https://cloud.google.com/run/pricing)
  and [AWS Lambda](https://aws.amazon.com/lambda/pricing/) free usage with the
  same forecast for the complete run; never move a live service merely to use a free tier.
- Minimize total cost across CI, hosting, storage, network, APIs, AI and tooling.
  Choose the least costly option that meets the task's quality, security,
  reliability and performance requirements. Preserve mandated models and gates;
  never trade away correctness, coverage, accessibility or data safety for price.
- Use the fewest hosted CI runs that still cover changed paths, scheduled
  checks and required gates. Iterate locally, route jobs by scope, reuse valid
  caches, avoid duplicate runs and bound retries/concurrency. Cancel superseded
  verification when safe; review releases and migrations before cancellation.
  Preserve checks for the exact commit and native platforms. Measure usage,
  expire disposable artifacts and retire only verified idle resources within
  task authority.
- Keep Markdown focused: one canonical home per topic, short sections and useful
  links. Keep commands and safeguards near their use; move detailed history to
  dated evidence. Update stale guidance against code, preserve release records,
  and avoid duplicating this policy in every document.

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
