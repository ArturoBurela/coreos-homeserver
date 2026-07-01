# coreos-homeserver

Custom, immutable Fedora CoreOS image for Homelab with ZFS and Tailscale pre-installed.

This image is built daily via GitHub Actions using [BlueBuild](https://blue-build.org/).

## Features
- **Base OS**: Fedora CoreOS (Stable)
- **Filesystem**: ZFS support compiled dynamically via DKMS matching the specific kernel version.
- **Networking**: Tailscale pre-installed and enabled.
- **Security**: 
  - Firewalld installed with a default `drop` policy, allowing only `ssh`.
  - Cryptographically signed with `cosign`.
- **Containers**: Podman-compose included out of the box.

## How to Install (Bare Metal)

1. Download the [Fedora CoreOS Live ISO](https://fedoraproject.org/coreos/download?stream=stable) and flash it to a USB drive.
2. Boot your bare-metal server from the USB drive.
3. Once in the terminal, identify your target installation drive using:
   ```bash
   lsblk
   ```
   *(For example, `/dev/sda` or `/dev/nvme0n1`)*
4. Run the installer, pointing it to your drive, this GitHub repository's Ignition file, and the custom image URL:
   ```bash
   sudo coreos-installer install /dev/sda \
     --ignition-url https://raw.githubusercontent.com/ArturoBurela/coreos-homeserver/main/config.ign \
     --image-url ostree-unverified-registry:ghcr.io/arturoburela/coreos-homeserver:latest
   ```
5. Remove the USB drive and reboot (`sudo reboot`). Your server will boot into your customized OS with your SSH keys pre-installed.

*(Note: If you have an existing Fedora CoreOS installation, you can switch to this image by running `rpm-ostree rebase ostree-unverified-registry:ghcr.io/arturoburela/coreos-homeserver:latest` and rebooting).*
