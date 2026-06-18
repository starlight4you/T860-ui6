# SM-T860 One UI 6 Android 14 v31 Images

Target: Samsung Galaxy Tab S6 Wi-Fi SM-T860 / gts6lwifi

Image set:

- `BOOT.img`: T860 stock boot
- `DTBO.img`: T860 stock dtbo
- `VBMETA.img`: T860 stock vbmeta with verification/hashtree disabled
- `VENDOR.img`: T860 vendor patched for APEX compatibility and four-speaker output
- `SYSTEM.img`: T733 Android 14 system patched through v31
- `PRODUCT.img`: T733 Android 14 product, trimmed to fit T860 PRODUCT
- `TWRP-3.7.0_9-0-gts6lwifi.img`: TWRP recovery

Suggested flash command:

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

After flashing, boot directly to TWRP and wipe data/cache before first system boot if coming from another build.
