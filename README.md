# fedora-devops-silverblue

Automated setup for Fedora 44 Silverblue using [go-task](https://taskfile.dev).

> Tested on Fedora 44 Silverblue

## Prerequisites

- [`go-task`](https://taskfile.dev/installation/) installed

## Before You Start

> **Important:** Always run `upgrade` first, then install system packages, before proceeding with any setup task.

```bash
go-task upgrade
# reboot after upgrade completes

go-task install-desktop   # or install-laptop
# reboot after packages are layered
```

## Usage

```bash
# List all available tasks
go-task

# Full desktop setup
go-task setup-desktop

# Full laptop setup (includes Tailscale + Bluetooth disable)
go-task setup-laptop
```

## Tasks

| Task | Description |
|------|-------------|
| `upgrade` | Upgrade system via rpm-ostree |
| `install-desktop` | Install system packages for desktop |
| `install-laptop` | Install system packages for laptop (includes Tailscale) |
| `toolbox` | Create devops toolbox container |
| `flatpak-config` | Configure Flatpak remotes (disable Fedora, enable Flathub) |
| `flatpak-install-desktop` | Install Flatpak apps for desktop |
| `flatpak-install-laptop` | Install Flatpak apps for laptop |
| `ohmyzsh` | Install Oh My Zsh and configure `.zshrc` |
| `bluetooth-disable` | Disable Bluetooth autostart (laptop only) |
| `auto-theme-switcher` | Set up systemd timers for light (6AM) / dark (8PM) theme switching |
| `luks-verify` | Verify LUKS partition and print detected device |
| `luks-tpm2-enroll` | Enroll TPM2 key for LUKS auto-unlock on boot (laptop only) |
| `setup-desktop` | Full desktop setup |
| `setup-laptop` | Full laptop setup |

## Notes

- `install-desktop` / `install-laptop` require a reboot after `rpm-ostree install` to apply layered packages.
- `luks-tpm2-enroll` binds the LUKS slot to TPM2 PCRs 0+1+2+7, updates `/etc/crypttab`, and regenerates the initramfs via `dracut`.
