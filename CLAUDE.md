# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Automated Fedora 44 Silverblue setup using [go-task](https://taskfile.dev). The entire automation lives in a single `taskfile.yaml`. Tasks are split into **desktop** and **laptop** variants — the laptop variant adds Tailscale, Bluetooth disable (`bluetooth-autostart-disable`), and `dev.deedles.Trayscale` Flatpak.

## Running Tasks

```bash
go-task                    # list all tasks
go-task <task-name>        # run a task
```

Tasks that call `rpm-ostree install` or `rpm-ostree initramfs` require a reboot before changes take effect.

## Task Execution Order

Some tasks have hard ordering dependencies:

1. `upgrade` → reboot → `install-desktop` or `install-laptop` → reboot → setup tasks
2. `luks-tpm2-enroll` → `update-initramfs` → reboot
3. `install-virt` → reboot → `config-virt` → log out/in

The composite `setup-desktop` and `setup-laptop` tasks call the individual subtasks in the correct order automatically.

## Key Constraints

- `rpm-ostree` is immutable-OS aware — package changes are staged and only active after reboot.
- `luks-tpm2-enroll` auto-detects the LUKS device via `lsblk`. Run `lsblk -f | grep crypto_LUKS` to verify the device before running.
- TPM2 PCR binding uses `0+1+2+7`. Changing firmware or bootloader will break auto-unlock.
- The `ohmyzsh` task appends to `~/.zshrc` rather than replacing it — running it twice will duplicate entries.
- `toolbox` pulls `ghcr.io/whilcayangyang/whil-docker-images/fedora-devops-toolbox:latest` — this is the user's private image.

## Adding Tasks

When adding a new task:
- Mirror the desktop/laptop split pattern if the task applies to only one form factor.
- Add it to the relevant composite task (`setup-desktop` / `setup-laptop`) if it should run as part of full setup.
- Use `vars` with `sh:` for dynamic values (see `luks-tpm2-enroll` as the reference pattern).
- Update the task table in `README.md` to match.
