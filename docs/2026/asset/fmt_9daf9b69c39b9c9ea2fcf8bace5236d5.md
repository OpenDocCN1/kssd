

```bash
# this function is for the first round of binwalk
# in case no other EMBA extractor did something and the
# current firmware file is a file and not multiple files
binwalking() {
    local FIRMWARE_PATH_="${1:-}"
    export OUTPUT_DIR_BINWALK=""

    if ! [[ -f "$FIRMWARE_PATH_" ]]; then
        print_output "[-] No file for extraction provided"
        return
    fi

    sub_module_title "Analyze binary firmware blob with binwalk"

    # 简单地用binwalk分析一下
    print_output "[*] Basic analysis with binwalk"
    binwalk "$FIRMWARE_PATH_" | tee -a "$LOG_FILE"

    print_ln "no_log"

    # we use the original FIRMWARE_PATH for entropy testing, just if it is a file
    if [[ -f $FIRMWARE_PATH_BAK ]] && ! [[ -f "$LOG_DIR"/firmware_entropy.png ]]; then
        print_output "[*] Entropy testing with binwalk ... "

        # we have to change the working directory for binwalk, because everything except the log directory is read-only in
        # Docker container and binwalk fails to save the entropy picture there
        if [[ $IN_DOCKER -eq 1 ]]; then
            cd "$LOG_DIR" || return

            # 用binwalk生成熵图
            print_output "$(binwalk -E -F -J "$FIRMWARE_PATH_BAK")"
            mv "$(basename "$FIRMWARE_PATH_".png)" "$LOG_DIR"/firmware_entropy.png 2>/dev/null || true
            cd /emba || return
        else
            print_output "$(binwalk -E -F -J "$FIRMWARE_PATH_BAK")"
            mv "$(basename "$FIRMWARE_PATH_".png)" "$LOG_DIR"/firmware_entropy.png 2>/dev/null || true
        fi
    fi

    OUTPUT_DIR_BINWALK=$(basename "$FIRMWARE_PATH_")
    OUTPUT_DIR_BINWALK="$FIRMWARE_PATH_CP"/"$OUTPUT_DIR_BINWALK"_binwalk_emba

    print_ln
    print_output "[*] Extracting firmware to directory $ORANGE$OUTPUT_DIR_BINWALK$NC"

    # this is not working in background. I have created a new function that gets executed in the background
    # probably there is a more elegant way
    # binwalk is executed in Matryoshka mode
    binwalk_deep_extract_helper 1 "$FIRMWARE_PATH_" "$OUTPUT_DIR_BINWALK" &
    WAIT_PIDS+=("$!")
    wait_for_extractor
    WAIT_PIDS=( )

    MD5_DONE_DEEP+=("$(md5sum "$FIRMWARE_PATH_" | awk '{print $1}')")
}
```

## `detect_root_dir_helper`函数

通过特殊二进制路径来推测文件系统根目录在哪

|  |  |
| --- | --- |

