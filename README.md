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

## How to use this image
Once you have installed a standard Fedora CoreOS on your server, you can rebase your system to this custom image by running:

```bash
rpm-ostree rebase ostree-unverified-registry:ghcr.io/ArturoBurela/coreos-homeserver:latest
```

After rebooting, you can pin the image and enable signature verification for future updates.
