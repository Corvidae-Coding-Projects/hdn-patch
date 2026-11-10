# HDN Linux kernel patch

This repository publishes HDN Linux kernel-hardening patches for the generic
upstream Linux 7.0.12 source tree and three qualified vendor source families.
It is the public release-artifact repository; kernel development happens
elsewhere.

HDN is an independent implementation with its own policy model, interfaces,
tests, and source structure. It is not source-, configuration-, or
ABI-compatible with grsecurity or PaX.

## Vendor-specific patches

These patches include the current HDN source, the persistent exec-after-write
history fix, guarded read-only-filesystem transaction cancellation, native
vendor adaptations, and the qualified optional RAP/private-stack work.

| Exact target source | Patch | SHA-256 |
| --- | --- | --- |
| Ubuntu 26.04, Linux 7.0.14 | `hdn-linux-7.0.14-ubuntu-26.04.patch` | `cad040388cf3a95fb0275c014912fd16de50615abb349aaee0685d18ba04867b` |
| Ubuntu 24.04 HWE, Linux 7.0.14, GitLab variant | `hdn-linux-7.0.14-ubuntu-24.04-hwe-gitlab.patch` | `bdf7012198949702e02e97a8dfa43ec2cd5848113bfc9795de23faaf21d9f576` |
| Debian, Linux 7.3.1, Wazuh variant | `hdn-linux-7.3.1-debian-wazuh.patch` | `ff5311bddf94afe4d6eb88ddf214a5e4ec3aa7f2b2c8e1de001fcfbc36b28bf3` |

The patches are not interchangeable. Use only the patch matching the exact
vendor family and version. Do not force rejected or offset hunks onto a
different revision.

From the root of an unmodified matching source tree, verify the repository and
preflight the selected patch:

```sh
sha256sum -c /path/to/hdn-patch/SHA256SUMS

git apply --check --whitespace=nowarn \
  /path/to/hdn-patch/the-matching-vendor.patch
git apply --whitespace=nowarn \
  /path/to/hdn-patch/the-matching-vendor.patch
```

`--whitespace=nowarn` preserves inherited vendor/upstream whitespace; it does
not permit fuzzy or rejected hunks. Each published patch was independently
replayed against a fresh copy of its pinned baseline, and the result matched
the qualified source tree byte-for-byte, including file modes and symlinks.

### Configure a vendor kernel

Begin with the vendor's configuration for that exact kernel. Enable
`CONFIG_SECURITY_HARDENING=y`, resolve new symbols, and compare the result with
the original vendor configuration:

```sh
cp /path/to/vendor.config .config
make olddefconfig
scripts/config --file .config -e SECURITY -e SECURITY_HARDENING
make olddefconfig
scripts/diffconfig /path/to/vendor.config .config
```

Do not weaken KASLR, strong stack protection, strict kernel/module RWX, module
signature enforcement, memory initialization, FORTIFY, hardened usercopy,
AppArmor, BPF restrictions, or perf restrictions merely to obtain a build.
Leave options ending in `_TEST`, `_FIXTURE`, or `_BAD_FIXTURE` disabled in a
production configuration.

The optional RAP/private-stack profile requires GCC 16 and its complete
compiler/binary verification workflow. It passed isolated compiler coverage
and four-vCPU KVM runtime qualification on all three listed vendor source
families and on generic Linux 7.0.12, but remains default-off in the ordinary
profile. That evidence does not replace native package, Secure Boot,
physical-hardware, workload, or deployment acceptance.

The ordinary HDN profile for all three vendor patches compiled with
`CONFIG_WERROR=y`, forced module signatures, strict kernel/module RWX, HDN
user-WX, and HDN ROFS. Each resulting image booted in an isolated q35 KVM guest
and passed all 1,118 recorded HDN runtime checks with a clean kernel-fault scan.

## Generic upstream 7.0.12 release identity

| Item | Value |
| --- | --- |
| Upstream version | Linux 7.0.12 |
| Upstream stable commit | `f53879e2e1e2fa053040e734c1ef8f386109a61b` |
| Upstream archive SHA-256 | `57edc9a41efc1ca6b797afa8f4a587a30da2af6bca7356eb56e1e1a4ada265da` |
| HDN source commit | `159983aa8b7cc5c47d15c735153ac5805679a5d0` |
| Patch | `hdn-linux-7.0.12.patch` |
| Patch SHA-256 | `7638eb3c1c556841c7987af631b7f0ad94ad31786579e0cecec1faf681ed09ab` |
| Patch size | 5,182,984 bytes; 157,247 lines |

The source commit is public at:

https://github.com/Corvidae-Coding-Projects/hdn-kernel/commit/159983aa8b7cc5c47d15c735153ac5805679a5d0

This checkpoint is newer than the separately qualified HDN release-30 kernel
package and OS image. Their test results must not be treated as qualification
of this newer patch. Its exact successor source passed a fresh complete GCC 16
compiler/binary inventory for 3,310 expected units, an ordinary default-off
regression build, and a four-vCPU KVM boot with RAP/private stacks active,
signed probe modules, and KVM VM creation. The broader 1,118-test smoke,
analyzer, sanitizer, cross-architecture, fuzz, performance, physical-hardware,
and final acceptance records remain bound to predecessors or separate work.

The retained successor evidence is published with the development handoff at:

https://github.com/Corvidae-Coding-Projects/hdn-linux/tree/e8b0d8a42ba243657ccc6a471f568fe33c5df118/verification/evidence/rap-private-stack-7.0.12

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
  satisfy their compiler and verification requirements first. RAP requires
  its GCC 16 producer ABI and a source/config/compiler build identity.
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
