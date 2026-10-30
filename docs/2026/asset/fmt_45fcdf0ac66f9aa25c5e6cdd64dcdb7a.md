
# fw_bin_detector

```bash
fw\_bin\_detector() {
# 固件路径
local CHECK\_FILE="${1:-}"
local FILE\_BIN\_OUT=""
local DLINK\_ENC\_CHECK=""
local QNAP\_ENC\_CHECK=""
local AVM\_CHECK=0
local UEFI\_CHECK=0
set\_p02\_default\_exports
FILE\_BIN\_OUT=$(file "$CHECK\_FILE")
# 输出第一行用于检查是否为DLINK公司产品
DLINK\_ENC\_CHECK=$(hexdump -C "$CHECK\_FILE" | head -1 || true)
# 从固件中所有的字符串中检查AVM正则出现的行数。用于判断固件是否为AVM公司产品。
AVM\_CHECK=$(strings "$CHECK\_FILE" | grep -c "AVM GmbH .\*. All rights reserved.\|(C) Copyright .\* AVM" || true)
# 搜索字符串并只显示包含指定字符串的结果，用于判断是否为QNAP加密文件系统
QNAP\_ENC\_CHECK=$(binwalk -y "qnap encrypted" "$CHECK\_FILE")
# we are running binwalk on the file to analyze the output afterwards:
binwalk "$CHECK\_FILE" > "$TMP\_DIR"/s02\_binwalk\_output.txt
UEFI\_CHECK=$(grep -c "UEFI" "$TMP\_DIR"/s02\_binwalk\_output.txt || true)
UEFI\_CHECK=$(( "$UEFI\_CHECK" + "$(grep -c "UEFI" "$CHECK\_FILE" || true)" ))
if [[ -f "$KERNEL\_CONFIG" ]] && [[ "$KERNEL" -eq 1 ]]; then
# we set the FIRMWARE\_PATH to the kernel config path if we have only -k parameter
if [[ "$(md5sum "$KERNEL\_CONFIG" | awk '{print $1}')" == "$(md5sum "$FIRMWARE\_PATH" | awk '{print $1}')" ]]; then
print\_output "[+] Identified Linux kernel configuration file"
write\_csv\_log "kernel config" "yes" "NA"
export SKIP\_PRE\_CHECKERS=1
return
fi
fi
if [[ "$UEFI\_CHECK" -gt 0 ]]; then
print\_output "[+] Identified possible UEFI firmware - using fwhunt-scan vulnerability scanning module"
export UEFI\_DETECTED=1
UEFI\_AMI\_CAPSULE=$(grep -c "AMI.\*EFI.\*capsule" "$TMP\_DIR"/s02\_binwalk\_output.txt || true)
if [[ "$UEFI\_AMI\_CAPSULE" -gt 0 ]]; then
print\_output "[+] Identified possible UEFI-AMI capsule firmware - using capsule extractors"
fi
write\_csv\_log "UEFI firmware detected" "yes" "NA"
fi
if [[ "$AVM\_CHECK" -gt 0 ]] || [[ "$FW\_VENDOR" == \*"AVM"\* ]]; then
print\_output "[+] Identified AVM firmware - using AVM extraction module"
export AVM\_DETECTED=1
write\_csv\_log "AVM firmware detected" "yes" "NA"
fi
# if we have a zip, tgz, tar archive we are going to use the patools extractor
if [[ "$FILE\_BIN\_OUT" == \*"gzip compressed data"\* || "$FILE\_BIN\_OUT" == \*"Zip archive data"\* || \
"$FILE\_BIN\_OUT" == \*"POSIX tar archive"\* || "$FILE\_BIN\_OUT" == \*"ISO 9660 CD-ROM filesystem data"\* || \
"$FILE\_BIN\_OUT" == \*"7-zip archive data"\* || "$FILE\_BIN\_OUT" == \*"XZ compressed data"\* || \
"$FILE\_BIN\_OUT" == \*"bzip2 compressed data"\* ]]; then
# as the AVM images are also zip files we need to bypass it here:
if [[ "$AVM\_DETECTED" -ne 1 ]]; then
print\_output "[+] Identified gzip/zip/tar/iso/xz/bzip2 archive file - using patools extraction module"
export PATOOLS\_INIT=1
write\_csv\_log "basic compressed (patool)" "yes" "NA"
fi
fi
if [[ "$FILE\_BIN\_OUT" == \*"QEMU QCOW2 Image"\* ]]; then
print\_output "[+] Identified Qemu QCOW image - using QCOW extraction module"
export QCOW\_DETECTED=1
write\_csv\_log "Qemu QCOW firmware detected" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"VMware4 disk image"\* ]]; then
print\_output "[+] Identified VMWware VMDK archive file - using VMDK extraction module"
export VMDK\_DETECTED=1
write\_csv\_log "VMDK" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"UBI image"\* ]]; then
print\_output "[+] Identified UBI filesystem image - using UBI extraction module"
export UBI\_IMAGE=1
write\_csv\_log "UBI filesystem" "yes" "NA"
fi
# 识别DLINK固件
if [[ "$DLINK\_ENC\_CHECK" == \*"SHRS"\* ]]; then
print\_output "[+] Identified D-Link SHRS encrpyted firmware - using D-Link extraction module"
export DLINK\_ENC\_DETECTED=1
write\_csv\_log "D-Link SHRS" "yes" "NA"
fi
if [[ "$DLINK\_ENC\_CHECK" =~ 00000000\ \ 00\ 00\ 00\ 00\ 00\ 00\ 0.\ ..\ \ 00\ 00\ 0.\ ..\ 31\ 32\ 33\ 00 ]]; then
print\_output "[+] Identified EnGenius encrpyted firmware - using EnGenius extraction module"
export ENGENIUS\_ENC\_DETECTED=1
write\_csv\_log "EnGenius encrypted" "yes" "NA"
fi
if [[ "$DLINK\_ENC\_CHECK" =~ 00000000\ \ 00\ 00\ 00\ 00\ 00\ 00\ 01\ 01\ \ 00\ 00\ 0.\ ..\ 33\ 2e\ 3[89]\ 2e ]]; then
print\_output "[+] Identified EnGenius encrpyted firmware - using EnGenius extraction module"
export ENGENIUS\_ENC\_DETECTED=1
write\_csv\_log "EnGenius encrypted" "yes" "NA"
fi
if [[ "$DLINK\_ENC\_CHECK" == \*"encrpted\_img"\* ]]; then
print\_output "[+] Identified D-Link encrpted\_img encrpyted firmware - using D-Link extraction module"
export DLINK\_ENC\_DETECTED=2
write\_csv\_log "D-Link encrpted\_img encrypted" "yes" "NA"
fi
# 识别u-boot固件
if [[ "$FILE\_BIN\_OUT" == \*"u-boot legacy uImage"\* ]]; then
print\_output "[+] Identified u-boot firmware - using u-boot module"
export UBOOT\_IMAGE=1
write\_csv\_log "Uboot image" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"Unix Fast File system [v2]"\* ]]; then
print\_output "[+] Identified UFS filesytem - using UFS filesytem extraction module"
export BSD\_UFS=1
write\_csv\_log "BSD UFS filesystem" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"Linux rev 1.0 ext2 filesystem data"\* ]]; then
print\_output "[+] Identified Linux ext2 filesytem - using EXT filesytem extraction module"
export EXT\_IMAGE=1
write\_csv\_log "EXT2 filesystem" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"Linux rev 1.0 ext3 filesystem data"\* ]]; then
print\_output "[+] Identified Linux ext3 filesytem - using EXT filesytem extraction module"
export EXT\_IMAGE=1
write\_csv\_log "EXT3 filesystem" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"Linux rev 1.0 ext4 filesystem data"\* ]]; then
print\_output "[+] Identified Linux ext4 filesytem - using EXT filesytem extraction module"
export EXT\_IMAGE=1
write\_csv\_log "EXT4 filesystem" "yes" "NA"
fi
if [[ "$QNAP\_ENC\_CHECK" == \*"QNAP encrypted firmware footer , model"\* ]]; then
print\_output "[+] Identified QNAP encrpyted firmware - using QNAP extraction module"
export QNAP\_ENC\_DETECTED=1
write\_csv\_log "QNAP encrypted filesystem" "yes" "NA"
fi
# probably we need to take a deeper look to identify the gpg compressed firmware files better.
# Currently this detection mechanism works quite good on the known firmware images
if [[ "$DLINK\_ENC\_CHECK" =~ 00000000\ \ a3\ 01\ ]]; then
GPG\_CHECK="$(gpg --list-packets "$FIRMWARE\_PATH" | grep "compressed packet:")"
if [[ "$GPG\_CHECK" == \*"compressed packet: algo="\* ]]; then
print\_output "[+] Identified GPG compressed firmware - using GPG extraction module"
export GPG\_COMPRESS=1
write\_csv\_log "GPG compressed firmware" "yes" "NA"
fi
fi
if [[ "$DLINK\_ENC\_CHECK" == \*"CrAU"\* ]]; then
print\_output "[+] Identified Android OTA payload.bin update file - using Android extraction module"
export ANDROID\_OTA=1
write\_csv\_log "Android OTA update" "yes" "NA"
fi
if [[ "$FILE\_BIN\_OUT" == \*"openssl enc'd data with salted password"\* ]]; then
print\_output "[+] Identified OpenSSL encrypted file - trying OpenSSL module for Foscam firmware"
export OPENSSL\_ENC\_DETECTED=1
write\_csv\_log "OpenSSL encrypted" "yes" "NA"
fi
# This check is currently only tested on one firmware - further tests needed:
if [[ "$DLINK\_ENC\_CHECK" =~ 00000000\ \ 62\ 67\ 6e\ 00\ 00\ 00\ 00\ 00\ \ 00\ 00\ 00\ b9\ 01\ ]]; then
print\_output "[+] Identified Buffalo encrpyted firmware - using Buffalo extraction module"
export BUFFALO\_ENC\_DETECTED=1
write\_csv\_log "Buffalo encrypted" "yes" "NA"
fi
if [[ "$(basename "$CHECK\_FILE")" =~ .\*\.ri ]] && [[ "$FILE\_BIN\_OUT" == \*"data"\* ]]; then
# ri files are usually used by zyxel
if [[ $(find "$LOG\_DIR"/firmware -name "$(basename -s .ri "$CHECK\_FILE")".bin | wc -l) -gt 0 ]]; then
# if we find a bin file with the same name then it is a Zyxel firmware image
print\_output "[+] Identified ZyXel encrpyted ZIP firmware - using ZyXel extraction module"
export ZYXEL\_ZIP=1
write\_csv\_log "ZyXel encrypted ZIP" "yes" ""
fi
fi
print\_ln
}
```

## P05_patools_init.sh

用 `patools` 工具提取 `zip`、`tar`、`tgz`
