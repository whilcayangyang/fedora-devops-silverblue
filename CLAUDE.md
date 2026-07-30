# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Automated Fedora 44 Silverblue setup using [go-task](https://taskfile.dev). The entire automation lives in a single `taskfile.yaml`. Tasks are split into **desktop (KDE Plasma)** and **laptop (GNOME)** variants — the laptop variant adds Tailscale, Bluetooth disable (`bluetooth-autostart-disable`), and `dev.deedles.Trayscale` Flatpak.

## Running Tasks

```bash
go-task                    # list all tasks
go-task <task-name>        # run a task
```

Tasks that call `rpm-ostree install` or `rpm-ostree initramfs` require a reboot before changes take effect.

## Task Execution Order

Some tasks have hard ordering dependencies:

1. `rpm-ostree upgrade` (manual prerequisite, see README) → reboot → `install-plasma-system-app` or `install-gnome-system-app` → reboot
2. `luks-tpm2-enroll` → `luks-initramfs` → reboot
3. `install-virt` → reboot → `config-virt` → log out/in

There is no composite setup task — each task is run individually in the order above.

## Key Constraints

- `rpm-ostree` is immutable-OS aware — package changes are staged and only active after reboot.
- `luks-tpm2-enroll` auto-detects the LUKS device via `lsblk`. Run `lsblk -f | grep crypto_LUKS` to verify the device before running.
- TPM2 PCR binding uses `0+2+7`. Changing firmware or bootloader will break auto-unlock.
- The `ohmyzsh` task appends to `~/.zshrc` rather than replacing it — running it twice will duplicate entries.
- `toolbox` pulls `ghcr.io/whilcayangyang/whil-docker-images/fedora-devops-toolbox:latest` — this is the user's private image.

## Adding Tasks

When adding a new task:
- Mirror the desktop (Plasma) / laptop (GNOME) split pattern if the task applies to only one form factor.
- Use `vars` with `sh:` for dynamic values (see `luks-tpm2-enroll` as the reference pattern).
- Update the task table in `README.md` to match.
