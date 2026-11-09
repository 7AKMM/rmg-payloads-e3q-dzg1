# rmg-payloads-e3q-dzg1

Payload pair for the **Galaxy S24 Ultra (SM-S928B)** on firmware **S928BXXS6DZG1**
(kernel `6.1.145-android14-11-33419968-abS928BXXS6DZG1`), built so the KernelSU
feature **SELinux Hide** (`selinux_hide`) actually installs on Samsung's
KDP/RKP-protected kernel.

This is meant to be added as a payload source in **Root My Galaxy Next**
(`rushiranpise/Root-My-Galaxy-Next`).

## What it contains

| File | Size | SHA-256 |
|---|---:|---|
| `artifacts/e3q-S928BXXS6DZG1/cve-2026-43499-app.so` | 104,128 | `a49b378d654c7e637697a701c3c4c5fd02d22b9b30a7069c03e64ec5844af206` |
| `kernelsu/android14-6.1_kernelsu-e3q-S928BXXS6DZG1-kdp.ko` | 409,200 | `91c636d6fb5d21bc00709d61467d66d74cfa505cb2654b92f30c8b15325e629f` |
| `kernelsu/ksud-e3q-S928BXXS6DZG1-kdp` | 4,891,808 | `edba75d3f66992cfb10eaf705af59fb8b760e52c476b2e3bf7636dd07b7d87e2` |

The payload is the exact fixed-size build that already roots this device
(`APP_S928_STABLE_RACE=1`, 104,128 bytes). The KernelSU half is a fresh pair for
this release string.

## KernelSU build

- Upstream tag **v3.2.5** (`b0bc817b4e966aa6aa830834eaf6ef765d821d40`), version code **32525**.
- Samsung delta: `KernelSU-v3.2.5-samsung-kdp-rkp-defex.patch` from
  `rushiranpise/Root-My-Galaxy-Payloads`, which includes the **`selinux_hide` kprobe
  fallback** (the fix from BuSung-dev/Root-My-Galaxy-Payloads PR #257, matured:
  per-slot logging, task-keyed xarray guard, and the enable-result check).
- Config: `CONFIG_KSU=m`, `CONFIG_KSU_SAMSUNG_KDP=y`, `CONFIG_KSU_SAMSUNG_RKP=y`,
  `CONFIG_KSU_SAMSUNG_DEFEX=y`, `CONFIG_KSU_SAMSUNG_NO_PATCH_TEXT=y`.
- Module built in `ghcr.io/ylarod/ddk-min:android14-6.1-20260313`, with the DDK's
  generated release replaced by the target release, then `llvm-strip -d`.
  `vermagic: 6.1.145-android14-11-33419968-abS928BXXS6DZG1 SMP preempt mod_unload modversions aarch64`,
  empty `__versions` section.
- `ksud` cross-compiled for `aarch64-linux-android` with NDK r28, embedding the
  module above (`userspace/ksud/bin/aarch64/android14-6.1_kernelsu.ko`).

## Why this build exists

Stock `selinux_hide` writes new function pointers into four SELinux hook slots.
On these Samsung kernels those slots live in the rodata range handed to RKP, so
the write is refused and the feature returns `-ENOSYS` (the manager shows
`error -38`). The patch diverts the same call sites with kprobes instead, which
RKP accepts, and reports which route each slot took.

## Using it in Root My Galaxy Next

1. **Settings → Payload Management → Payload Sources → Add**:
   `7AKMM/rmg-payloads-e3q-dzg1`, branch `main`.
2. Open the payload sheet and pick
   **Galaxy S24 Ultra (International) | DZG1 | KernelSU 3.2.5 + SELinux hide fix**
   (`e3q-S928BXXS6DZG1-ksu325`). It is offered as the exact kernel-release match
   for `S928BXXS6DZG1`.
3. Run. The app downloads this `ksud`, stages it, and the run log should say
   `KernelSU 3.2.5` for the manager.

## Verifying SELinux Hide after the run

```sh
su -c 'ksud feature get selinux_hide'    # 1
su -c 'ksud feature enable selinux_hide'
su -c 'dmesg | grep -E "selinux_hide: (context_write|access_write|setprocattr|sel_open_handle_status):"'
```

Expected: four lines, each ending in `slot patched` or `kprobe route active`.
A `kprobe err:` line means that slot could not be diverted.

## Risks

- The kprobe fallback is **not device-tested on SM-S928B / DZG1**. It is built
  from the patch that is device-tested on other Samsung targets (Z Fold 8,
  S23 family), and the module is audited only against the DDK tree: no
  `check_symbol` run against a DZG1 `vmlinux` was possible here.
- Root and the module are per-boot; a reboot returns the phone to stock.
- If the module misbehaves, power-cycle; nothing is flashed and no partition is
  written.
- Keep the module and the daemon together: `ksud` embeds the `.ko`; do not swap
  one without the other.

## Rebuilding

```sh
git clone --branch v3.2.5 https://github.com/tiann/KernelSU
git apply patches/KernelSU-v3.2.5-samsung-kdp-rkp-defex.patch
# module: see "KernelSU build" above, then:
#   llvm-strip -d kernelsu.ko
#   cp kernelsu.ko userspace/ksud/bin/aarch64/android14-6.1_kernelsu.ko
#   cargo build --release --target aarch64-linux-android
```

## Verified on device (2026-11-09)

SM-S928B / S928BXXS6DZG1, KernelSU 3.2.5 (32525), the `working` entry
(`cve-2026-43499-app-diag.so`) with `ksud-e3q-S928BXXS6DZG1-kdp`:

``text
KernelSU: selinux_hide: set to 1
KernelSU: selinux_hide: init selinux hide
KernelSU: selinux_hide: sel_open_handle_status: patch_text refused (-95), diverting with a kprobe
KernelSU: selinux_hide: sel_open_handle_status: kprobe route active
KernelSU: selinux_hide: context_write: patch_text refused (-95), diverting with a kprobe
KernelSU: selinux_hide: context_write: kprobe route active
KernelSU: selinux_hide: access_write: patch_text refused (-95), diverting with a kprobe
KernelSU: selinux_hide: access_write: kprobe route active
KernelSU: selinux_hide: setprocattr: ksu_lsm_hook refused (-14), diverting with a kprobe
KernelSU: selinux_hide: setprocattr: kprobe route active
``

All four slots fell back to kprobes exactly as the patch intends: RKP refuses the
text write (-95) and the LSM table write (-14), and the kprobe route installs.
`ksud feature get selinux_hide` reports `1` while the hooks are active.
