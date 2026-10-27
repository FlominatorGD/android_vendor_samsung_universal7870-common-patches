# android_vendor_samsung_universal7870-common-patches — `lineage-22.2-bpf-private`

Private patch set for the signed LineageOS 22.2 `a3y17lte`
(Exynos7870) builds. It is the public
`samsungexynos7870/android_vendor_samsung_universal7870-common-patches`
branch `lineage-22.2-bpf` **plus three commits on top**:

1. `device/samsung/a3y17lte/0001-…OTA-channe.patch` — sets
   `lineage.updater.uri` to
   `https://raw.githubusercontent.com/Exynos7870-labs/lineage_release/lineage-22.2/{device}.json`.
   This is the functional OTA switch (stock Updater prefers the
   property over its built-in server URL).
2. `packages/apps/Updater/0001-…OTA-channel.patch` — aligns the
   Updater app's built-in fallback URL/changelog with the same
   channel. Carried as a patch; **no Updater fork is maintained**.
3. This README + `.gitignore` (commit 3).

## How it gets into the build

```bash
mkdir ~/Los22 && cd ~/Los22
repo init -u https://github.com/LineageOS/android.git -b lineage-22.2 --git-lfs
git clone -b lineage-22.2 \
  https://github.com/samsungexynos7870/android_manifest_samsung_a3y17lte.git \
  .repo/local_manifests

# THE override — local file only, exactly as now documented upstream:
cat > .repo/local_manifests/a3y17lte.xml <<'EOF_MANIFEST'
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <!--
    * target: a3y17lte
    * rom: LineageOS 22.2
    -->

    <!-- Remote -->
    <remote fetch="https://github.com/samsungexynos7870" name="gitty" />

    <!-- Device -->
    <project name="android_device_samsung_a3y17lte" path="device/samsung/a3y17lte" remote="gitty" revision="lineage-22.2" />
    <project name="android_device_samsung_universal7870-common" path="device/samsung/universal7870-common" remote="gitty" revision="lineage-22.2" />

    <!-- Kernel -->
    <project name="android_kernel_samsung_exynos7870" path="kernel/samsung/exynos7870" remote="gitty" revision="lineage-22.2" />

    <!-- Vendor -->
    <project name="android_vendor_samsung_a3y17lte" path="vendor/samsung/a3y17lte" remote="gitty" revision="lineage-22.2" />
    <project name="android_vendor_samsung_universal7870-common" path="vendor/samsung/universal7870-common" remote="gitty" revision="lineage-22.2" />

    <!-- board support package (bsp) -->
    <project name="android_hardware_samsung_slsi-linaro_config" path="hardware/samsung_slsi-linaro/config" remote="gitty" revision="lineage-22.2" />
    <project name="android_hardware_samsung_slsi-linaro_graphics" path="hardware/samsung_slsi-linaro/graphics" remote="gitty" revision="lineage-22.2" />
    <project name="android_hardware_samsung_slsi-linaro_exynos" path="hardware/samsung_slsi-linaro/exynos" remote="gitty" revision="lineage-22.2" />
    <project name="android_hardware_samsung_slsi-linaro_openmax" path="hardware/samsung_slsi-linaro/openmax" remote="gitty" revision="lineage-22.2" />
    <project name="android_hardware_samsung_slsi-linaro_exynos5" path="hardware/samsung_slsi-linaro/exynos5" remote="gitty" revision="lineage-22.2" />
    <project name="android_hardware_samsung_slsi-linaro_exynos7870" path="hardware/samsung_slsi-linaro/exynos7870" remote="gitty" revision="lineage-22.2" />

    <!-- Remote -->
    <remote fetch="https://github.com/LineageOS" name="lineage-imports" />

    <!-- Slsi SEpolicy -->
    <project name="android_device_samsung_slsi_sepolicy" path="device/samsung_slsi/sepolicy" remote="lineage-imports" revision="lineage-22.2" />

    <!-- Hardware -->
    <project name="android_hardware_samsung" path="hardware/samsung" remote="lineage-imports" revision="lineage-22.2" />

    <!-- Patches: private set lives on FlominatorGD's GitHub (overrides gitty) -->
    <remote fetch="https://github.com/FlominatorGD" name="flominatorgd" />
    <project name="android_vendor_samsung_universal7870-common-patches" path="vendor/samsung/universal7870-common-patches" remote="flominatorgd" revision="lineage-22.2-bpf" />
</manifest>

EOF_MANIFEST

repo sync --force-sync --no-clone-bundle --no-tags
```

Everything else — device tree, Updater app, kernel, vendor — comes
from the public repos unmodified. As before,
`device/samsung/universal7870-common/vendorsetup.sh` runs
`apply_patches.sh`, which `git am`s every `*.patch` of each directory
listed in `PATCH_DIRS` into the matching project (idempotently, by
commit subject).

## Not in here: signing keys

OTA only installs between builds signed with the same keys, but the
keys are **not** a patch and **not** in this repo. They are generated
once on the build machine into the untracked
`vendor/lineage-priv/keys` directory (which
`vendor/lineage/config/common.mk` picks up automatically) and kept
in `~/.android-certs`, backed up offline. Never commit `*.pk8` or
private `*.pem` anywhere — see the kit's `scripts/gen_keys_once.sh`.

## Keeping this branch current

Keep the three private commits rebased on the public branch:

```bash
git remote add public https://github.com/samsungexynos7870/android_vendor_samsung_universal7870-common-patches.git  # once
git fetch public lineage-22.2-bpf
git checkout lineage-22.2-bpf-private
git rebase public/lineage-22.2-bpf
git push --force-with-lease origin lineage-22.2-bpf-private
```

If a rebase changes `apply_patches.sh` upstream, re-check that
`device/samsung/a3y17lte` and `packages/apps/Updater` are still in
`PATCH_DIRS`.
