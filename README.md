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
  - Full Disk Encryption (LUKS) bound to the hardware TPM2.
  - Passwordless `sudo` configured via SSH keys.
- **Containers**: Podman-compose included out of the box.

## How to Install (Bare Metal)

The recommended installation method leverages a "Live" environment (like Fedora Workstation Live) to run the `coreos-installer`. This avoids common issues with offline signature verification.

### 1. Initial Installation
1. Download a [Fedora Workstation Live ISO](https://fedoraproject.org/workstation/download) and flash it to a USB drive (e.g. using Ventoy).
2. Boot your bare-metal server from the USB drive.
3. Open a terminal and identify your target installation drive using `lsblk` (e.g., `/dev/sda` or `/dev/nvme0n1`).
4. Install `coreos-installer`:
   ```bash
   sudo dnf install -y coreos-installer
   ```
5. Run the installer, pointing it to your drive and this repository's Ignition file. Do **not** use the `--image-url` flag yet, just install the base OS:
   ```bash
   sudo coreos-installer install /dev/sda \
     --ignition-url https://raw.githubusercontent.com/arturoburela/coreos-homeserver/main/config.ign
   ```
   *Replace `/dev/sda` with your actual drive.*
6. Remove the USB drive and reboot (`sudo reboot`). 

### 2. Rebasing to the Custom Image
Your server will now boot into the base Fedora CoreOS using your SSH keys and LUKS encryption.
1. SSH into your new server from your machine:
   ```bash
   ssh aburela@<IP_ADDRESS>
   ```
   *(Note: If you get a "Remote host identification has changed" error, run `ssh-keygen -R <IP_ADDRESS>` on your Mac).*
2. Rebase the OS to the custom BlueBuild image. Use `--bypass-driver` to override Zincati:
   ```bash
   sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/arturoburela/coreos-homeserver:latest --bypass-driver
   ```
3. Reboot the server.
   ```bash
   sudo reboot
   ```

### 3. Final Security Steps (Post-Reboot)
1. **Enable Signature Verification**: Once you boot into the new image, lock down the system so it only accepts signed updates:
   ```bash
   sudo rpm-ostree rebase ostree-image-signed:docker://ghcr.io/arturoburela/coreos-homeserver:latest --bypass-driver
   ```
2. **Connect Tailscale**:
   ```bash
   sudo tailscale up
   ```
3. **Set a Custom Hostname**:
   By default, the server will have a generic hostname (like `localhost`). Set a custom name so it is easily identifiable on your network and in Tailscale:
   ```bash
   sudo hostnamectl set-hostname <your-custom-hostname>
   sudo reboot
   ```

## TPM2 Encryption & Recovery Key

During installation, Ignition generates a highly secure, random master key for the LUKS volume, stores it in the TPM2 chip, and discards the plaintext password. If your motherboard or TPM chip fails, you will lose access to the drive entirely.

**It is highly recommended to extract and back up the TPM-generated passphrase:**

1. Find the LUKS slot used by the TPM (usually `1` or `0`):
   ```bash
   sudo clevis luks list -d /dev/sda4
   ```
   *(Replace `/dev/sda4` with your LUKS partition).*
2. Extract the randomly generated passphrase:
   ```bash
   sudo clevis luks pass -d /dev/sda4 -s 1
   ```
   *(Replace `1` with the correct slot number from the previous command).*
3. **Save this passphrase in a secure password manager.**
4. You can verify the passphrase works without rebooting:
   ```bash
   sudo cryptsetup --test-passphrase open /dev/sda4
   ```
   Paste the password. If it returns without error, the password is correct.
