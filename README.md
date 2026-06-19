# SM-T860 One UI 6 (Android 14) — Custom ROM Image Set

为三星Galaxy Tab S6适配的oneUI6（Android14）项目，除指纹外所有全部硬件可用。

中国大陆网络快速下载：https://1817112769.share.123865.com/123pan/XdYVVv-OxbL

**Target device:** Samsung Galaxy Tab S6 Wi-Fi — SM-T860 (`gts6lwifi`)  
**Base firmware:** T860XXS5DWH1 (vendor/boot/dtbo) + T733XXS9DYF1 (system/product, Android 14 / One UI 6)  
**Build date:** 2026-06-19

---

## Image set

| File | Description |
|------|-------------|
| `BOOT.img` | T860 stock boot image |
| `DTBO.img` | T860 stock dtbo |
| `VBMETA.img` | T860 stock vbmeta — verification & hashtree disabled (flags `0x03`) |
| `VENDOR.img` | T860 vendor, patched for APEX compatibility & four-speaker output |
| `SYSTEM.img` | T733 Android 14 system image, patched through v31 |
| `PRODUCT.img` | T733 Android 14 product image, trimmed to fit T860 PRODUCT partition |
| `TWRP-3.7.0_9-0-gts6lwifi.img` | TWRP 3.7.0 recovery for gts6lwifi |

---

## Flashing

Use **heimdall** (included in this repository):

```sh
./heimdall flash \
  --BOOT BOOT.img \
  --DTBO DTBO.img \
  --VBMETA VBMETA.img \
  --VENDOR VENDOR.img \
  --SYSTEM SYSTEM.img \
  --PRODUCT PRODUCT.img \
  --RECOVERY TWRP-3.7.0_9-0-gts6lwifi.img \
  --no-reboot
```

**After flashing:** boot directly to TWRP (`PWR + VOL-UP + USB`) and wipe data/cache before first system boot if coming from another ROM.

---

## SHA256 checksums

```
714ae657b927ba64c8ee58cc03d16370ad2f05a9c020086df5b2d73488a9ed1f  BOOT.img
864ca763a1362a77aa929c3e2f147baa70196c31401b52f76bf0f879d7eb99ec  DTBO.img
a1ba9abc6295994365cd45c98d6a6dd37f3bcabc5b98260901a0d18e927ec196  PRODUCT.img
99025cacc5ffa7fabd9aaf507bb1a5c400b577b8dc6215ad32a1ea85863c4ef5  SYSTEM.img
cbc6e03563a9229b7034dd775964ef412af94a7074f50b519ce1ccc1fd4f2e16  TWRP-3.7.0_9-0-gts6lwifi.img
24f81a45b1f0d074fab40133fe2da91db4f3c6b5e0d612adecd21c86b62a8afe  VBMETA.img
6075c82a1d6872604bc5c5c62f2540eef2abff42238b75a7605d58753bbf9cb7  VENDOR.img
```

---

## AVB / vbmeta notes

Samsung's SM-T860 bootloader uses a **non-standard big-endian AVB** implementation that rejects standard little-endian vbmeta images. This image set includes the **stock vbmeta with only the flags field modified** (offset 120–123 set to `00 00 00 03` in big-endian), which disables both verification and hashtree checking while preserving the original auth block and descriptor chain integrity.

**Key points:**
- ❌ Do NOT create vbmeta from scratch — Samsung rejects it
- ❌ Do NOT clear the auth block or remove descriptors
- ✅ Only the flags field is changed (`0x00000000` → `0x00000003`)
- ✅ All multi-byte fields remain big-endian as Samsung expects

**Result:** Boots with TWRP, custom system images, and Magisk-patched boot without AVB verification failures.

### Stock vbmeta info (T860XXS5DWH1)

```
Size:       8,320 bytes
Header:     256 bytes
Auth block: 576 bytes (offset 256)
Aux block:  6,976 bytes (offset 832)
Flags:      0x00000003 (verification + hashtree disabled)
```

---

## Partition layout

```
BOOT:     stock boot.img (can be Magisk-patched)
RECOVERY: TWRP 3.7.0
VBMETA:   stock vbmeta (flags = 0x03)
SYSTEM:   T733 One UI 6 Android 14
PRODUCT:  T733 One UI 6 Android 14 (trimmed)
VENDOR:   T860 stock, patched for APEX + four-speaker
DTBO:     T860 stock
```

All partitions not listed (cache, userdata, etc.) are kept stock from T860XXS5DWH1.

---

## See also

- `AVB_FIX.md` for the full technical write-up on Samsung's non-standard AVB and how the vbmeta fix was derived.
- `heimdall` binary (included) for flashing on macOS/Linux.
