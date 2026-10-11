
# 方正 Apabi Reader 多文档格式阅读器 1.53

> 原文：[`www.pediy.com/kssd/pediy05/pediy50014.htm`](https://www.pediy.com/kssd/pediy05/pediy50014.htm)

## 方正 Apabi Reader 多文档格式阅读器拷贝文字限制解除

程序版本：`1.53` 补丁版 `build 1037`

软件下载：`http://www.apabi.com/download/readerDL.htm`

Crack by `fwnl`

工具：`TRW2000`, `WINHEX`

### 软件说明

一个可以阅读 `CEB`, `PDF`, `TXT`, `HTML` 的阅读器，最用之处就是看 `PDF` 文档可用金山词霸抓词和可以拷贝加了防拷贝密码保护的 `PDF` 文档，但是每次只能拷 `64` 个字，真是太少了，恰好我要把一个加了防拷密码的 `PDF` 文档里的内容拷出来，在网上找了好久才发现只有这个东西可以办到，哈哈，所以解其限制就是我下面要做的事了。

### 操作步骤

每当选取超过 `64` 个字拷贝时就弹出警告对话框，所以从这个对话框下手。

1.  用 `TRW2000` load 主程序，`F5` 返回后用 `Apabi Reader` 随便打开一段超过 `64` 个字的文字，全部选定文字后，`Ctrl+N` 叫出 `TRW2000` 后 `bpx lockmytask`，按 `F5` 返回，按右键选拷贝文字，程序中断后 `bd *` 关闭断点。
2.  按一下 `f11`，然后不断按 `f12` 直到警告窗口出现。
3.  点 `0k` 就到了 `TRW2000` 的画面，这时向上看一下程序发现 `0041E60C` 可能是警告对话框 call，于是设断 `41b66f` 后，`F5` 返回主程序，又选取全文，然后右键点拷贝文字，程序立刻断在 `0041E60C`。

```
0167:0041E609 50               PUSH    EAX
0167:0041E60A 8BCE             MOV     ECX,ESI
0167:0041E60C E83F020000       CALL    0041E850   ===========警告对话框 CALL，F8 进入
0167:0041E611 8D4C240C         LEA     ECX,[ESP+0C]
0167:0041E615 C78424CC000000FF MOV     DWORD [ESP+CC],FFFFFFFF
0167:0041E620 E81B4F0100       CALL    00433540
0167:0041E625 8B8C24C4000000   MOV     ECX,[ESP+C4]
0167:0041E62C 5F               POP     EDI
0167:0041E62D 5E               POP     ESI
0167:0041E62E 64890D00000000   MOV     [FS:00],ECX
0167:0041E635 81C4C8000000     ADD     ESP,C8
0167:0041E63B C20400           RET     04
0167:0041E63E 90               NOP
0167:0041E63F 90               NOP
```

进入 `0041E60C` CALL 后来到：

```
0167:0041E875 8BCF             MOV     ECX,EDI
0167:0041E877 740A             JZ      0041E883
0167:0041E879 E822000000       CALL    0041E8A0   ===========警告对话框 CALL，F8 进入
0167:0041E87E 5F               POP     EDI
0167:0041E87F 5E               POP     ESI
0167:0041E880 C20400           RET     04
0167:0041E883 E808060000       CALL    0041EE90
0167:0041E888 5F               POP     EDI
```

进入 `0041E879` CALL 后来到：

```
0167:0041E92A 8BC8             MOV     ECX,EAX
0167:0041E92C FF520C           CALL    NEAR [EDX+0C] ===========警告对话框 CALL，F8 进入
0167:0041E92F 85C0             TEST    EAX,EAX
0167:0041E931 C7442420FFFFFFFF MOV     DWORD [ESP+20],FFFFFFFF
0167:0041E939 8D4C2428         LEA     ECX,[ESP+28]
0167:0041E93D 7518             JNZ     0041E957
0167:0041E93F E8CC090000       CALL    0041F310
```

进 `0041E92C` CALL 后来到，不久发现到了 `SmartReader.dll` 里：

```
0167:028781DB 8D4DE8           LEA     ECX,[EBP-18]
0167:028781DE E80DE2FFFF       CALL    028763F0   ===========警告对话框 CALL，F8 进入
0167:028781E3 E922010000       JMP     0287830A
0167:028781E8 8D4DE8           LEA     ECX,[EBP-18]
0167:028781EB E8E0DFFFFF       CALL    028761D0
0167:028781F0 E915010000       JMP     0287830A
```

进 `0167:028781DE` CALL 后来到：

```
0167:028764A4 8B44240C         MOV     EAX,[ESP+0C]
0167:028764A8 BF80000000       MOV     EDI,80     ===>这里就是字数比较
0167:028764AD 3BC7             CMP     EAX,EDI
0167:028764AF 7E6F             JNG     02876520   ===>这里跳就 ok 了
0167:028764B1 8B0DA8E58902     MOV     ECX,[0289E5A8]
0167:028764B7 894C2410         MOV     [ESP+10],ECX
0167:028764BB 6831750000       PUSH    DWORD 7531
0167:028764C0 8D4C2414         LEA     ECX,[ESP+14]
0167:028764C4 C744244000000000 MOV     DWORD [ESP+40],00
```

### 补丁修改

code on 发现 `28764AF` 处代码是 `7E6f`，要它跳就改成 `EB6f`，由于这段代码是在 `SmartReader.dll` 里，于是用 `winhex` 在 `Apabi Reader` 的安装目录下找到 `SmartReader.dll` 打开，查 `BF800000003BC77E6F8B0DA8E58902`，改为 `BF800000003BC7EB6F8B0DA8E58902`。

这样就解除了拷贝文字 `64` 个的限制，不过拷出来的东西最好复制在写字板，不要用记事本。

### 结语

不知有没有比这个更好的读 `PDF` 的东西，如果谁知道请告诉我一声，不要说 `Adobe Acrobat` 哟，这个东太烂了，限制太多，在精华 `3` 看到一篇文章说可以改 `Adobe Acrobat`，可我看了 `X*N` 次也没看懂。那位知道改的大侠一定要告诉我一下 (`fwnl@sina.com`)，先谢谢了。

`fwnl`
`2002.3.24`
`长沙 4dd466d0e1e0cc123afbe504da2462ca`
