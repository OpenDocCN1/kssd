
```bash
P13\_uboot\_mkimage() {
local NEG\_LOG=0
if [[ "$UBOOT\_IMAGE" -eq 1 ]]; then
module\_log\_init "${FUNCNAME[0]}"
local IMAGE\_NAME=""
local IMAGE\_TYPE=""
module\_title "Uboot image details"
pre\_module\_reporter "${FUNCNAME[0]}"
# 使用mkimage工具读取固件
mkimage -l "$FIRMWARE\_PATH" | tee -a "$LOG\_FILE"
IMAGE\_NAME=$(grep "Image Name" "$LOG\_FILE" 2>/dev/null | awk '{print $3,$4,$5,$6,$7,$8,$9,$10}' || true)
IMAGE\_TYPE=$(grep "Image Type" "$LOG\_FILE" 2>/dev/null | awk '{print $3,$4,$5,$6,$7,$8,$9,$10}' || true)
write\_csv\_log "Identifier" "Value"
write\_csv\_log "ImageName" "$IMAGE\_NAME"
write\_csv\_log "ImageType" "$IMAGE\_TYPE"
NEG\_LOG=1
module\_end\_log "${FUNCNAME[0]}" "$NEG\_LOG"
fi
}
```

### P14\_ext\_mounter.sh

挂载和提取extX映像

`ext_extractor`函数：

```bash
ext\_extractor() {
local EXT\_PATH\_="${1:-}"
local EXTRACTION\_DIR\_="${2:-}"
local TMP\_EXT\_MOUNT="$TMP\_DIR""/ext\_mount\_$RANDOM"
local DIRS\_EXT\_MOUNT=0
FILES\_EXT\_MOUNT=0
if ! [[ -f "$EXT\_PATH\_" ]]; then
print\_output "[-] No file for decryption provided"
return
fi
sub\_module\_title "EXT filesystem extractor"
mkdir -p "$TMP\_EXT\_MOUNT" || true
print\_output "[\*] Trying to mount $ORANGE$EXT\_PATH\_$NC to $ORANGE$TMP\_EXT\_MOUNT$NC directory"
mount -o ro "$EXT\_PATH\_" "$TMP\_EXT\_MOUNT"
if mount | grep -q ext\_mount; then
print\_output "[\*] Copying $ORANGE$TMP\_EXT\_MOUNT$NC to firmware tmp directory ($EXTRACTION\_DIR\_)"
mkdir -p "$EXTRACTION\_DIR\_"
cp -pri "$TMP\_EXT\_MOUNT"/\* "$EXTRACTION\_DIR\_"
print\_ln
print\_output "[\*] Using the following firmware directory ($ORANGE$EXTRACTION\_DIR\_$NC) as base directory:"
find "$EXTRACTION\_DIR\_" -xdev -maxdepth 1 -ls | tee -a "$LOG\_FILE"
print\_ln
print\_output "[\*] Unmounting $ORANGE$TMP\_EXT\_MOUNT$NC directory"
FILES\_EXT\_MOUNT=$(find "$EXTRACTION\_DIR\_" -type f | wc -l)
DIRS\_EXT\_MOUNT=$(find "$EXTRACTION\_DIR\_" -type d | wc -l)
print\_output "[\*] Extracted $ORANGE$FILES\_EXT\_MOUNT$NC files and $ORANGE$DIRS\_EXT\_MOUNT$NC directories from the firmware image."
write\_csv\_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
write\_csv\_log "EXT filesystem extractor" "$EXT\_PATH\_" "$EXTRACTION\_DIR\_" "$FILES\_EXT\_MOUNT" "$DIRS\_EXT\_MOUNT" "NA"
umount "$TMP\_EXT\_MOUNT" || true
fi
rm -r "$TMP\_EXT\_MOUNT"
}
```

### P15\_ubi\_extractor.sh

提取ubi文件系统

`ubi_extractor`函数：

```bash
ubi\_extractor() {
local UBI\_PATH\_="${1:-}"
local EXTRACTION\_DIR\_="${2:-}"
local UBI\_FILE=""
local UBI\_INFO=""
local UBI\_1st\_ROUND=""
local UBI\_DATA=""
local DIRS\_UBI\_EXT=0
FILES\_UBI\_EXT=0
if ! [[ -f "$UBI\_PATH\_" ]]; then
print\_output "[-] No file for extraction provided"
return
fi
sub\_module\_title "UBI filesystem extractor"
print\_output "[\*] Extracts UBI firmware image $ORANGE$UBI\_PATH\_$NC with ${ORANGE}ubireader\_extract\_images$NC."
print\_output "[\*] File details: $ORANGE$(file "$UBI\_PATH\_" | cut -d ':' -f2-)$NC"
# 提取ubi镜像
# 工具仓库：https://github.com/jrspruitt/ubi\_reader
ubireader\_extract\_images -i -v -w -o "$EXTRACTION\_DIR\_"/ubi\_images "$UBI\_PATH\_" | tee -a "$LOG\_FILE" || true
FILES\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_"/ubi\_images -type f | wc -l)
DIRS\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_"/ubi\_images -type d | wc -l)
print\_output "[\*] Extracted $ORANGE$FILES\_UBI\_EXT$NC files and $ORANGE$DIRS\_UBI\_EXT$NC directories from the firmware image via UBI extraction round 1."
print\_output "[\*] Extracts UBI firmware image $ORANGE$UBI\_PATH\_$NC with ${ORANGE}ubireader\_extract\_files$NC."
ubireader\_extract\_files -i -v -w -o "$EXTRACTION\_DIR\_"/ubi\_files "$UBI\_PATH\_" | tee -a "$LOG\_FILE" || true
FILES\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_"/ubi\_files -type f | wc -l)
DIRS\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_"/ubi\_files -type d | wc -l)
print\_output "[\*] Extracted $ORANGE$FILES\_UBI\_EXT$NC files and $ORANGE$DIRS\_UBI\_EXT$NC directories from the firmware image via UBI extraction round 2."
# 深度提取，继续查找UBI文件镜像，然后提取
if [[ -d "$EXTRACTION\_DIR\_" ]]; then
mapfile -t UBI\_1st\_ROUND < <(find "$EXTRACTION\_DIR\_" -type f -exec file {} \; | grep "UBI image" || true)
for UBI\_DATA in "${UBI\_1st\_ROUND[@]}"; do
UBI\_FILE=$(safe\_echo "$UBI\_DATA" | cut -d: -f1)
UBI\_INFO=$(safe\_echo "$UBI\_DATA" | cut -d: -f2)
if [[ "$UBI\_INFO" == \*"UBIfs image"\* ]]; then
sub\_module\_title "UBIfs deep extraction"
print\_output "[\*] Extracts UBIfs firmware image $ORANGE$UBI\_PATH\_$NC with ${ORANGE}ubireader\_extract\_files$NC."
print\_output "[\*] File details: $ORANGE$(file "$UBI\_FILE" | cut -d ':' -f2-)$NC"
ubireader\_extract\_files -l -i -w -v -o "$EXTRACTION\_DIR\_"/UBIfs\_extracted "$UBI\_FILE" | tee -a "$LOG\_FILE" || true
FILES\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_"/UBIfs\_extracted -type f | wc -l)
DIRS\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_"/UBIfs\_extracted -type d | wc -l)
print\_output "[\*] Extracted $ORANGE$FILES\_UBI\_EXT$NC files and $ORANGE$DIRS\_UBI\_EXT$NC directories from the firmware image via UBI deep extraction."
fi
done
print\_ln
FILES\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_" -type f | wc -l)
DIRS\_UBI\_EXT=$(find "$EXTRACTION\_DIR\_" -type d | wc -l)
print\_output "[\*] Extracted $ORANGE$FILES\_UBI\_EXT$NC files and $ORANGE$DIRS\_UBI\_EXT$NC directories from the firmware image."
write\_csv\_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
write\_csv\_log "UBI filesystem extractor" "$UBI\_PATH\_" "$EXTRACTION\_DIR\_" "$FILES\_UBI\_EXT" "$DIRS\_UBI\_EXT" "NA"
else
print\_output "[-] First round UBI extractor failed!"
fi
}
```

### P16\_EnGenius\_decryptor.sh

用EnGenius提取固件镜像

`engenius_enc_extractor`函数：

```bash
engenius\_enc\_extractor() {
local ENGENIUS\_ENC\_PATH\_="${1:-}"
local EXTRACTION\_FILE\_="${2:-}"
if ! [[ -f "$ENGENIUS\_ENC\_PATH\_" ]]; then
print\_output "[-] No file for decryption provided"
return
fi
sub\_module\_title "EnGenius encrypted firmware extractor"
hexdump -C "$ENGENIUS\_ENC\_PATH\_" | head | tee -a "$LOG\_FILE" || true
if [[ -f "$EXT\_DIR"/engenius-decrypt.py ]]; then
# 运行解密脚本
python3 "$EXT\_DIR"/engenius-decrypt.py "$ENGENIUS\_ENC\_PATH\_" > "$EXTRACTION\_FILE\_"
else
print\_output "[-] Decryptor not found - check your installation"
fi
print\_ln
if [[ -f "$EXTRACTION\_FILE\_" ]]; then
print\_output "[+] Decrypted EnGenius firmware file to $ORANGE$EXTRACTION\_FILE\_$NC"
export FIRMWARE\_PATH="$EXTRACTION\_FILE\_"
MD5\_DONE\_DEEP+=( "$(md5sum "$ENGENIUS\_ENC\_PATH\_" | awk '{print $1}')" )
print\_ln
print\_output "[\*] Firmware file details: $ORANGE$(file "$EXTRACTION\_FILE\_")$NC"
write\_csv\_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
write\_csv\_log "EnGenius decryptor" "$ENGENIUS\_ENC\_PATH\_" "$EXTRACTION\_FILE\_" "1" "NA" "NA"
if [[ -z "${FW\_VENDOR:-}" ]]; then
FW\_VENDOR="EnGenius"
backup\_var "FW\_VENDOR" "$FW\_VENDOR"
fi
else
print\_output "[-] Decryption of EnGenius firmware file failed"
fi
}
```

### P17\_gpg\_decompress.sh

提取没有加密的gpg压缩的固件镜像

`gpg_decompress_extractor`函数：
