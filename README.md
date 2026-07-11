# fedora-devops-silverblue

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Fedora](https://img.shields.io/badge/Fedora-44%20Silverblue-294172?logo=fedora&logoColor=white)](https://fedoraproject.org/silverblue/)
[![go-task](https://img.shields.io/badge/built%20with-go--task-00ADD8?logo=task&logoColor=white)](https://taskfile.dev)

Automated setup for Fedora 44 Silverblue using [go-task](https://taskfile.dev).

> Tested on Fedora 44 Silverblue

## Prerequisites

- [`go-task`](https://taskfile.dev/installation/) installed

```bash
rpm-ostree install go-task
# reboot after package is layered
```

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
| `flatpak-config` | Configure Flatpak remotes (enable & prioritize Flathub) |
| `flatpak-install-desktop` | Install Flatpak apps for desktop |
| `flatpak-install-laptop` | Install Flatpak apps for laptop |
| `ohmyzsh` | Install Oh My Zsh and configure `.zshrc` |
| `bluetooth-autostart-disable` | Disable Bluetooth autostart (laptop only) |
| `ssh-enable` | Configure and enable SSH daemon with secure defaults |
| `auto-theme-switcher` | Set up systemd timers for light (6AM) / dark (8PM) theme switching |
| `luks-tpm2-enroll` | Enroll TPM2 key for LUKS auto-unlock on boot (laptop only) |
| `update-initramfs` | Enable rpm-ostree initramfs with TPM2 support (reboot required) |
| `install-virt` | Install virtualization packages (virt-manager, qemu-kvm, libvirt, etc.) |
| `config-virt` | Enable libvirtd and add user to libvirt group |
| `firmware-update` | Refresh metadata and apply firmware updates via fwupd |
| `setup-desktop` | Full desktop setup |
| `setup-laptop` | Full laptop setup |

## Toolbox Aliases

`ohmyzsh` appends aliases to `~/.zshrc` that transparently redirect common DevOps tooling into the `devops` toolbox container (created by the `toolbox` task), so the host system stays package-free:

```bash
alias claude='toolbox run --container devops bash -c "$HOME/.local/bin/claude update; exec $HOME/.local/bin/claude"'
alias kubectl='toolbox run --container devops kubectl'
alias k9s='toolbox run --container devops k9s'
alias talosctl='toolbox run --container devops talosctl'
alias terraform='toolbox run --container devops terraform'
alias vim='toolbox run --container devops vim'
```

Run `go-task toolbox` before `ohmyzsh` so the `devops` container exists when these aliases are first invoked.

## Theme Switcher

To test the theme timers manually after running `auto-theme-switcher`:

```bash
systemctl --user start theme-light.service
systemctl --user start theme-dark.service
```

## LUKS / TPM2 Setup

Before enrolling TPM2, verify your LUKS device:

```bash
lsblk -f | grep crypto_LUKS
```

Then run tasks in order:

```bash
go-task luks-tpm2-enroll
go-task update-initramfs
# reboot after initramfs is updated
```

`luks-tpm2-enroll` binds the LUKS slot to TPM2 PCRs 0+1+2+7. Run `update-initramfs` after to patch `/etc/crypttab` and enable rpm-ostree initramfs with TPM2 support.

## Virtualization Setup

Run tasks in order:

```bash
go-task install-virt
# reboot after packages are layered

go-task config-virt
# log out and back in for group membership to take effect
```

`install-virt` layers virt-manager, qemu-kvm, libvirt, libvirt-daemon-config-network, virt-install, and bridge-utils via rpm-ostree. `config-virt` enables libvirtd, adds the `libvirt` and `kvm` group entries from `/usr/lib/group`, and adds your user to both.

## Flatpak Icon Troubleshooting

After installing Flatpak apps via script, application icons may not appear in the app launcher. This happens because the icon cache isn't refreshed automatically. Run:

```bash
sudo gtk-update-icon-cache -f /var/lib/flatpak/exports/share/icons/hicolor/
sudo gtk4-update-icon-cache -f /var/lib/flatpak/exports/share/icons/hicolor/
```

- `gtk-update-icon-cache` targets GTK3 apps; `gtk4-update-icon-cache` targets GTK4 apps — run both to cover all Flatpaks.
- The `-f` flag forces a rebuild even if the cache appears up to date.
- A log out / log in or `killall gnome-shell` may still be needed for GNOME Shell to pick up the new cache.

## Notes

- `install-desktop` / `install-laptop` / `install-virt` require a reboot after `rpm-ostree install` to apply layered packages.

## License

Licensed under the [GNU General Public License v3.0](LICENSE).
