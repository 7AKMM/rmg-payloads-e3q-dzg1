# Credits and provenance

This repository is **not** an original project. It is a small, device-specific
payload source derived from the Root My Galaxy catalog and built on other
people's work. Everything here that is not ours is credited below.

## Upstream catalog and feed contract

- **[BuSung-dev/Root-My-Galaxy-Payloads](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads)** —
  the original catalog this repository is derived from: the `support/targets-v3.json`
  schema, the payload/daemon pair contract, the per-target profile layout
  (`src/targets/...`), and the exploit artifact this feed serves
  (`cve-2026-43499-app.so`, the S928B DZF2 stable build, SHA-256
  `a49b378d654c7e637697a701c3c4c5fd02d22b9b30a7069c03e64ec5844af206`).
- **[rushiranpise/Root-My-Galaxy-Payloads](https://github.com/rushiranpise/Root-My-Galaxy-Payloads)** —
  the fork whose KernelSU v3.2.5 Samsung patch is vendored here as
  `patches/KernelSU-v3.2.5-samsung-kdp-rkp-defex.patch` and used to build the
  module and daemon.

Both catalogs are forks of one another; this repository is a third, much
smaller derivative that serves exactly one device.

## The `selinux_hide` fix

The KernelSU feature this build exists for is upstream KernelSU's
`selinux_hide` (feature id 4, "SELinux Hide - sanitize /sys/fs/selinux access
results for app UIDs"). On Samsung's KDP/RKP kernels its stock implementation
cannot install, because the four hook slots it writes are in read-only memory.

The kprobe fallback that makes it install comes from:

- **[PR #257 — "Fix Hide SELinux modification feature in KSU via kprobe (guarded by NO_PATCH_TEXT)"](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/pull/257)**
  by **[diabl0w](https://github.com/diabl0w)** — the original patch
  (`kernelsu/patches/KernelSU-v3.2.5-samsung-selinux_hide.patch`): kprobes on
  the four handlers, an xarray recursion guard keyed on the calling task, and
  the `CONFIG_KSU_SAMSUNG_NO_PATCH_TEXT` guard.
- The implementation in the `rushiranpise` patch, which this build compiles:
  the same approach carried forward with per-slot logging
  (`slot patched` / `kprobe route active` / `kprobe err`), a `find_kernel_symbol_exact`
  lookup for `selinux_setprocattr`, and the fix that reports the feature as
  disabled when an enable is refused instead of before it.

Our contribution is limited to: building that patch for the DZG1 release string,
packaging the module and daemon as a pair, and publishing the feed entry that
points at them.

## Kernel and userspace

- **[tiann/KernelSU](https://github.com/tiann/KernelSU)** v3.2.5
  (`b0bc817b4e966aa6aa830834eaf6ef765d821d40`) — the kernel module and `ksud`
  this pair is built from.
- **Samsung KDP/RKP/DEFEX delta** — maintained in the `rushiranpise` patch named
  above, itself derived from the work published in the upstream catalog.
- **DDK build image** `ghcr.io/ylarod/ddk-min:android14-6.1-20260313` — the
  kernel tree the module is compiled against.
- **Android NDK r28** — the `ksud` cross-compile.

## Exploit payload

`cve-2026-43499-app.so` is served unchanged from the upstream catalog above. Its
source and build recipe live there; the exploit research is credited by that
project to **NebuSec / CyberMeowfia** (see e.g. the acknowledgements in
[PR #426](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/pull/426)).

## Applications

- **Root My Galaxy** by **BuSung-dev** — the installer this catalog format and
  payload contract belong to.
- **[Root My Galaxy Next](https://github.com/rushiranpise/Root-My-Galaxy-Next)**
  by **rushiranpise** — the fork this source is meant to be added to; it reads
  `support/targets-v3.json` from a source like this one.

## What is ours

- `support/targets-v3.json` — one entry, `e3q-S928BXXS6DZG1-ksu325*`, for the
  SM-S928B on `S928BXXS6DZG1`.
- `kernelsu/android14-6.1_kernelsu-e3q-S928BXXS6DZG1-kdp.ko` and
  `kernelsu/ksud-e3q-S928BXXS6DZG1-kdp` — the pair built for that release.
- The build and verification record in `README.md`.

Nothing here is redistributed under a different licence than its upstream:
KernelSU and the Samsung delta are GPL-2.0, the catalog artifacts follow the
upstream catalog's terms.
