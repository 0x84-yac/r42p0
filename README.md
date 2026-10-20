All five scripts parse clean. Summary of what was implemented:
New: flavor.sh (sourced by all four scripts)
- DRIVER_FLAVOR=valhall|bifrost → prefix / repo / version:
- valhall → VX504X08X + mali-valhall-gpu, default r42p0-01eac0 ← new default
- bifrost → BX304L01B + mali-bifrost-gpu, default r42p0-01eac0
- Derives DRIVER_TARBALL="${PREFIX}-SW-99002-${VER}.tar" and the Arm download DRIVER_URL; all of DRIVER_PREFIX, DRIVER_REPO, DRIVER_TARBALL, DRIVER_URL remain individually overridable (pinning a full tarball auto-syncs DRIVER_VERSION from its rXpY part).
- Derives FLAVOR_ID=<flavor>-<version>, OUT=out/<FLAVOR_ID>, KDIR=build/kernel-<kver>-<flavor>, MDIR=build/mali-<FLAVOR_ID>.
- flavor_detect() picks the flavor/version from a single existing out/<FLAVOR_ID>/ dir; several candidates → clear error listing them.
Build·Big PickleOpenCode
build.sh
~/Desktop/CVE-2023-4211/r42p0:main134.2K (67%)
- Defaults switched to valhall r42p0 (VX504X08X-SW-99002-r42p0-01eac0.tar from mali-valhall-gpu); DRIVER_FLAVOR=bifrost ./build.sh builds Bifrost side by side.
- Per-flavor everything: kernel tree, driver tree, logs build/logs/<FLAVOR_ID>/, artifacts out/<FLAVOR_ID>/ (incl. shared.img, initramfs, DTB). DTB work dir renamed IMG→IMGDIR (fixes the old variable collision with disk.sh's image).
- Extraction locates driver/product/kernel with find (handles the wrapper-dir difference between the two tarballs).
- Purge-before-install of drivers/gpu/arm, include/uapi/gpu and the five Kbase include/linux/*.h shims so flavor/version switches never leave stale driver files.
- New hard checks: DT arm,mali-midgard compatible present in the extracted driver, and CSF config verification — MALI_MODE=csf must end up as CONFIG_MALI_CSF_SUPPORT=y after olddefconfig (guards the depends on MALI_MIDGARD=m Kconfig variants), and must be off for jm.
- Flavor passed explicitly to initramfs_build.sh and disk.sh; summaries/usage print the flavor.
run_arm64.sh
- Resolves out dir: OUT > --flavor / DRIVER_FLAVOR > auto-detect (error if ambiguous).
- Re-stages share/mali/ from the selected flavor's out dir before boot (NO_STAGE=1 to skip), so the 9p share can never deliver the other flavor's module; its disk auto-build now passes IMG explicitly.
initramfs_build.sh / disk.sh
- Flavor-aware defaults for KO/OUT and IMG (auto-detect when unpinned; OUT pinned without KO → module taken from the same dir); explicit env still wins. Added flavor line to the initramfs summary; docs/--help ranges updated in all scripts.
tools/mali_smoke.c
- Comments only: ioctl definitions confirmed stable across r36p0↔r42p0, so one binary serves both builds (behavior unchanged).
CSF answer (implemented as the check above): Valhall r42p0 supports CSF exactly like the r36p0 Bifrost driver — CONFIG_MALI_CSF_SUPPORT exists, defaults to n (Job Manager), so MALI_MODE=jm stays default and MALI_MODE=csf is opt-in for either flavor.
Build · Big Pickle · 15m 47s · 41.4 tok/s
