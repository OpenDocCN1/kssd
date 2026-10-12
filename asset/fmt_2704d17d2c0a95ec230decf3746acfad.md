
在命令行方式下直接启动 `THOCRecog`，后跟一个图片文件路径，通过 `filemon` 发现 `THOCRecog` 同样会在图片路径创建一些文件，但是很快又删除了，怀疑这就是 `THOCRecog` 处理的结果。

如果我跳开 `THOCRecog` 中的删除文件的函数，就可以查看这些文件的内容。

找到 `THOCRecog` 中对 `DeleteFileA` 引用的部分。

```
.text:00401313 loc_401313:                      ; CODE XREF: sub_4010B0+1FD j
.text:00401313                                   ; sub_4010B0+207 j
.text:00401313             mov     edx, [ebp+lpFileName]
.text:00401316             mov     esi, ds:DeleteFileA
.text:0040131C             push    edx           ; lpFileName
.text:0040131D             call    esi ; DeleteFileA
```

将 `00401313` 处的代码改为 `retn`。

观察图片目录，多了 `4` 个文件，其中一个和图片同名的 `.txt` 中存放了识别的结果。

上面的处理达到了效果，但是显得很暴力。

分析该 `DeleteFileA` 所在的函数，可以推测 `THOCRecog` 大概的过程：

1. 根据传入 `.bmp` 文件路径，`THOCRecog` 分析该文件，将分析结果存在图片目录，与图片文件同名的 `.txt` 文件中。
2. `THOCRecog` 读取该 `.txt` 文件并把读取的内存规格为与 `.caj` 约定的格式。
3. `THOCRecog` 将已经规格化的内容存在系统剪切板中。
4. `THOCRecog` 删除 `.txt` 等图片目录下的 `4` 个文件。结束进程。

### 七. 怎么利用

我对 `MFC` 不熟悉，没法逆向解析剪切板数据的相关过程，所以采取下面方法：

```
.text:00401207             test    eax, eax
.text:00401209             jz      short loc_401214
.text:00401209
.text:0040120B             mov     byte ptr [ebp+var_4], 9
.text:0040120F             jmp     loc_40135C
.text:0040120F
.text:00401214 ; ---------------------------------------------------------------------------
.text:00401214
.text:00401214 loc_401214:                      ; CODE XREF: sub_4010B0+159 j
.text:00401214             call    sub_401710
```

将 `.text:00401209             jz      short loc_401214` 改为 `nop` 可以直接跳过上面的 `4` 个过程。

然后用自己的程序读取数据，删除文件……

--------------------------------------------------------------------------------

【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2008 年 05 月 16 日 11:50:48
