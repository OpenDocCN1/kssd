
### P13_uboot_mkimage

```bash
P13_uboot_mkimage() {
  local NEG_LOG=0
  if [[ "$UBOOT_IMAGE" -eq 1 ]]; then
    module_log_init "${FUNCNAME[0]}"
    local IMAGE_NAME=""
    local IMAGE_TYPE=""
    module_title "Uboot image details"
    pre_module_reporter "${FUNCNAME[0]}"
    # 使用mkimage工具读取固件
    mkimage -l "$FIRMWARE_PATH" | tee -a "$LOG_FILE"
    IMAGE_NAME=$(grep "Image Name" "$LOG_FILE" 2>/dev/null | awk '{print $3,$4,$5,$6,$7,$8,$9,$10}' || true)
    IMAGE_TYPE=$(grep "Image Type" "$LOG_FILE" 2>/dev/null | awk '{print $3,$4,$5,$6,$7,$8,$9,$10}' || true)
    write_csv_log "Identifier" "Value"
    write_csv_log "ImageName" "$IMAGE_NAME"
    write_csv_log "ImageType" "$IMAGE_TYPE"
    NEG_LOG=1
    module_end_log "${FUNCNAME[0]}" "$NEG_LOG"
  fi
}
```

### P14_ext_mounter.sh

挂载和提取 extX 映像

`ext_extractor` 函数：

```bash
ext_extractor() {
  local EXT_PATH_="${1:-}"
  local EXTRACTION_DIR_="${2:-}"
  local TMP_EXT_MOUNT="$TMP_DIR"/ext_mount_$RANDOM
  local DIRS_EXT_MOUNT=0
  FILES_EXT_MOUNT=0

  if ! [[ -f "$EXT_PATH_" ]]; then
    print_output "[-] No file for decryption provided"
    return
  fi

  sub_module_title "EXT filesystem extractor"

  mkdir -p "$TMP_EXT_MOUNT" || true
  print_output "[*] Trying to mount $ORANGE$EXT_PATH_$NC to $ORANGE$TMP_EXT_MOUNT$NC directory"
  mount -o ro "$EXT_PATH_" "$TMP_EXT_MOUNT"
  if mount | grep -q ext_mount; then
    print_output "[*] Copying $ORANGE$TMP_EXT_MOUNT$NC to firmware tmp directory ($EXTRACTION_DIR_)"
    mkdir -p "$EXTRACTION_DIR_"
    cp -pri "$TMP_EXT_MOUNT"/* "$EXTRACTION_DIR_"
    print_ln
    print_output "[*] Using the following firmware directory ($ORANGE$EXTRACTION_DIR_$NC) as base directory:"
    find "$EXTRACTION_DIR_" -xdev -maxdepth 1 -ls | tee -a "$LOG_FILE"
    print_ln
    print_output "[*] Unmounting $ORANGE$TMP_EXT_MOUNT$NC directory"

    FILES_EXT_MOUNT=$(find "$EXTRACTION_DIR_" -type f | wc -l)
    DIRS_EXT_MOUNT=$(find "$EXTRACTION_DIR_" -type d | wc -l)
    print_output "[*] Extracted $ORANGE$FILES_EXT_MOUNT$NC files and $ORANGE$DIRS_EXT_MOUNT$NC directories from the firmware image."
    write_csv_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
    write_csv_log "EXT filesystem extractor" "$EXT_PATH_" "$EXTRACTION_DIR_" "$FILES_EXT_MOUNT" "$DIRS_EXT_MOUNT" "NA"
    umount "$TMP_EXT_MOUNT" || true
  fi
  rm -r "$TMP_EXT_MOUNT"
}
```

### P15_ubi_extractor.sh

提取 ubi 文件系统

`ubi_extractor` 函数：

```bash
ubi_extractor() {
  local UBI_PATH_="${1:-}"
  local EXTRACTION_DIR_="${2:-}"
  local UBI_FILE=""
  local UBI_INFO=""
  local UBI_1st_ROUND=""
  local UBI_DATA=""
  local DIRS_UBI_EXT=0
  FILES_UBI_EXT=0
  if ! [[ -f "$UBI_PATH_" ]]; then
    print_output "[-] No file for extraction provided"
    return
  fi

  sub_module_title "UBI filesystem extractor"

  print_output "[*] Extracts UBI firmware image $ORANGE$UBI_PATH_$NC with ${ORANGE}ubireader_extract_images$NC."
  print_output "[*] File details: $ORANGE$(file "$UBI_PATH_" | cut -d ':' -f2-)$NC"

  # 提取ubi镜像
  # 工具仓库：f61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7M7Y4y4H3M7Y4g2A6N6s2c8Q4x3V1k6#2j5X3W2Q4y4h3k6J5k6h3q4V1k6i4t1`.
  ubireader_extract_images -i -v -w -o "$EXTRACTION_DIR_"/ubi_images "$UBI_PATH_" | tee -a "$LOG_FILE" || true
  FILES_UBI_EXT=$(find "$EXTRACTION_DIR_"/ubi_images -type f | wc -l)
  DIRS_UBI_EXT=$(find "$EXTRACTION_DIR_"/ubi_images -type d | wc -l)
  print_output "[*] Extracted $ORANGE$FILES_UBI_EXT$NC files and $ORANGE$DIRS_UBI_EXT$NC directories from the firmware image via UBI extraction round 1."

  print_output "[*] Extracts UBI firmware image $ORANGE$UBI_PATH_$NC with ${ORANGE}ubireader_extract_files$NC."

  ubireader_extract_files -i -v -w -o "$EXTRACTION_DIR_"/ubi_files "$UBI_PATH_" | tee -a "$LOG_FILE" || true
  FILES_UBI_EXT=$(find "$EXTRACTION_DIR_"/ubi_files -type f | wc -l)
  DIRS_UBI_EXT=$(find "$EXTRACTION_DIR_"/ubi_files -type d | wc -l)
  print_output "[*] Extracted $ORANGE$FILES_UBI_EXT$NC files and $ORANGE$DIRS_UBI_EXT$NC directories from the firmware image via UBI extraction round 2."

  # 深度提取，继续查找UBI文件镜像，然后提取
  if [[ -d "$EXTRACTION_DIR_" ]]; then
    mapfile -t UBI_1st_ROUND < <(find "$EXTRACTION_DIR_" -type f -exec file {} \; | grep "UBI image" || true)

    for UBI_DATA in "${UBI_1st_ROUND[@]}"; do
      UBI_FILE=$(safe_echo "$UBI_DATA" | cut -d: -f1)
      UBI_INFO=$(safe_echo "$UBI_DATA" | cut -d: -f2)
      if [[ "$UBI_INFO" == *"UBIfs image"* ]]; then
        sub_module_title "UBIfs deep extraction"
        print_output "[*] Extracts UBIfs firmware image $ORANGE$UBI_PATH_$NC with ${ORANGE}ubireader_extract_files$NC."
        print_output "[*] File details: $ORANGE$(file "$UBI_FILE" | cut -d ':' -f2-)$NC"
        ubireader_extract_files -l -i -w -v -o "$EXTRACTION_DIR_"/UBIfs_extracted "$UBI_FILE" | tee -a "$LOG_FILE" || true
        FILES_UBI_EXT=$(find "$EXTRACTION_DIR_"/UBIfs_extracted -type f | wc -l)
        DIRS_UBI_EXT=$(find "$EXTRACTION_DIR_"/UBIfs_extracted -type d | wc -l)
        print_output "[*] Extracted $ORANGE$FILES_UBI_EXT$NC files and $ORANGE$DIRS_UBI_EXT$NC directories from the firmware image via UBI deep extraction."
      fi
    done

    print_ln
    FILES_UBI_EXT=$(find "$EXTRACTION_DIR_" -type f | wc -l)
    DIRS_UBI_EXT=$(find "$EXTRACTION_DIR_" -type d | wc -l)
    print_output "[*] Extracted $ORANGE$FILES_UBI_EXT$NC files and $ORANGE$DIRS_UBI_EXT$NC directories from the firmware image."
    write_csv_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
    write_csv_log "UBI filesystem extractor" "$UBI_PATH_" "$EXTRACTION_DIR_" "$FILES_UBI_EXT" "$DIRS_UBI_EXT" "NA"
  else
    print_output "[-] First round UBI extractor failed!"
  fi
}
```

### P16_EnGenius_decryptor.sh

用 EnGenius 提取固件镜像

`engenius_enc_extractor` 函数：
