# android_vendor_samsung_universal7870-common-patches — `lineage-22.2-bpf`

Platform patches for **LineageOS 22.2 (Android 15)** on Samsung Exynos 7870
(`universal7870-common`), used by the `a3y17lte` (Galaxy A3 2017) builds.

This branch is the public
[`samsungexynos7870/android_vendor_samsung_universal7870-common-patches`](https://github.com/samsungexynos7870/android_vendor_samsung_universal7870-common-patches)
`lineage-22.2-bpf` set **plus three private commits** (the OTA channel patches
and this README/`.gitignore`) that exist only on this GitHub account.

## Patches

| Directory | Patch | What it does |
| --- | --- | --- |
| `build/soong` | `0001-soong-cc-globally-disable-Shadow-Call-Stack-SCS.patch` | Disables ARM64 Shadow Call Stack globally, avoiding x18-clobber SIGSEGVs from `libGLES_mali.so` in Bluetooth/NFC without disabling Zygote GL preload. |
| `packages/modules/Bluetooth` | `0001-Dont-abort-when-real-time-scheduling-is-unavailable.patch` | Stops Bluetooth from aborting when real-time scheduling is unavailable. |
| `device/samsung/a3y17lte` | `0001-a3y17lte-Point-Updater-to-Exynos7870-labs-OTA-channe.patch` | Sets `lineage.updater.uri` to `https://raw.githubusercontent.com/Exynos7870-labs/lineage_release/lineage-22.2/{device}.json` — the functional OTA switch (the stock Updater prefers the property over its built-in URL). |
| `packages/apps/Updater` | `0001-Updater-Default-to-Exynos7870-labs-OTA-channel.patch` | Aligns the Updater app's built-in fallback URL and changelog target with the same channel. Carried as a patch; no Updater fork is maintained. |

The last two rows are the private additions on top of the public set.

Each patch is `git format-patch` output applied with `git am`, signed off by
`FlominatorGD <github.flominatorgd@gmail.com>`.

## How the patches get applied

`apply_patches.sh` is invoked by
`device/samsung/universal7870-common/vendorsetup.sh` during `brunch`. It `git am`s
every `*.patch` in each directory listed in `PATCH_DIRS` into the matching
Android project, skipping any patch whose commit subject is already present in
history (so re-runs are idempotent).

```bash
PATCH_DIRS=("build/soong" "device/samsung/a3y17lte"
            "packages/apps/Updater" "packages/modules/Bluetooth")
```

It can also be run manually from anywhere inside the tree:

```bash
./vendor/samsung/universal7870-common-patches/apply_patches.sh [--revert|--export]
```

## Syncing into your tree

`a3y17lte.xml` in
[`samsungexynos7870/android_manifest_samsung_a3y17lte`](https://github.com/samsungexynos7870/android_manifest_samsung_a3y17lte)
already points the patches project at this GitHub account — **no separate
override file is needed** (no `local_manifest.sh`, no `zz-a3y17lte-private.xml`).

```bash
mkdir -p ~/Los22 && cd ~/Los22
repo init -u https://github.com/LineageOS/android.git -b lineage-22.2 --git-lfs
git clone -b lineage-22.2 \
  https://github.com/samsungexynos7870/android_manifest_samsung_a3y17lte.git \
  .repo/local_manifests

repo sync --force-sync --no-clone-bundle --no-tags
```

The two lines in `a3y17lte.xml` that do it:

```xml
<remote fetch="https://github.com/FlominatorGD" name="flominatorgd" />
<project name="android_vendor_samsung_universal7870-common-patches"
         path="vendor/samsung/universal7870-common-patches"
         remote="flominatorgd" revision="lineage-22.2-bpf" />
```

Everything else — device tree, kernel, vendor, Updater — comes from the public
repos unmodified.

> If you ever move the private commits to a separate branch (e.g.
> `lineage-22.2-bpf-private`), change `revision` here to match — the branch that
> actually carries the private patches is the one to point at.

## Signing keys

OTA installs only work between builds signed with the same keys, but the keys
are **not** a patch and **not** in this repo. Generate them once on the build
machine into the untracked `vendor/lineage-priv/keys` directory (picked up
automatically by `vendor/lineage/config/common.mk`) and keep a backup in
`~/.android-certs`. `.gitignore` blocks `*.pk8`, `*.pem`, keystores and build
artifacts so key material can never be committed here by accident.

## Keeping this branch current

Rebase the private commits onto the public branch:

```bash
git remote add public https://github.com/samsungexynos7870/android_vendor_samsung_universal7870-common-patches.git  # once
git fetch public lineage-22.2-bpf
git rebase public/lineage-22.2-bpf
git push --force-with-lease origin lineage-22.2-bpf
```

If a rebase changes `apply_patches.sh` upstream, re-check that all four
directories are still listed in `PATCH_DIRS`.
