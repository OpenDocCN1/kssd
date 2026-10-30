
```bash
gpg\_decompress\_extractor() {
local GPG\_FILE\_PATH\_="${1:-}"
local EXTRACTION\_FILE\_="${2:-}"
if ! [[ -f "$GPG\_FILE\_PATH\_" ]]; then
print\_output "[-] No file for extraction provided"
return
fi
sub\_module\_title "GPG compressed firmware extractor"
# 列出包列表并解密
gpg --list-packets "$GPG\_FILE\_PATH\_" 2>/dev/null | tee -a "$LOG\_FILE"
gpg --decrypt "$GPG\_FILE\_PATH\_" > "$EXTRACTION\_FILE\_" || true
print\_ln
if [[ -f "$EXTRACTION\_FILE\_" ]]; then
print\_output "[+] Extracted GPG compressed firmware file to $ORANGE$EXTRACTION\_FILE\_$NC"
export FIRMWARE\_PATH="$EXTRACTION\_FILE\_"
backup\_var "FIRMWARE\_PATH" "$FIRMWARE\_PATH"
print\_ln
print\_output "[\*] Firmware file details: $ORANGE$(file "$EXTRACTION\_FILE\_")$NC"
write\_csv\_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
write\_csv\_log "GPG decompression" "$GPG\_FILE\_PATH\_" "$EXTRACTION\_FILE\_" "1" "NA" "NA"
else
print\_output "[-] Extraction of GPG compressed firmware file failed"
fi
}
```

### `P18_qnap_decryptor.sh`

从QNAP中提取加密固件映像

首先调用 `qnap_enc_extractor` 函数：

```bash
qnap\_enc\_extractor() {
local QNAP\_ENC\_PATH\_="${1:-}"
local EXTRACTION\_FILE\_="${2:-}"
export QNAP=0
if ! [[ -f "$QNAP\_ENC\_PATH\_" ]]; then
print\_output "[-] No file for decryption provided"
return
fi
sub\_module\_title "QNAP encrypted firmware extractor"
hexdump -C "$QNAP\_ENC\_PATH\_" | head | tee -a "$LOG\_FILE" || true
if [[ -f "$EXT\_DIR"/PC1 ]]; then
print\_ln
print\_output "[\*] Decrypting QNAP firmware with leaked key material ..."
print\_ln
# 如果有PC1工具，则用PC1工具提取
"$EXT\_DIR"/PC1 d QNAPNASVERSION4 "$QNAP\_ENC\_PATH\_" "$EXTRACTION\_FILE\_" | tee -a "$LOG\_FILE"
else
print\_output "[-] QNAP decryptor not found - check your installation"
fi
print\_ln
if [[ -f "$EXTRACTION\_FILE\_" && "$(file "$EXTRACTION\_FILE\_")" == \*"gzip compressed data"\* ]]; then
print\_output "[+] Decrypted QNAP firmware file to $ORANGE$EXTRACTION\_FILE\_$NC"
MD5\_DONE\_DEEP+=( "$(md5sum "$QNAP\_ENC\_PATH\_" | awk '{print $1}')" )
export FIRMWARE\_PATH="$EXTRACTION\_FILE\_"
backup\_var "FIRMWARE\_PATH" "$FIRMWARE\_PATH"
# 如果提取出来是gzip文件，那再提取一次，使用下面的函数
export QNAP=1
print\_ln
print\_output "[\*] Firmware file details: $ORANGE$(file "$EXTRACTION\_FILE\_")$NC"
print\_ln
write\_csv\_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
write\_csv\_log "QNAP decryptor" "$QNAP\_ENC\_PATH\_" "$EXTRACTION\_FILE\_" "1" "NA" "gzip compressed data"
if [[ -z "${FW\_VENDOR:-}" ]]; then
FW\_VENDOR="QNAP"
backup\_var "FW\_VENDOR" "$FW\_VENDOR"
fi
else
print\_output "[-] Decryption of QNAP firmware file failed"
fi
}
```

`qnap_extractor`函数：
