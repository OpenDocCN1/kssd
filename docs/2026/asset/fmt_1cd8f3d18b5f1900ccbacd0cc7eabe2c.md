
```bash
fw_bin_detector() {
  # 固件路径
  local CHECK_FILE="${1:-}"
  local FILE_BIN_OUT=""
  local DLINK_ENC_CHECK=""
  local QNAP_ENC_CHECK=""
  local AVM_CHECK=0
  local UEFI_CHECK=0

  set_p02_default_exports

  FILE_BIN_OUT=$(file "$CHECK_FILE")

  # 输出第一行用于检查是否为DLINK公司产品
  DLINK_ENC_CHECK=$(hexdump -C "$CHECK_FILE" | head -1 || true)

  # 从固件中所有的字符串中检查AVM正则出现的行数。用于判断固件是否为AVM公司产品。
  AVM_CHECK=$(strings "$CHECK_FILE" | grep -c "AVM GmbH .*. All rights reserved.\|(C) Copyright .* AVM" || true)

  # 搜索字符串并只显示包含指定字符串的结果，用于判断是否为QNAP加密文件系统
  QNAP_ENC_CHECK=$(binwalk -y "qnap encrypted" "$CHECK_FILE")
  # we are running binwalk on the file to analyze the output afterwards:
  binwalk "$CHECK_FILE" > "$TMP_DIR"/s02_binwalk_output.txt
  UEFI_CHECK=$(grep -c "UEFI" "$TMP_DIR"/s02_binwalk_output.txt || true)
  UEFI_CHECK=$(( "$UEFI_CHECK" + "$(grep -c "UEFI" "$CHECK_FILE" || true)" ))

  if [[ -f "$KERNEL_CONFIG" ]] && [[ "$KERNEL" -eq 1 ]]; then
    # we set the FIRMWARE_PATH to the kernel config path if we have only -k parameter
    if [[ "$(md5sum "$KERNEL_CONFIG" | awk '{print $1}')" == "$(md5sum "$FIRMWARE_PATH" | awk '{print $1}')" ]]; then
      print_output "[+] Identified Linux kernel configuration file"
      write_csv_log "kernel config" "yes" "NA"
      export SKIP_PRE_CHECKERS=1
      return
    fi
  fi

  if [[ "$UEFI_CHECK" -gt 0 ]]; then
    print_output "[+] Identified possible UEFI firmware - using fwhunt-scan vulnerability scanning module"
    export UEFI_DETECTED=1
    UEFI_AMI_CAPSULE=$(grep -c "AMI.*EFI.*capsule" "$TMP_DIR"/s02_binwalk_output.txt || true)
    if [[ "$UEFI_AMI_CAPSULE" -gt 0 ]]; then
      print_output "[+] Identified possible UEFI-AMI capsule firmware - using capsule extractors"
    fi
    write_csv_log "UEFI firmware detected" "yes" "NA"
  fi

  if [[ "$AVM_CHECK" -gt 0 ]] || [[ "$FW_VENDOR" == *"AVM"* ]]; then
    print_output "[+] Identified AVM firmware - using AVM extraction module"
    export AVM_DETECTED=1
    write_csv_log "AVM firmware detected" "yes" "NA"
  fi

  # if we have a zip, tgz, tar archive we are going to use the patools extractor
  if [[ "$FILE_BIN_OUT" == *"gzip compressed data"* || "$FILE_BIN_OUT" == *"Zip archive data"* || \
    "$FILE_BIN_OUT" == *"POSIX tar archive"* || "$FILE_BIN_OUT" == *"ISO 9660 CD-ROM filesystem data"* || \
    "$FILE_BIN_OUT" == *"7-zip archive data"* || "$FILE_BIN_OUT" == *"XZ compressed data"* || \
    "$FILE_BIN_OUT" == *"bzip2 compressed data"* ]]; then
    # as the AVM images are also zip files we need to bypass it here:
    if [[ "$AVM_DETECTED" -ne 1 ]]; then
      print_output "[+] Identified gzip/zip/tar/iso/xz/bzip2 archive file - using patools extraction module"
      export PATOOLS_INIT=1
      write_csv_log "basic compressed (patool)" "yes" "NA"
    fi
  fi

  if [[ "$FILE_BIN_OUT" == *"QEMU QCOW2 Image"* ]]; then
    print_output "[+] Identified Qemu QCOW image - using QCOW extraction module"
    export QCOW_DETECTED=1
    write_csv_log "Qemu QCOW firmware detected" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"VMware4 disk image"* ]]; then
    print_output "[+] Identified VMWware VMDK archive file - using VMDK extraction module"
    export VMDK_DETECTED=1
    write_csv_log "VMDK" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"UBI image"* ]]; then
    print_output "[+] Identified UBI filesystem image - using UBI extraction module"
    export UBI_IMAGE=1
    write_csv_log "UBI filesystem" "yes" "NA"
  fi

  # 识别DLINK固件
  if [[ "$DLINK_ENC_CHECK" == *"SHRS"* ]]; then
    print_output "[+] Identified D-Link SHRS encrpyted firmware - using D-Link extraction module"
    export DLINK_ENC_DETECTED=1
    write_csv_log "D-Link SHRS" "yes" "NA"
  fi

  if [[ "$DLINK_ENC_CHECK" =~ 00000000\ \ 00\ 00\ 00\ 00\ 00\ 00\ 0.\ ..\ \ 00\ 00\ 0.\ ..\ 31\ 32\ 33\ 00 ]]; then
    print_output "[+] Identified EnGenius encrpyted firmware - using EnGenius extraction module"
    export ENGENIUS_ENC_DETECTED=1
    write_csv_log "EnGenius encrypted" "yes" "NA"
  fi

  if [[ "$DLINK_ENC_CHECK" =~ 00000000\ \ 00\ 00\ 00\ 00\ 00\ 00\ 01\ 01\ \ 00\ 00\ 0.\ ..\ 33\ 2e\ 3[89]\ 2e ]]; then
    print_output "[+] Identified EnGenius encrpyted firmware - using EnGenius extraction module"
    export ENGENIUS_ENC_DETECTED=1
    write_csv_log "EnGenius encrypted" "yes" "NA"
  fi

  if [[ "$DLINK_ENC_CHECK" == *"encrpted_img"* ]]; then
    print_output "[+] Identified D-Link encrpted_img encrpyted firmware - using D-Link extraction module"
    export DLINK_ENC_DETECTED=2
    write_csv_log "D-Link encrpted_img encrypted" "yes" "NA"
  fi

  # 识别u-boot固件
  if [[ "$FILE_BIN_OUT" == *"u-boot legacy uImage"* ]]; then
    print_output "[+] Identified u-boot firmware - using u-boot module"
    export UBOOT_IMAGE=1
    write_csv_log "Uboot image" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"Unix Fast File system [v2]"* ]]; then
    print_output "[+] Identified UFS filesytem - using UFS filesytem extraction module"
    export BSD_UFS=1
    write_csv_log "BSD UFS filesystem" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"Linux rev 1.0 ext2 filesystem data"* ]]; then
    print_output "[+] Identified Linux ext2 filesytem - using EXT filesytem extraction module"
    export EXT_IMAGE=1
    write_csv_log "EXT2 filesystem" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"Linux rev 1.0 ext3 filesystem data"* ]]; then
    print_output "[+] Identified Linux ext3 filesytem - using EXT filesytem extraction module"
    export EXT_IMAGE=1
    write_csv_log "EXT3 filesystem" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"Linux rev 1.0 ext4 filesystem data"* ]]; then
    print_output "[+] Identified Linux ext4 filesytem - using EXT filesytem extraction module"
    export EXT_IMAGE=1
    write_csv_log "EXT4 filesystem" "yes" "NA"
  fi

  if [[ "$QNAP_ENC_CHECK" == *"QNAP encrypted firmware footer , model"* ]]; then
    print_output "[+] Identified QNAP encrpyted firmware - using QNAP extraction module"
    export QNAP_ENC_DETECTED=1
    write_csv_log "QNAP encrypted filesystem" "yes" "NA"
  fi

  # probably we need to take a deeper look to identify the gpg compressed firmware files better.
  # Currently this detection mechanism works quite good on the known firmware images
  if [[ "$DLINK_ENC_CHECK" =~ 00000000\ \ a3\ 01\  ]]; then
    GPG_CHECK="$(gpg --list-packets "$FIRMWARE_PATH" | grep "compressed packet:")"
    if [[ "$GPG_CHECK" == *"compressed packet: algo="* ]]; then
      print_output "[+] Identified GPG compressed firmware - using GPG extraction module"
      export GPG_COMPRESS=1
      write_csv_log "GPG compressed firmware" "yes" "NA"
    fi
  fi

  if [[ "$DLINK_ENC_CHECK" == *"CrAU"* ]]; then
    print_output "[+] Identified Android OTA payload.bin update file - using Android extraction module"
    export ANDROID_OTA=1
    write_csv_log "Android OTA update" "yes" "NA"
  fi

  if [[ "$FILE_BIN_OUT" == *"openssl enc'd data with salted password"* ]]; then
    print_output "[+] Identified OpenSSL encrypted file - trying OpenSSL module for Foscam firmware"
    export OPENSSL_ENC_DETECTED=1
    write_csv_log "OpenSSL encrypted" "yes" "NA"
  fi

  # This check is currently only tested on one firmware - further tests needed:
  if [[ "$DLINK_ENC_CHECK" =~ 00000000\ \ 62\ 67\ 6e\ 00\ 00\ 00\ 00\ 00\ \ 00\ 00\ 00\ b9\ 01\  ]]; then
    print_output "[+] Identified Buffalo encrpyted firmware - using Buffalo extraction module"
    export BUFFALO_ENC_DETECTED=1
    write_csv_log "Buffalo encrypted" "yes" "NA"
  fi

  if [[ "$(basename "$CHECK_FILE")" =~ .*\.ri ]] && [[ "$FILE_BIN_OUT" == *"data"* ]]; then
    # ri files are usually used by zyxel
    if [[ $(find "$LOG_DIR"/firmware -name "$(basename -s .ri "$CHECK_FILE")".bin | wc -l) -gt 0 ]]; then
      # if we find a bin file with the same name then it is a Zyxel firmware image
      print_output "[+] Identified ZyXel encrpyted ZIP firmware - using ZyXel extraction module"
      export ZYXEL_ZIP=1
      write_csv_log "ZyXel encrypted ZIP" "yes" ""
    fi
  fi

  print_ln
}
```
