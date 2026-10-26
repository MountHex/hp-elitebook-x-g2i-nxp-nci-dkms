# Source provenance

The four DKMS package files originate from the verified, installed HP EliteBook X G2i driver source snapshot dated October 2026. For publication, `i2c.c` includes an additional dated, comment-only modification notice; the driver logic is unchanged. Their SHA-256 digests are listed in `SOURCE-MANIFEST.tsv`. They are **not** a freshly rebased Linux kernel driver.

`i2c.c` is derived from Linux `drivers/nfc/nxp-nci/i2c.c` (SPDX `GPL-2.0-only`), originally authored by Clément Perrochaud (NXP Semiconductors) and Oleg Zhurakivskyy (Intel Corporation). `nxp-nci.h` is derived from Linux `drivers/nfc/nxp-nci/nxp-nci.h` (SPDX `GPL-2.0-only`) and preserves its original notices.

Upstream reference files:
- https://github.com/torvalds/linux/blob/master/drivers/nfc/nxp-nci/i2c.c
- https://github.com/torvalds/linux/blob/master/drivers/nfc/nxp-nci/nxp-nci.h

The tested derivative includes a DMI-scoped GPIO assignment for the HP EliteBook X G2i, using firmware GPIO resource 0 and enable GPIO resource 1. The generic Linux mapping is preserved for nonmatching systems. The tested snapshot also retains its established IRQ/error behaviour; no claim is made that the HP GPIO change is the *only* difference against any particular upstream Linux release. The exact original base commit has not been established.

`Kbuild` and `dkms.conf` provide the out-of-tree DKMS packaging. The repo contains **no** private signing key, compiled driver, firmware blob or patch to the kernel's `nxp_nci` core.

The NXP source is licensed `GPL-2.0-only`, and the original source-level SPDX, author and copyright notices are maintained. The full GPL version 2 text is in `LICENSE`.
