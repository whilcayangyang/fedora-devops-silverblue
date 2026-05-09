# fedora-devops-silverblue

Automated setup for Fedora 44 Silverblue using [go-task](https://taskfile.dev).

> Tested on Fedora 44 Silverblue

## Prerequisites

- [`go-task`](https://taskfile.dev/installation/) installed

## Getting Started

All tasks run inside the devops toolbox container. Create and enter it first:

```bash
toolbox create --image ghcr.io/whilcayangyang/whil-docker-images/fedora-devops-toolbox:latest devops
toolbox enter devops
```

Once inside the container, use `go-task` as normal.

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
| `flatpak-config` | Configure Flatpak remotes (disable Fedora, enable Flathub) |
| `flatpak-install-desktop` | Install Flatpak apps for desktop |
| `flatpak-install-laptop` | Install Flatpak apps for laptop |
| `ohmyzsh` | Install Oh My Zsh and configure `.zshrc` |
| `bluetooth-disable` | Disable Bluetooth autostart (laptop only) |
| `auto-theme-switcher` | Set up systemd timers for light (6AM) / dark (8PM) theme switching |
| `luks-tpm2-enroll` | Enroll TPM2 key for LUKS auto-unlock on boot (laptop only) |
| `dracut-regenerate` | Regenerate initramfs for all kernels via dracut |
| `install-virt` | Install virtualization packages (virt-manager, qemu-kvm, libvirt, etc.) |
| `config-virt` | Enable libvirtd and add user to libvirt group |
| `setup-desktop` | Full desktop setup |
| `setup-laptop` | Full laptop setup |

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
go-task dracut-regenerate
```

`luks-tpm2-enroll` binds the LUKS slot to TPM2 PCRs 0+1+2+7 and updates `/etc/crypttab`. Run `dracut-regenerate` after to apply the initramfs changes.

## Virtualization Setup

Run tasks in order:

```bash
go-task install-virt
# reboot after packages are layered

go-task config-virt
# log out and back in for group membership to take effect
```

`install-virt` layers virt-manager, qemu-kvm, libvirt, libvirt-daemon-config-network, virt-install, and bridge-utils via rpm-ostree. `config-virt` enables libvirtd, adds the `libvirt` group entry from `/usr/lib/group`, and adds your user to it.

## Notes

- `install-desktop` / `install-laptop` / `install-virt` require a reboot after `rpm-ostree install` to apply layered packages.
