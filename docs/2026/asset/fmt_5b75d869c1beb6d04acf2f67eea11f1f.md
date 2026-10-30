
`P60_firmware_bin_extractor()` 函数：

```bash
P60_firmware_bin_extractor() {
  module_log_init "${FUNCNAME[0]}"
  module_title "Binary firmware deep extractor"
  pre_module_reporter "${FUNCNAME[0]}"
  export DISK_SPACE_CRIT=0
  # If we have not found a linux filesystem we try to do an extraction round on every file multiple times
  if [[ $RTOS -eq 0 ]] ; then
    module_end_log "${FUNCNAME[0]}" 0
    return
  fi
  # 跟前面一样，检查sasquatch链接
  # we need to check if sasquatch is the correct one for binwalk:
  if ! [[ "$(readlink -q -f "$UNBLOB_PATH"/sasquatch)" == "/usr/local/bin/sasquatch_binwalk" ]]; then
    if [[ -L "$UNBLOB_PATH"/sasquatch ]]; then
      rm "$UNBLOB_PATH"/sasquatch
    fi
    ln -s /usr/local/bin/sasquatch_binwalk "$UNBLOB_PATH"/sasquatch || true
  fi
  # 用于检查磁盘空间，如果没超过最大的额外空间，就会开启深度提取
  check_disk_space
  if ! [[ "$DISK_SPACE" -gt "$MAX_EXT_SPACE" ]]; then
    deep_extractor
  else
    print_output "[!] $(date) - Extractor needs too much disk space $DISK_SPACE" "main"
    print_output "[!] $(date) - Ending extraction processes - no deep extraction performed" "main"
    DISK_SPACE_CRIT=1
  fi
  print_ln
  FILES_EXT=$(find "$FIRMWARE_PATH_CP" -xdev -type f | wc -l )
  UNIQUE_FILES=$(find "$FIRMWARE_PATH_CP" "${EXCL_FIND[@]}" -xdev -type f -exec md5sum {} \; 2>/dev/null | sort -u -k1,1 | cut -d\ -f3 | wc -l )
  DIRS_EXT=$(find "$FIRMWARE_PATH_CP" -xdev -type d | wc -l )
  BINS=$(find "$FIRMWARE_PATH_CP" "${EXCL_FIND[@]}" -xdev -type f -exec file {} \; | grep -c "ELF" || true)
  if [[ "$BINS" -gt 0 || "$UNIQUE_FILES" -gt 0 ]]; then
    export LINUX_PATH_COUNTER=0
    linux_basic_identification_helper "$FIRMWARE_PATH_CP"
    print_ln
    print_output "[*] Found $ORANGE$FILES_EXT$NC files ($ORANGE$UNIQUE_FILES$NC unique files) and $ORANGE$DIRS_EXT$NC directories at all."
    print_output "[*] Found $ORANGE$BINS$NC binaries."
    print_output "[*] Additionally the Linux path counter is $ORANGE$LINUX_PATH_COUNTER$NC."
    print_ln
    tree -csh "$FIRMWARE_PATH_CP" | tee -a "$LOG_FILE"
    # now it should be fine to also set the FIRMWARE_PATH ot the FIRMWARE_PATH_CP
    export FIRMWARE_PATH="$FIRMWARE_PATH_CP"
    if [[ "${#ROOT_PATH[@]}" -gt 0 ]] ; then
      write_csv_log "FILES" "UNIQUE_FILES" "DIRS" "Binaries" "LINUX_PATH_COUNTER" "Root PATH detected"
      for R_PATH in "${ROOT_PATH[@]}"; do
        write_csv_log "$FILES_EXT" "$UNIQUE_FILES" "$DIRS_EXT" "$BINS" "$LINUX_PATH_COUNTER" "$R_PATH"
      done
    fi
    backup_var "FILES_EXT" "$FILES_EXT"
  fi
  module_end_log "${FUNCNAME[0]}" "$FILES_EXT"
}
```

### `check_disk_space`函数

用于检查磁盘空间

```bash
check_disk_space() {
  export DISK_SPACE
  DISK_SPACE=$(du -hm "$FIRMWARE_PATH_CP" --max-depth=1 --exclude="proc" 2>/dev/null | awk '{ print $1 }' | sort -hr | head -1 || true)
}
```

### `deep_extractor`函数

深度提取模式

```bash
deep_extractor() {
  sub_module_title "Deep extraction mode"
  FILE_MD5=""
  FILES_BEFORE_DEEP=$(find "$FIRMWARE_PATH_CP" -xdev -type f | wc -l )
  # if we run into the deep extraction mode we always do at least one extraction round:
  if [[ "$DISK_SPACE_CRIT" -eq 0 ]]; then
    print_output "[*] Deep extraction - 1st round"
    print_output "[*] Walking through all files and try to extract what ever possible"
    # 进行深度提取，将在下方给出分析
    deeper_extractor_helper
    detect_root_dir_helper "$FIRMWARE_PATH_CP"
  fi
  # 进行多轮提取
  if [[ $RTOS -eq 1 && "$DISK_SPACE_CRIT" -eq 0 ]]; then
    print_output "[*] Deep extraction - 2nd round"
    print_output "[*] Walking through all files and try to extract what ever possible"
    deeper_extractor_helper
    detect_root_dir_helper "$FIRMWARE_PATH_CP"
  fi
  if [[ $RTOS -eq 1 && "$DISK_SPACE_CRIT" -eq 0 ]]; then
    print_output "[*] Deep extraction - 3rd round"
    print_output "[*] Walking through all files and try to extract what ever possible"
    deeper_extractor_helper
    detect_root_dir_helper "$FIRMWARE_PATH_CP"
  fi
  if [[ $RTOS -eq 1 && "$DISK_SPACE_CRIT" -eq 0 ]]; then
    print_output "[*] Deep extraction - 4th round"
    print_output "[*] Walking through all files and try to extract what ever possible with binwalk matryoshka mode"
    print_output "[*] WARNING: This is the last extraction round that is executed."
    # if we are already that far we do a final matryoshka extraction mode
    deeper_extractor_helper "M"
    detect_root_dir_helper "$FIRMWARE_PATH_CP"
  fi
  FILES_AFTER_DEEP=$(find "$FIRMWARE_PATH_CP" -xdev -type f | wc -l )
  print_output "[*] Before deep extraction we had $ORANGE$FILES_BEFORE_DEEP$NC files, after deep extraction we have now $ORANGE$FILES_AFTER_DEEP$NC files extracted."
}
```

### `deeper_extractor_helper`函数

进行深度提取
