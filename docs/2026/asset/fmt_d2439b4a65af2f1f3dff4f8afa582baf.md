
### P12_avm_freetz_ng_extract.sh

使用`Freetz-NG`提取`AVM`固件映像

`avm_extractor`函数：

```
avm_extractor() {
local AVM_FW_PATH_="${1:-}"
local EXTRACTION_DIR_="${2:-}"
if ! [[ -f "$AVM_FW_PATH_" ]]; then
return
fi
local FRITZ_DIRS=0
local FIT_IMAGES=()
local FIT_IMAGE=""
local RAM_DISKS=()
local RAM_DISK=""
local RAM_DISK_NAME=""
export FRITZ_FILE=0
export FRITZ_VERSION=""

local MODULE_DISABLED=1

if [[ "$MODULE_DISABLED" -eq 1 ]]; then
print_output "[*] Module ${FUNCNAME[0]} is deprecated and will be removed in the future"
return
fi

sub_module_title "AVM freetz-ng firmware extractor"
# read only filesystem bypass:
# 将 freetz-ng 工具链的 .config 文件复制到临时目录中
cp "$EXT_DIR"/freetz-ng/.config "$TMP_DIR"/.config
# 使用 freetz-ng/fwmod 工具从 AVM 固件中提取文件
"$EXT_DIR"/freetz-ng/fwmod -u -i "$TMP_DIR"/.config -d "$EXTRACTION_DIR_" "$AVM_FW_PATH_" | tee -a "$LOG_FILE" || true

if [[ -d "$EXTRACTION_DIR_" ]]; then
# FRITZ 是来自德国 AVM 公司生产的一系列家用网络设备，包括路由器、网关、电话和其他相关产品。
# 统计提取出的文件和文件夹数量
FRITZ_FILES=$(find "$EXTRACTION_DIR_" -type f | wc -l)
FRITZ_DIRS=$(find "$EXTRACTION_DIR_" -type d | wc -l)
# 提取版本信息
FRITZ_VERSION=$(grep "detected firmware version:" "$LOG_FILE" | cut -d ":" -f2- || true)
if [[ -z "$FRITZ_VERSION" ]]; then
FRITZ_VERSION="NA"
else
print_output "[+] Detected Fritz version: $ORANGE$FRITZ_VERSION$NC"
fi

# fitimages are handled here with fitimg - binwalk and unblob are also able to handle these images
# but it is currently more beautiful doing the AVM extraction in one place here
mapfile -t FIT_IMAGES < <(find "$EXTRACTION_DIR_" -type f -name "fit-image")
# 在提取的文件中，该函数会检测是否存在名为 "fit-image" 的文件
# 如果存在则调用 fitimg 工具对其解压缩
# 解压缩结果输出到 EXTRACTION_DIR/fit-image-extraction 目录中
# 并从中提取 RAM_DISK
if [[ "${#FIT_IMAGES[@]}" -gt 0 ]]; then
if [[ -f "$EXT_DIR"/fitimg-0.8/fitimg ]]; then
for FIT_IMAGE in "${FIT_IMAGES[@]}"; do
print_output "[*] Detected fit-image: $ORANGE$FIT_IMAGE$NC"
print_output "[*] Extracting fit-image with fitimg to $ORANGE$EXTRACTION_DIR/fit-image-extraction$NC"
mkdir -p "$EXTRACTION_DIR/fit-image-extraction"
"$EXT_DIR"/fitimg-0.8/fitimg -x "$FIT_IMAGE" -d "$EXTRACTION_DIR"/fit-image-extraction || true
mapfile -t RAM_DISKS < <(find "$EXTRACTION_DIR_" /fit-image-extraction -type f -name "*ramdisk")
print_ln
done
else
print_output "[-] Fitimg installation not available - check your installation"
fi
fi
if [[ "${#RAM_DISKS[@]}" -gt 0 ]]; then
for RAM_DISK in "${RAM_DISKS[@]}"; do
print_output "[*] Detected AVM ramdisk: $ORANGE$RAM_DISK$NC"
RAM_DISK_NAME="$(basename "$RAM_DISK")"
# 对RAM_DISK进行深度提取
binwalk_deep_extract_helper 1 "$RAM_DISK" "$EXTRACTION_DIR_"/fit-image-extraction/"$RAM_DISK_NAME"_binwalk
print_ln
done
fi

if [[ "$FRITZ_FILES" -gt 0 ]]; then
print_ln
print_output "[*] Extracted $ORANGE$FRITZ_FILES$NC files and $ORANGE$FRITZ_DIRS$NC directories from the firmware image."
write_csv_log "Extractor module" "Original file" "extracted file/dir" "file counter" "directory counter" "further details"
write_csv_log "AVM extractor" "$AVM_FW_PATH_" "$EXTRACTION_DIR_" "$FRITZ_FILES" "$FRITZ_DIRS" "$FRITZ_VERSION"
export DEEP_EXTRACTOR=1
MD5_DONE_DEEP+=( "$(md5sum "$AVM_FW_PATH_" | awk '{print $1}')")

if [[ -z "${FW_VENDOR:-}" ]]; then
FW_VENDOR="AVM"
backup_var "FW_VENDOR" "$FW_VENDOR"
fi
if [[ -z "${FW_VERSION:-}" && "$FRITZ_VERSION" != "NA" ]]; then
FW_VERSION="$FRITZ_VERSION"
fi
fi
fi
}
```

### P13_uboot_mkimage.sh

显示`Uboot`映像的内部

`P13_uboot_mkimage`函数：
