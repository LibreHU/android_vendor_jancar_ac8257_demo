# android_vendor_jancar_ac8257_demo

*Français : [README.md](README.md)*

Proprietary `/system` files for **LineageOS 16.0** on the Jancar **UJC201** head unit (AC8257), to be placed in
`vendor/jancar/ac8257_demo` (tree: [android_device_alps_ac8257_demo](https://github.com/LibreHU/android_device_alps_ac8257_demo),
branch `lineage-16.0`, guide `docs/BUILD.en.md`). Branch `lineage-16.0`. Makefiles written by the tree's
`setup-makefiles.sh` (LineageOS 16.0 `extract_utils.sh`).

The whole stock vendor partition (`vendor.img`, 2 GiB) goes in this folder too but is **not in the repository** (too
big, specific to each unit: `.gitignore`).

## Content

| File | Source | Role |
|---|---|---|
| `proprietary/framework/autochips.jar` | stock ROM `UJC201-V1.151R6`: `system/framework/autochips.jar` (empty) + `boot-autochips.vdex` | BOOTCLASSPATH jar (`PRODUCT_BOOT_JARS += autochips`): `AtcMetazone`, `AtcLogo`, `BackCar`, `LocalBluetoothManager`, CarPlay / iAP2, DVR, AVM… (418 classes) |
| `proprietary/lib{,64}/vendor.autochips.hardware.*@*.so` (9 in 32-bit, 10 in 64-bit) | same ROM, `/system/lib{,64}` | HIDL interfaces of the vendor's Autochips HALs (client side) |
| `proprietary/lib{,64}/lib{atcavm,atcvobsub,dvr,logo,metazone}_jni.so` | same ROM | JNI libraries of `autochips.jar` |
| `proprietary/lib{,64}/libusbdvr.so` | same ROM | needed by `libdvr_jni.so` (USB recording) |
| `proprietary/app/AtcWatermark/AtcWatermark.apk` | same ROM, `/system/app` | watermark service (`com.autochips.watermarkservice`, system UID, persistent): serves `IWatermark/WatermarkService` (declared in the stock vendor manifest) to the 360° HAL (`avmhal`), which stamps date, time, GPS and speed on the 360° kit recordings; sends the time and the GPS position every second. Re-signed with the platform key (module `AtcWatermark`) |
| `proprietary/lib{,64}/libwatermark{,_config,_impl,_jni,_service}.so` | same ROM | native watermark server (Skia + FreeType rendering, `/vendor/etc/atc_watermark_config.xml`) |
| `proprietary/lib{,64}/lib{dpframework_mtk,ged_sys,gralloc_extra_sys,graphics2d_ext,ion_mtk_sys,ion_ulit_sys,jpeg-alpha,pq_prot_mtk}.so`, `vendor.mediatek.hardware.{power,pq}@2.0.so` | same ROM | MediaTek dependencies of the watermark (NEEDED, extracted by `stock_extract_vendor.sh` on 14/11/2026; sha1 checked) |

All the other dependencies (`NEEDED`) of these files are in LineageOS 16.0 (`libhidltransport`, `libmedia_omx`,
`libstagefright_foundation`, `libstdc++`, `android.hidl.*`…). List and checksums: `stock_extract_vendor.sh` report
(debugfs on the stock `system.img`, read only); same list as the tree's `proprietary-files.txt`. Firmware other than
V1.151R6: `extract-files.sh ~/stock` takes these files from your `/system` (the jar, pinned, stays the one here).

## Deodexing autochips.jar

The stock jar only holds its manifest; its code is a compact dex (`cdex001`) inside `boot-autochips.vdex` (vdex 019).
Done like `oat2dex()` of `extract_utils.sh` (LineageOS 16.0), then rewritten by d8:

```bash
vdexExtractor -o out/ -i boot-autochips.vdex            # vendor/lineage/build/tools/Linux (sha256 e4c77a30…04cc)
compact_dex_converter out/boot-autochips_classes.cdex  # same (sha256 9f74ccec…4771) -> dex 039
d8 --release --min-api 28 --output d8out classes.dex   # Android SDK build-tools 36.0.0
```

d8 is needed: the dex of `compact_dex_converter` puts an `annotation_set_item` at an unaligned address (`dexdump`:
"Offset(393961) should be aligned by 4 for maplist item of type 4099"), which ART's verifier refuses. The d8 dex passes
`dexdump -c` (checksum and verification); no "quick" instruction left. The file is pinned in the tree's
`proprietary-files.txt` (`-framework/autochips.jar|<sha1>`): `extract-files.sh` keeps it instead of redoing it.

Checksums: stock `autochips.jar` sha256 `364603c0…dec96`, `boot-autochips.vdex` sha256 `9fe6787f…cbef2d`; jar here
sha1 `18a76522…56fde3`.

Checked before publishing: no MAC address, serial number nor certificate (Apple MFi or other) in these files.

## Licence

Proprietary files of Jancar / Autochips, taken from the device's stock ROM to build LineageOS for that same device.
The generated makefiles are under Apache-2.0 (LineageOS header).
