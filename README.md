# HP EliteBook X G2i NXP NFC DKMS driver

This is the HP-specific NXP NFC I²C driver used on the **HP EliteBook X G2i 14-inch** running Fedora. It makes the laptop's built-in NFC controller available to Linux as a native NFC device (`nfc0`). It does not, on its own, make NFC security keys usable in browser WebAuthn prompts; that requires the separate [HP EliteBook X G2i Linux NFC Bridge](https://github.com/MountHex/hp-elitebook-x-g2i-linux-nfc-bridge).

The driver code matches the tested four-file DKMS package; `i2c.c` also includes a comment-only modification notice for publication. It is intended for people who need the HP-specific driver rather than the standard kernel module. **Do not replace an already working driver just to follow these instructions.**

## Why a separate driver?

The standard `nxp-nci_i2c` transport driver expects the firmware and enable GPIO resources at indices 1 and 2. The tested HP EliteBook X G2i exposes two GPIO resources in a different order. This driver selects indices 0 and 1 when the system identifies itself as the matching HP model with board `8E86`. Other systems use the standard mapping.

The driver also retains its existing IRQ and error-handling behaviour from the tested source. This repository is a known-working source snapshot, **not** a claim that the same source will work against every future kernel.

## Tested configuration

- HP EliteBook X G2i 14-inch, board `8E86`, built-in NXP1002 NFC controller
- Fedora 45; DKMS modules installed for kernels `7.2.10-300.fc45.x86_64` and `7.2.11-300.fc45.x86_64`
- Secure Boot enabled with a signed DKMS module
- Native NFC reader working together with the separately installed NFC bridge and an NFC-capable YubiKey

Other laptops and distributions have **not** been tested. The HP-specific mapping applies only to the exact model and board identity in `i2c.c`.

## What is included

- `i2c.c` — NXP NCI I²C transport with the HP-specific DMI/GPIO mapping
- `nxp-nci.h` — local copy of the kernel NXP NCI header required to build the module
- `Kbuild` — builds the transport module only; uses the kernel's existing `nxp_nci` core
- `dkms.conf` — registers the package as `nxp-nci-i2c-hp-g2i`, version `1.0`

There are no prebuilt kernel modules, MOK signing keys or firmware files here.

## Installing on Fedora (new installation only)

**Read this entire section first.** Replacing a kernel driver is not equivalent to installing an ordinary application. Keep a working kernel available for recovery, and make a system backup before changing drivers. Do not unload, rebind or reset the NFC controller while it is in use. In particular, if the reader or an NFC bridge is already working, leave it alone.

These commands show how to install the **source** through DKMS on a Fedora system with the matching hardware. They have not been independently retested as a complete fresh-install procedure; the bundled driver itself was installed and physically verified on the configuration above.

Install the build tools and headers for the running kernel:

```bash
sudo dnf install dkms gcc make "kernel-devel-$(uname -r)"
```

From this repository's root, copy the four DKMS files into the package source directory:

```bash
sudo install -d -m 0755 /usr/src/nxp-nci-i2c-hp-g2i-1.0
sudo install -m 0644 i2c.c nxp-nci.h Kbuild dkms.conf /usr/src/nxp-nci-i2c-hp-g2i-1.0/
```

Register, build and install for the **running** kernel:

```bash
sudo dkms add -m nxp-nci-i2c-hp-g2i -v 1.0
sudo dkms build -m nxp-nci-i2c-hp-g2i -v 1.0 -k "$(uname -r)"
sudo dkms install -m nxp-nci-i2c-hp-g2i -v 1.0 -k "$(uname -r)"
```

**Secure Boot:** A module must be signed with a key trusted by your machine before the kernel will load it. The tested laptop uses DKMS module signing and an enrolled Machine Owner Key (MOK). Verify your own DKMS signing setup and key enrollment **before relying on the new module**, especially before a kernel change or reboot. Do not copy someone else's signing keys, disable Secure Boot as a workaround, or place private keys in the repository. The commands above do not by themselves prove that signing and enrollment succeeded.

Useful read-only checks:

```bash
uname -r
dkms status
modinfo -F filename nxp_nci_i2c
modinfo -F signer nxp_nci_i2c
mokutil --sb-state
```

An installed DKMS module is not proof that the NFC reader is healthy. Check the reader separately **without unloading or reprobeing the driver**. Once the driver is known to work, install the companion NFC bridge following its own instructions.

## Limits and troubleshooting

- This driver only replaces the **I²C transport**, not the upstream `nxp_nci` core module.
- `dkms.conf` uses Fedora's `/usr/src/kernels/<kernel-version>` header layout. Other distributions require adapting the build setup.
- A kernel update can break an out-of-tree driver build or signing. Confirm the module is built and signed for the new kernel before booting into it.
- Installing or building does not require manually unbinding the controller. Avoid repeated NFC resets, driver reloads or concurrent NFC polling as troubleshooting shortcuts.
- This project is not affiliated with HP, NXP or Intel.

## Licence and upstream attribution

The NXP driver source is derived from the Linux kernel and remains under **GPL-2.0-only**. Original NXP and Intel copyright and author notices remain in the C files. The repository includes the GPL version 2 licence text and an [upstream provenance note](UPSTREAM-PROVENANCE.md).
