# SM-T860 AVB Verification Bypass Fix

## Problem

When flashing a GSI (Generic System Image) or TWRP on the Samsung Galaxy Tab S6 (SM-T860), the bootloader rejects the boot with various vbmeta errors:
- `unsupported AVB version (7)` — wrong endianness in version field
- `invalid vbmeta header (6)` — corrupted/missing auth block
- `Hash of data does not match digest` — partition hash mismatch

## Root Cause

Samsung's bootloader on SM-T860 (T860XXS5DWH1) implements a **non-standard AVB** that:

1. **Uses BIG-ENDIAN for all fields** in the vbmeta header (standard AVB uses little-endian)
2. **Requires a valid authentication block structure** even when verification flags are set to disabled
3. **Does NOT respect** the standard `AVB_VBMETA_IMAGE_FLAGS_VERIFICATION_DISABLED` flag alone — the vbmeta must still be structurally valid

Specifically:
- The `required_libavb_version` field at offset 4 must be `00 00 00 01` (big-endian = 1.0)
- Writing `01 00 00 00` (little-endian = 1) causes Samsung to read it as 16,777,216 → "unsupported AVB version (7)"
- Clearing the auth block causes "invalid vbmeta header (6)" even with flags=3
- Removing descriptors causes structural validation failure

## Solution

**Take the STOCK vbmeta image, and ONLY modify the flags field (offset 120, 4 bytes) from `00 00 00 00` to `00 00 00 03` (big-endian).**

The flags field uses big-endian on Samsung:
- Flag bit 0 (`00 00 00 01` in BE) = `AVB_VBMETA_IMAGE_FLAGS_VERIFICATION_DISABLED`
- Flag bit 1 (`00 00 00 02` in BE) = `AVB_VBMETA_IMAGE_FLAGS_HASHTREE_DISABLED`
- Value `00 00 00 03` = both disabled

### Step-by-step

```python
import lz4.frame

# 1. Decompress stock vbmeta
with lz4.frame.open('vbmeta.img.lz4', 'rb') as f:
    data = bytearray(f.read())

# 2. Change ONLY the flags at offset 120-123 (big-endian)
data[120] = 0x00
data[121] = 0x00
data[122] = 0x00
data[123] = 0x03  # Both verification + hashtree disabled

# 3. Save
with open('vbmeta_disabled.img', 'wb') as f:
    f.write(data)
```

### Flash with heimdall

```bash
heimdall flash --VBMETA vbmeta_disabled.img
```

### Key Principles

1. **NEVER create vbmeta from scratch** — Samsung won't accept it
2. **NEVER clear the auth block** — Samsung validates it structurally
3. **NEVER remove descriptors** — they're part of the chain validation
4. **ONLY change flags** — bit 0 (verification) + bit 1 (hashtree)
5. **Use big-endian** for all multi-byte fields

### Stock vbmeta info (T860XXS5DWH1)

```
Size: 8320 bytes
Header: 256 bytes
Auth block: 576 bytes (offset 256, includes signature + public key)
Aux block: 6976 bytes (offset 832, includes descriptors for: recovery, boot, dtbo, system, product, vendor, vbmeta)
Flags: 0x00000000 → changed to 0x00000003
```

### What the flags do

With flags=0x03, the bootloader:
- ✅ Passes vbmeta header validation (structurally identical to stock)
- ✅ Passes auth block structural validation (signature exists but not enforced)
- ✅ Skips hash verification of all chained partitions (boot, recovery, dtbo, system, product, vendor)
- ✅ Allows booting with TWRP, custom ROMs, Magisk-patched boot

### Working Configuration

```
BOOT:     stock boot.img (or Magisk-patched)
RECOVERY: TWRP 3.7.0
VBMETA:   stock vbmeta with flags=0x03
SYSTEM:   crDroid GSI (or any GSI)
```

All other partitions (vendor, product, dtbo, etc) kept stock from T860XXS5DWH1.
