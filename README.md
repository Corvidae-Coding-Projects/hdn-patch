# HDN Linux kernel patch

This repository publishes the HDN Linux kernel-hardening patch for a clean
upstream Linux 7.0.12 source tree. It is the public release artifact; kernel
development happens elsewhere.

HDN is an independent implementation with its own policy model, interfaces,
tests, and source structure. It is not source-, configuration-, or
ABI-compatible with grsecurity or PaX.

## Release identity

| Item | Value |
| --- | --- |
| Upstream version | Linux 7.0.12 |
| Upstream stable commit | `f53879e2e1e2fa053040e734c1ef8f386109a61b` |
| Upstream archive SHA-256 | `57edc9a41efc1ca6b797afa8f4a587a30da2af6bca7356eb56e1e1a4ada265da` |
| HDN source commit | `c2f47ae442eb8669acd93cc9d6e5f606a10a1b35` |
| Patch | `hdn-linux-7.0.12.patch` |
| Patch SHA-256 | `311b4f013a5979d8e37f285a58f99d5edb2eb07d7e98a4de30b7dbc9efccbf5f` |
| Patch size | 4,896,236 bytes; 148,739 lines |

The source commit is public at:

https://github.com/Corvidae-Coding-Projects/hdn-kernel/commit/c2f47ae442eb8669acd93cc9d6e5f606a10a1b35

This checkpoint is newer than the separately qualified HDN release-30 kernel
package and OS image. Their test results must not be treated as qualification
of this newer patch. Its fresh x86 KVM run passed all 1,118 hardening smoke
checks, and the affected tools passed focused compilation and static analysis.
Broader hardware, sanitizer, adversarial, performance, and final acceptance
work remains separate.

## Obtain and verify Linux 7.0.12

Download the official archive:

```sh
curl -LO https://cdn.kernel.org/pub/linux/kernel/v7.x/linux-7.0.12.tar.xz
```

Verify both inputs before applying the patch:

```sh
printf '%s  %s\n' \
  57edc9a41efc1ca6b797afa8f4a587a30da2af6bca7356eb56e1e1a4ada265da \
  linux-7.0.12.tar.xz | sha256sum -c -

sha256sum -c /path/to/hdn-patch/SHA256SUMS
```

Do not apply this patch to another kernel version or to an already modified
tree.

## Apply the patch

Extract the verified upstream archive and enter its source directory:

```sh
tar -xf linux-7.0.12.tar.xz
cd linux-7.0.12
```

First perform a strict dry run, then apply it without allowing fuzzy hunks:

```sh
patch --dry-run --fuzz=0 -p1 \
  < /path/to/hdn-patch/hdn-linux-7.0.12.patch

patch --fuzz=0 -p1 \
  < /path/to/hdn-patch/hdn-linux-7.0.12.patch
```

The dry run and real application must complete without failed, offset, or
fuzzy hunks.

For a Git-managed clean Linux 7.0.12 tree, this is an equivalent preflight:

```sh
git apply --check --whitespace=error \
  /path/to/hdn-patch/hdn-linux-7.0.12.patch
git apply /path/to/hdn-patch/hdn-linux-7.0.12.patch
```

## Configure the kernel

The supported public target is x86_64. GCC is the currently exercised compiler
for the generic configuration. Start with either your distribution's x86_64
kernel configuration or `x86_64_defconfig`; do not assume that an unrelated
configuration is suitable for your hardware.

Example using the upstream x86_64 default:

```sh
build="$PWD/build"
mkdir -p "$build"

make O="$build" x86_64_defconfig
scripts/config --file "$build/.config" \
  -e SECURITY \
  -e SECURITY_HARDENING
make O="$build" olddefconfig
```

Example using the running distribution kernel as a starting point:

```sh
build="$PWD/build"
mkdir -p "$build"
cp "/boot/config-$(uname -r)" "$build/.config"

make O="$build" olddefconfig
scripts/config --file "$build/.config" \
  -e SECURITY \
  -e SECURITY_HARDENING
make O="$build" olddefconfig
```

Review the result interactively before building:

```sh
make O="$build" menuconfig
```

The master option is located under **Security options → Hardening contract
layer**. Enabling `CONFIG_SECURITY_HARDENING=y` selects the default-on HDN
policy and exploit-mitigation baseline. Confirm the final state with:

```sh
grep -E '^CONFIG_(SECURITY|SECURITY_HARDENING)(=|_)' "$build/.config"
```

Important configuration boundaries:

- Options that remain default-off are not required for the ordinary HDN
  baseline. Several are qualification-stage mechanisms with additional
  compiler, architecture, or runtime prerequisites.
- Leave every option ending in `_TEST`, `_FIXTURE`, or `_BAD_FIXTURE` disabled
  in production kernels. Some deliberately expose destructive probes intended
  only for disposable virtual machines.
- `CONFIG_SECURITY_HARDENING_RAP`, RESPECTRE, private stacks, page-table
  ownership, KERNSEAL, and related advanced options must not be enabled merely
  to make the configuration look more complete. Read their Kconfig help and
  satisfy their compiler and verification requirements first. RAP currently
  requires its GCC 16 producer ABI and a source/config/compiler build identity.
- When modules are enabled, the HDN master option forces module-signature
  enforcement. Arrange a trusted signing key and sign any required external or
  DKMS modules. The applied tree documents this in
  `Documentation/admin-guide/module-signing.rst`.
- Secure Boot image signing and bootloader installation are distribution- and
  platform-specific; applying this source patch does not enroll a key or sign
  the resulting kernel image.

The patch contains the kernel enforcement surface only. A complete HDN system
also needs compatible signed policy and userspace management components.
Without them, profile-driven policy is not a complete deployed system even
though build-time kernel defaults are present.

## Build

Build out of tree:

```sh
make O="$build" -j"$(nproc)" bzImage modules
```

Kernel installation, initramfs generation, bootloader updates, module signing,
and rollback entry creation are distribution-specific. Keep a known-good
kernel and recovery path before installing a custom kernel.

## Licensing

The Linux kernel is distributed under GPL-2.0 with the Linux syscall note, and
individual files can carry additional compatible SPDX expressions. This patch
does not replace those per-file terms. `LICENSE` contains the GNU General
Public License version 2; after application, the kernel tree's `COPYING` and
`LICENSES/` directories remain the authoritative licensing record for the
combined source.

## Reporting problems

Please include the exact patch SHA-256, compiler version, architecture, final
`.config`, build log, and whether the problem reproduces on unmodified Linux
7.0.12 when opening an issue:

https://github.com/Corvidae-Coding-Projects/hdn-patch/issues
