# usability-misc

A collection of usability improvements for [Kicksecure](https://www.kicksecure.com/) and [Whonix](https://www.whonix.org/) Linux distributions, packaged as a Debian `.deb`. It ships shell wrappers, configuration drop-ins, systemd units, and a full `zsh` setup to make a freshly installed system more pleasant to use out of the box.

## What's Included

### CLI Tools (`usr/bin/`)

| Command | Description |
|---|---|
| `scurl` | Wrapper around `curl` that enforces TLS 1.3 and HTTPS-only (`--tlsv1.3 --proto =https`) |
| `scurl-download` | Like `scurl`, but delegates to `wcurl` with TLS 1.3 enforced |
| `curl-download` / `curlget` | Thin wrapper around `wcurl` |
| `gsudoedit` | Runs `sudoedit` with a graphical askpass dialog (Wayland-compatible, uses `yad`) |
| `gpl_download_sources` | Downloads GPL-licensed source code for all installed packages via `damngpl` + `apt-get source` |
| `ip_unpriv` | Runs `/bin/ip` through passwordless `sudo` for an unprivileged `tunnel` user |
| `iptables-save-deterministic` | Outputs `iptables-save` with counters zeroed and comments stripped for diffable results |
| `repo-add-dist` | Adds the Kicksecure APT repository and signing key to the system |
| `dist-installer-cli` | CLI installer for Kicksecure/Whonix (source script; auto-generates the standalone version) |
| `virtualbox-send-sysrq` | Sends SysRq key combinations to a VirtualBox VM via `VBoxManage` |
| `whonix-dev-backup` | Backs up Kicksecure and Whonix GitHub repos and wiki dumps |
| `flameshot.dist` | Flameshot wrapper that sets Wayland (sway) environment variables for compatibility |
| `orca-enable-autostart` | Removes the `OnlyShowIn` restriction from Orca screen reader autostart |

### sudo Configuration (`etc/sudoers.d/`)

- **`pwfeedback`** -- Shows asterisks while typing the sudo password.
- **`sudo-lecture-disable`** -- Disables the first-run "trust you have received the usual lecture" message.
- **`tunnel_unpriv`** -- Commented-out rules to allow unprivileged OpenVPN operation.
- **`user-passwordless`** -- Commented-out rule to allow the `user` account to run all commands without a password. Disabled by default.

### GRUB & Boot

- Sets `1024x768` boot screen resolution via `GRUB_GFXPAYLOAD_LINUX` (`etc/default/grub.d/30_screen_resolution.cfg`). Skipped in Qubes.
- Adds a GRUB submenu for switching keyboard layouts at boot (`etc/grub.d/44_kb_layout`).
- Generates GRUB keyboard layouts during package install using `set-grub-keymap --build-all`.
- Creates a Secure Boot MOK key via `shim-signed-mok-setup` on install.

### Shell Environment

- **Default editor** -- Sets `VISUAL` to `featherpad` or `mousepad` if available and unset (`etc/profile.d/50_default_editor.sh`).
- **XDG override** -- Prepends `/usr/share/usability-misc/xdg-override/` to `XDG_DATA_DIRS`, setting `pcmanfm-qt` as default file manager.

### Zsh Configuration (`etc/zsh/`)

Ships a complete zsh configuration (prompt, completions, vi-mode key bindings, syntax highlighting, auto-suggestions) under `/etc/zsh/`. Does **not** change the default shell -- that is handled by `dist-base-files`.

### APT

- Speeds up `apt-get update` by disabling language translations and Contents index (`etc/apt/apt.conf.d/30usability-misc`).

### systemd Services & Drop-ins

- **`avoid-needless-polkit-agent.service`** -- Disables the polkit authentication agent in unprivileged user sessions (when `user-sysmaint-split` is installed).
- **`check-user-slice-on-shutdown.service`** -- Reports hung user processes during shutdown.
- **`orca-kill-at-shutdown.service`** -- Gracefully terminates the Orca screen reader at shutdown.
- **`console-setup.service.d/30_fix.conf`** -- Workaround for a `setupcon` tmpfile race condition ([Debian #846256](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=846256#44)).
- **`openvpn@openvpn.service.d/50_unpriv.conf`** -- Runs OpenVPN as the unprivileged `tunnel` user.
- **`usbguard*.service.d/`** -- Skip USBGuard services when no PCI USB controller is present (virtual machines).

### Kernel Modules

- Prevents KVM and VirtualBox from conflicting by setting `enable_virt_at_load=0` for the `kvm` module (`usr/lib/modprobe.d/usability-misc.conf`).

### DKMS

- Lowers DKMS parallel compilation jobs to 1 on systems with less than 2 GB RAM to prevent VM freezes (`etc/dkms/framework.conf.d/30_usability-misc.conf`).

### Desktop Integration

- **On-Screen Keyboard** desktop entries for launching and stopping `wvkbd`.
- **Mousepad** configured to open files in new windows instead of tabs (via gsettings override).
- **XFCE Terminal** skeleton config with unlimited scrollback and no scroll-on-output.
- Creates `~/Downloads` and `~/Pictures` directories via skeleton placeholders.

### Package postinst Actions

- Creates the `tunnel` system user for unprivileged OpenVPN operation.
- Enables `systemd-journald-audit.socket` (required by `apparmor-info`/`apparmor-watch`).
- Creates `/usr/share/desktop-directories` (Bisq workaround, [bisq-network/bisq#848](https://github.com/bisq-network/bisq/issues/848)).
- Runs `update-grub` if available.

## Installation

### From the APT Repository

1. Download the APT signing key.

```
wget https://www.kicksecure.com/keys/derivative.asc
```

Users can [verify the signing key](https://www.kicksecure.com/wiki/Signing_Key) for better security.

2. Install the signing key.

```
sudo cp ~/derivative.asc /usr/share/keyrings/derivative.asc
```

3. Add the repository.

```
echo "deb [signed-by=/usr/share/keyrings/derivative.asc] https://deb.kicksecure.com trixie main contrib non-free" | sudo tee /etc/apt/sources.list.d/derivative.list
```

4. Update and install.

```
sudo apt-get update
sudo apt-get install usability-misc
```

### Building from Source

Standard Debian tooling works:

```
dpkg-buildpackage -b
```

See the full build instructions (replace `generic-package` with `usability-misc`):

* **A)** [Easy build](https://www.kicksecure.com/wiki/Dev/Build_Documentation/generic-package/easy)
* **B)** [Build with signature verification](https://www.kicksecure.com/wiki/Dev/Build_Documentation/generic-package)

## License

[GNU Affero General Public License v3 or later](https://www.gnu.org/licenses/agpl-3.0.html) (AGPL-3.0-or-later). See `COPYING` for the full text.

## Contact

* [Free Forum Support](https://forums.kicksecure.com)
* [Premium Support](https://www.kicksecure.com/wiki/Premium_Support)

## Donate

`usability-misc` requires [donations](https://www.kicksecure.com/wiki/Donate) to stay alive!
