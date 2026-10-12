
# UPX 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy06/pediy6721.htm`](https://www.pediy.com/kssd/pediy06/pediy6721.htm)

**【目标程序】**：`UPX` 加壳的 `EdrLib.dll`。附件中还有输入表、重定位数据、`UPXAngela` 以及 `UnPacked` 以供参考。

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：`WinXP`、`Ollydbg1.10C`、`WinHex`、`LordPE`、`UPXAngela`、`ImportREC`

**【实例下载】**：点击此处下载

---

## 一、脱壳过程

其实这篇东东只是《加密与解密》第 2 版中关于 `UPX` 脱壳的 `Ollydbg` 脱壳翻版。

`Ollydbg1.10C` 加 `UPXAngela` 脱 `UPX` 的 DLL 很是方便呀，呵呵。

### 1. 得到重定位表 RVA 和 OEP

不用跟踪，`Ctrl+S` 在当前位置下搜索命令序列：

```assembly
xchg ah,al
rol eax,10
xchg ah,al
add eax,esi
```

找到在 `003B826D` 处，在其上 `mov al,byte ptr ds:[edi]` 的 `003B825E` 处下断，`F9` 运行，断下。

```assembly
003B825E     8A07           mov al,byte ptr ds:[edi]
//EDI=003B7318-当前基址 003B0000＝00007318  ★ 这就是重定位表的 RVA
003B8260     47             inc edi
003B8261     09C0           or eax,eax
003B8263     74 22          je short EdrLib.003B8287
//重定位数据处理完毕则跳转
003B8265     3C EF          cmp al,0EF
003B8267     77 11          ja short EdrLib.003B827A
003B8269     01C3           add ebx,eax
003B826B     8B03           mov eax,dword ptr ds:[ebx]
003B826D     86C4           xchg ah,al//找到这里
003B826F     C1C0 10        rol eax,10
003B8272     86C4           xchg ah,al
003B8274     01F0           add eax,esi
003B8276     8903           mov dword ptr ds:[ebx],eax
003B8278     EB E2          jmp short EdrLib.003B825C
003B827A     24 0F          and al,0F
003B827C     C1E0 10        shl eax,10
003B827F     66:8B07        mov ax,word ptr ds:[edi]
003B8282     83C7 02        add edi,2
003B8285     EB E2          jmp short EdrLib.003B8269
```

在 `003B8287` 处下断，`F9` 运行，断下，重定位数据处理完毕。

当我们在 `003B8287` 处中断下来时，`EDI=003B74EA`，就是重定位表结束的地址。

```assembly
003B8287     61             popad
//此时 EDI=003B74EA  ★
003B8288     E9 3C8FFFFF    jmp EdrLib.003B11C9
//飞向光明之点巅！
```

```assembly
003B11C9     55             push ebp//OEP ★
003B11CA     8BEC           mov ebp,esp
003B11CC     53             push ebx
003B11CD     8B5D 08        mov ebx,dword ptr ss:[ebp+8]
003B11D0     56             push esi
003B11D1     8B75 0C        mov esi,dword ptr ss:[ebp+C]
003B11D4     57             push edi
003B11D5     8B7D 10        mov edi,dword ptr ss:[ebp+10]
003B11D8     85F6           test esi,esi
003B11DA     75 09          jnz short EdrLib.003B11E5
```

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `EdrLib.dll`，然后完整脱壳，得到 `dumped.dll`。

---

## 二、输入表

随便从程序找一个 API 调用，如：

```assembly
003B10FD     FF15 20403B00   call dword ptr ds:[3B4020]; kernel32.GetVersion
```

在转存中跟随 `3B4020`，上下看到许多函数地址，很明显的可以找到 IAT 开始和结束的地址：

```assembly
003B3FF0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
003B4000   1D 51 C4 77 1C 3A C4 77 3E E7 C4 77 CC D2 C4 77  .Q.w.:.w>..w...w
           

003B40B0   CE 7C E5 77 05 74 E5 77 F9 81 E5 77 EB 41 E4 77  .|.w.t.w...w.A.w
003B40C0   66 C8 E5 77 3E 18 F6 77 00 00 00 00 00 00 00 00   f..w>..w........
```

开始地址＝`003B4000`

结束地址＝`003B40C9`

运行 `ImportREC`，注意：去掉“使用来自磁盘的 PE 部首”的选项！

选中 `Ollydbg` 的 `loaddll.exe` 的进程，然后点“选取 DLL”，选择 `EdrLib.dll`，填入 `RVA＝00004000`、`大小＝000000C8`，点“`Get Import`”，得到输入表。改 `OEP=000011C9`，`FixDump`！

---

## 三、重定位表修复 + PE 修正

`UPX` 破坏了重定位表。在第一步我们已经得到重定位表的地址了，用 `WinHex` 打开 `dumped.dll`，复制 `7318`－`74EA` 之间的 16 进制数值，另存为 `fly.bin`

运行 看雪 老师写的辅助修复 `UPX` 加壳 DLL 重定位表的工具 `UPXAngela.exe`，打开 `fly.bin`，很快的提示 `pediy.bin` 文件创建成功！

从 `dumped.dll` 找块空白代码，如自 `UPX1` 段的 `6000` 处开始。够用就行，`pediy.bin` 中的数据长度为 `3B0`。

用 `WinHex` 把 `pediy.bin` 中的 16 进制数值全部复制、写入到 `dumped.dll` 的 `6000` 处。

用 `LordPE` 修正 `dumped.dll` 的重定位表 `RVA=00006000`、`大小＝000003B0`，保存之。

再用 `LordPE` 修正 `dumped.dll` 的基址为 `003B0000`，OK，脱壳完成啦。

---

```
              ,      \_/ 
             /| \_.-~/            \\_      ,        青春都一晌
            ( /~   /               \\~.\_ |\
            \`\\   \_/               \   ~\          忍把浮名 
  _-~~~-.)  \_\_/;;,.               \_  //'
 /'\_,\   --~   \ ~~~-  ,;;\__\(  (.-~~~-.        换了破解轻狂
 \`~ _\( ,_..\   (      ,;'' /    ~--   /.\_\`
  /~~//'   /  \`~\           ![](img/e45fac309594d4f9580d902b0ed40a73.jpg) /--..\_,  ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)  \`
   "  \`"  "      `"       /~'\`\\    \`\\~~
                         "    "   "~'  ""
```

Cracked By 巢水工作坊——fly [OCN][FCG][NUKE][DCM]

2004-05-30 21:21 `4dd466d0e1e0cc123afbe504da2462ca`
