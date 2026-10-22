# LineageOS 22.2 (Android 15) Patches for Samsung Exynos 7870 (`universal7870-common`)

Platform patches for Samsung Exynos 7870 devices (such as Galaxy A3 2017 `a3y17lte`) running LineageOS 22.2 with the 5.4-BPF + cgroup-v2 backported `android_kernel_samsung_exynos7870` (`lineage-22.2`) kernel.

## What Changed vs. Legacy 3.18 Patch Sets

All **BPF-less** (`packages/modules/Connectivity`, `Dns`, `DnsResolver`, `NetworkStack`, `system/bpf`, `system/netd`) and **cgroupv2-less** (`system/core` freezer/cgroupv2, `frameworks/base` `createProcessGroup` ignore) workarounds have been **removed** because the `lineage-22.2` Exynos 7870 kernel now supports:
- Linux 5.4 eBPF, maps, verifier, and `cgroup-bpf` (`NetBpfLoad v0.46`, `bpfloader`, and `netd` `NetdUpdatable` run natively)
- Unified `cgroup2` (`/sys/fs/cgroup`), `CONFIG_CPUSETS=y` (`/dev/cpuset`), `CONFIG_BLK_CGROUP=y` (`/dev/blkio`), `CONFIG_DUMMY=y` (`dummy0`), `xt_connmark` rev 2, and SELinux `extended_socket_class` (`NETLINK_TCPDIAG_SOCKET`)
- `# CONFIG_RT_GROUP_SCHED is not set` so cgroup membership does not block `SCHED_FIFO` threads

## Included Patches (8 Total)

| Directory | Patch | Purpose |
| :--- | :--- | :--- |
| `build/make` | `0001-Hack.patch` | Downgrade `link-type-error` to warning for legacy vendor/shim targets |
| `frameworks/av` | `0001-camera-Allow-devices-to-load-custom-CameraParameter-.patch` | Support custom Samsung `CameraParameters` shim hooks |
| `frameworks/native` | `0001-Disable-gpuservice.patch` | Disable `gpuservice` (Mali-T830 on 3.18 lacks Android 12+ GPU BPF tracepoints) |
| `frameworks/native` | `0002-SurfaceFlinger-Disable-SF-HWC-backpressure.patch` | Gate SF HWC backpressure on `persist.sys.enable_sf_hwc_backpressure` |
| `hardware/interfaces` | `0001-Revert-Remove-HIDL-implementation-for-the-WiFi.patch` | Restore `android.hardware.wifi@1.0-service` HIDL Wi-Fi HAL |
| `packages/modules/Bluetooth` | `0001-Dont-abort-when-real-time-scheduling-is-unavailable.patch` | Retry `SIGEV_THREAD` POSIX timers without `SCHED_FIFO` and avoid fatal aborts when `CAP_SYS_NICE` / `RLIMIT_RTPRIO` is unavailable |
| `system/core` | `0001-reboot-must-be-fast-on-legacy-too.patch` | Fast clean shutdown/remount on legacy non-VAB devices |
| `system/security` | `0001-keystore-hackup.patch` | Trustonic MobiCore Keymaster 1.0 `km_compat` digest/purpose compatibility |

## Usage

When checked out at `vendor/samsung/universal7870-common-patches`, `device/samsung/universal7870-common/vendorsetup.sh` automatically runs `apply_patches.sh` during `. build/envsetup.sh`. Re-running `apply_patches.sh` is idempotent and skips already-applied patches.

Manual invocation from the Android source root:

```bash
./vendor/samsung/universal7870-common-patches/apply_patches.sh
```
