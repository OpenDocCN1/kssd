
# Petite V2.2 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy06/pediy6920.htm`](https://www.pediy.com/kssd/pediy06/pediy6920.htm)

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  

【调试环境】：`WinXP`、`Ollydbg V1.10`、`LordPE`、`ImportREC`、`ReloX`

【实例下载】：点击下载  

---

## 脱壳过程

`Petite` 也加密了重定位表，写这篇简单的笔记只是再次推荐一下用 `ReloX` 来修复重定位表。

---

### 一、EP 和 OEP

设置 `Ollydbg` 忽略所有的异常选项，载入 `Petite` 加壳的 `EdrLib.dll`。

```
003D7042     B8 00703D00       mov eax,EdrLib.003D7000
```

//进入 `Ollydbg` 后暂停在这

```
003D7047     68 0E493D00       push EdrLib.003D490E
003D704C     64:FF35 00000000  push dword ptr fs:[0]
003D7053     64:8925 00000000  mov dword ptr fs:[0],esp
```

`Petite` 加壳程序的 `OEP` 非常好找，在 `EP` 处下断：`HE EIP`

如果不想脱壳后修改基址，可以在适当的时机 `Dump`。

下断：`BP GetProcAddress`

中断后取消断点，`Alt+F9` 返回

```
003D4AC4     FF95 94070000     call dword ptr ss:[ebp+794]; kernel32.GetProcAddress
003D4ACA     85C0              test eax,eax                ; kernel32.lstrlenW
```

//返回这里

```
003D4ACC     74 7F             je short EdrLib.003D4B4D
```

//下面的循环是处理输入表的

`Ctrl+S` 在当前位置下搜索命令序列：

```
mov edi,ecx
mov edx,ebp
mov eax,dword ptr ds:[ecx]
test eax,eax
```

找到在 `003D4C60` 处，直接 `F4` 至 `003D4C5A`

```
003D4C54     8DB5 0000C0FF     lea esi,dword ptr ss:[ebp+FFC00000]
003D4C5A     8D8D 00600000     lea ecx,dword ptr ss:[ebp+6000]
```

//这里 `[ebp+6000]` 的 `6000` 是原来重定位表的 `RVA`

```
003D4C60     8BF9              mov edi,ecx
003D4C62     8BD5              mov edx,ebp
003D4C64     8B01              mov eax,dword ptr ds:[ecx]
003D4C66     85C0              test eax,eax
003D4C68     74 1B             je short EdrLib.003D4C85
003D4C6A     3C FF             cmp al,0FF
003D4C6C     75 0B             jnz short EdrLib.003D4C79
```

此时输入表已经处理完毕，程序还没有进行重定位，运行 `LordPE` 完全 `Dump` 出这个 `dll`。

`F9` 运行，中断在刚开始下的硬件断点处：

```
003D7042     E9 82A1FFFF       jmp EdrLib.003D11C9
```

//飞向光明之点巅！

```
003D11C9     55                push ebp
```

//OEP `^O^`

```
003D11CA     8BEC              mov ebp,esp
003D11CC     53                push ebx
003D11CD     8B5D 08           mov ebx,dword ptr ss:[ebp+8]
```

---

### 二、输入表

随便从程序找一个 `API` 调用，如：

```
003D10FD     FF15 20403D00     call dword ptr ds:[3D4020]; kernel32.GetVersion
```

在转存中跟随 `3D4020`，上下看到许多函数地址，很明显的可以找到 `IAT` 开始和结束的地址：

```
003D3FF0   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
003D4000   74 81 EF 77 E8 7C EF 77 49 C4 EF 77 3B D7 EF 77  t 侊 w 鑭飛 I 娘 w;罪 w
…… ……
003D40C0   79 70 3D 00 40 7A 95 7C 00 00 00 00 00 00 00 00  yp=.@z 晐........
```

开始地址＝`003D4000`

结束地址＝`003D40C8`

运行 `ImportREC`。注意：去掉“使用来自磁盘的 `PE` 部首”的选项，选择“重建原始 `FT`”选项！

选中 `Ollydbg` 的 `loaddll.exe` 的进程，然后点“选取 `DLL`”，选择 `EdrLib.dll`，填入 `RVA=00004000`、大小＝`000000C8`，点“`Get Import`”，得到输入表，用追踪层次 `1` 修复无效函数。改 `OEP=000011C9`，`FixDump`！

---

### 三、ReloX：修复重定位表的“通用”简便方法

把 `EdrLib.dll` 复制一份，然后用 `DLL_LoadEX` 载入 `EdrLib.dll` 和复制 `EdrLib.dll`，注意两个 `DLL` 进程的基址。

用 `LordPE` 直接把这 `2` 个 `DLL` 进程 `Dump` 出来，另存为 `1.dll`（`ImageBase=01060000`）和 `2.dll`（`ImageBase=01080000`）

运行 `ReloX`，选择 `1.dll` 和 `2.dll`，分别填入其相应的基址，点 `ComPare`，稍等，分析完毕。

`Fix PE Module`, 提示 `dumped_.dll` 保存成功。`ReloX` 已经自动修正了 `Relocation RVA` 和 `Size`！

如果想删除壳区段，则需要挪移输出表数据，具体操作看《用 `Ollydbg` 手脱 `Neolite` 加壳的 `DLL`》教程。

可以用 `LordPE` 或者 `FileScanner` 优化一下脱壳后的文件。

测试一下，正常运行，脱壳完成了。

---

```
      ,     _/ 
    /| _.-~/            \_      ,        青春都一晌
   ( /~   /              ~.\ |\\    忍把浮名 
   \`\\  \_/                  \  ~\ )  换了破解轻狂
 _-~~~-.)  )\_;;,.            \  //'
 /'\_\\   --~   \ ~~~-  ,;;\_\_\(  (.-~~~-.
 \`~ _(_ ,\_..-\ (     ,;'' /    ~--   /.\_\`
  /~~//'   / \`~\        ) /--..\_, )\_  \`~
  "  \`~"  "      \`"      /~'\`\\    \`\\~~\\
                       "   "   "  "~'  ""
              UnPacked By  :  fly
               2004-12-27 20:00
```

`4dd466d0e1e0cc123afbe504da2462ca`
