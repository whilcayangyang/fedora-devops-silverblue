# fedora-devops-silverblue

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Fedora](https://img.shields.io/badge/Fedora-44%20Silverblue-294172?logo=fedora&logoColor=white)](https://fedoraproject.org/silverblue/)
[![go-task](https://img.shields.io/badge/built%20with-go--task-00ADD8?logo=task&logoColor=white)](https://taskfile.dev)

Automated setup for Fedora 44 Silverblue using [go-task](https://taskfile.dev).

> Tested on Fedora 44 Silverblue

## Prerequisites

- System upgraded and [`go-task`](https://taskfile.dev/installation/) installed, before proceeding with any setup task

```bash
rpm-ostree upgrade
rpm-ostree install go-task
# reboot once after both complete
```

## Before You Start

> **Important:** Install system packages before proceeding with any other setup task.

```bash
go-task install-plasma-system-app   # or install-gnome-system-app
# reboot after packages are layered
```

## Usage

```bash
# List all available tasks
go-task

# Run an individual task
go-task <task-name>
```

There is no single composite setup task — run the tasks you need in the order
shown in [Task Execution Order](#task-execution-order) below.

## Tasks

| Task | Description |
|------|-------------|
| `install-plasma-system-app` | Install system packages for desktop (KDE Plasma, reboot required) |
| `install-gnome-system-app` | Install system packages for laptop (GNOME, reboot required, includes Tailscale) |
| `flatpak-config` | Configure Flatpak remotes (enable & prioritize Flathub) |
| `flatpak-plasma-app` | Install Flatpak applications from Flathub (Plasma only) |
| `flatpak-gnome-app` | Install Flatpak applications from Flathub (GNOME only) |
| `toolbox` | Create devops toolbox container |
| `ohmyzsh` | Install Oh My Zsh and configure `.zshrc` |
| `gnome-auto-theme-switcher` | Set up systemd timers to switch GNOME theme (light at 6AM, dark at 8PM) |
| `bluetooth-autostart-disable` | Disable Bluetooth autostart (laptop only) |
| `ssh-enable` | Configure and enable SSH daemon with secure defaults |
| `luks-tpm2-enroll` | Enroll TPM2 key for LUKS auto-unlock on boot |
| `luks-initramfs` | Update initramfs to include TPM2 support for LUKS auto-unlock (reboot required) |
| `install-virt` | Install virtualization packages (virt-manager, qemu-kvm, libvirt, etc., reboot required) |
| `config-virt` | Enable libvirtd and add user to libvirt group (reboot required) |
| `firmware-update` | Refresh metadata and apply firmware updates via fwupd |

## Task Execution Order

Some tasks have hard ordering dependencies:

1. `rpm-ostree upgrade` (manual prerequisite, see [Prerequisites](#prerequisites)) → reboot → `install-plasma-system-app` or `install-gnome-system-app` → reboot
2. `luks-tpm2-enroll` → `luks-initramfs` → reboot
3. `install-virt` → reboot → `config-virt` → log out/in

All other tasks (`toolbox`, `ohmyzsh`, `flatpak-config`, `flatpak-plasma-app` / `flatpak-gnome-app`, `gnome-auto-theme-switcher`, `bluetooth-autostart-disable`, `ssh-enable`, `firmware-update`) can be run independently once system packages are installed.

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

To test the theme timers manually after running `gnome-auto-theme-switcher`:

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
go-task luks-initramfs
# reboot after initramfs is updated
```

`luks-tpm2-enroll` binds the LUKS slot to TPM2 PCRs 0+2+4+7 (PCR4 covers the bootloader/shim binary, so auto-unlock also fails if a different boot chain — e.g. alternate boot media — executes). Run `luks-initramfs` after to patch `/etc/crypttab` and enable rpm-ostree initramfs with TPM2 support.

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

- `install-plasma-system-app` / `install-gnome-system-app` / `install-virt` require a reboot after `rpm-ostree install` to apply layered packages.

## License

Licensed under the [GNU General Public License v3.0](LICENSE).
