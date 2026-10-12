
# PECompact V2.40 简便脱壳二法

> 原文：[`www.pediy.com/kssd/pediy06/pediy6676.htm`](https://www.pediy.com/kssd/pediy06/pediy6676.htm)

下载页面：[`www.hanzify.org/?Go=Show::List&ID=6189`](http://www.hanzify.org/?Go=Show::List&ID=6189)

- 软件大小： 1.3MB
- 所属分类： 加密解密
- 上传时间： 2004-08-08 00:44:47
- 软件汉化： `FeiXJ`
- 运行环境： `WIN9X` / `WINME` / `WINNT` / `WIN2000PRO` / `WINXP`
- 软件简介： 这是一个可执行文件的压缩（加壳）工具。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg` `V1.10`、`PEiD`、`LordPE`、`ImportREC`

## 脱壳过程

看到有兄弟问，所以简单写几句。

准备扔西红柿的兄弟可否考虑扔西瓜过来给我？呵呵

用 `PECompact` `V2.40` 汉化版给 `Win98` 的记事本加壳，`PECompact` 的压缩效果已经不错了。

设置 `Ollydbg` 忽略所有的异常选项。

### 一、第一种方法

```assembly
004010CC    B8 E8E54000        mov eax,Notepad.0040E5E8//进入 OD 后暂停在这
004010D1    50               push eax
004010D2    64:FF35 00000000 push dword ptr fs:[0]
004010D9    64:8925 00000000 mov dword ptr fs:[0],esp
004010E0    33C0             xor eax,eax
004010E2    8908             mov dword ptr ds:[eax],ecx
```

下断：`BP` `VirtualFree`

中断后取消断点，`Ctrl+F9` 两次，返回 `0040E695` 处

```assembly
0040E693    FFD7             call edi
0040E695    8985 23120010    mov dword ptr ss:[ebp+10001223],eax; Notepad.<ModuleEntryPoint>
0040E69B    8BF0             mov esi,eax
0040E69D    59               pop ecx
0040E69E    5A               pop edx
0040E69F    03CA             add ecx,edx
0040E6A1    68 00800000      push 8000
0040E6A6    6A 00            push 0
0040E6A8    57               push edi
0040E6A9    FF11             call dword ptr ds:[ecx]; kernel32.VirtualFree
0040E6AB    8BC6             mov eax,esi
0040E6AD    5A               pop edx
0040E6AE    5E               pop esi
0040E6AF    5F               pop edi
0040E6B0    59               pop ecx
0040E6B1    5B               pop ebx
0040E6B2    5D               pop ebp
0040E6B3    FFE0             jmp eax; Notepad.<ModuleEntryPoint>
```

//飞向光明之巅！ ^O^

### 二、第二种方法

```assembly
004010CC    B8 E8E54000        mov eax,Notepad.0040E5E8//进入 OD 后暂停在这
004010D1    50               push eax
004010D2    64:FF35 00000000 push dword ptr fs:[0]
004010D9    64:8925 00000000 mov dword ptr fs:[0],esp
004010E0    33C0             xor eax,eax
004010E2    8908             mov dword ptr ds:[eax],ecx
```

看到这个 `004010CC` 的入口，各位兄弟该有想法了吧？呵呵

`PECompact` `V1.84` 等旧版都是：

```assembly
JMP   XXXXXXXX
PUSH  XXXXXXXX//OEP 的 RVA
```

而新版索性就直接以原 `OEP` 为壳入口地址了

下断：`HE` `EIP`

`F9` 运行，直接中断在 `OEP` 处了

```assembly
004010CC    55               push ebp//OEP
004010CD    8BEC             mov ebp,esp
004010CF    83EC 44          sub esp,44
004010D2    56               push esi
004010D3    FF15 E4634000    call dword ptr ds:[4063E4]; kernel32.GetCommandLineA
```

拿出 `LordPE` 来 `DUMP` 进程，`ImportREC` 修复输入表吧。

注：第 2 种方法不适用于 `Delphi` 程序，看来很多壳对 `Delphi` 都特别“照顾”呀。

---

, /_
/| \_.-~/ \_ , 青春都一晌
( /~ / \~-.\ |
\`\\ \_/ \ ~\) 忍把浮名
\_-~~~-.) )\_//;;,. \ \//'
/'\_,\ --~ \ ~~~- ,;;\_( (.-~~~-. 换了破解轻狂
\`~ \_( ,\_..\\ ( ,;'' / ~-- /.\_
/~~//' /' \`~\) ) /--..\_, )\_ \`~
" \`~" " \`" /~'\`\\ \`\\~~\
" " "~' ""

Cracked By 巢水工作坊——`fly` `[FCG][NUKE][DCM]`

2004-08-11 00:00

## 【破文标题】`PECompact` `V2.40` 简便脱壳 - `NOTEPADw2k.EXE`

## 【破文作者】`cuituo`

## 【作者邮箱】`cuituo@163.com`

## 【所属组织】你的组织

## 【软件名称】`PECompact` `V2.40` 压缩 `notepad`

## 【破解工具】`FuckALL.exe`

## 【保护方式】`PECompact` `V2.40`

## 【破解难度】简单

---

破解声名：学习 `fly` 大侠的破文解压 `w2k` 的 `notepad`，见笑

---

## 【破解分析】

设置 `Ollydbg` 忽略所有的异常选项。

```assembly
01001000 >  B8 F4140101      MOV EAX,NOTEPADw.010114F4
01001005     50             PUSH EAX
01001006     64:FF35 0000000>PUSH DWORD PTR FS:[0]
0100100D     64:8925 0000000>MOV DWORD PTR FS:[0],ESP
01001014     33C0           XOR EAX,EAX
01001016     8908           MOV DWORD PTR DS:[EAX],ECX
01001018     50             PUSH EAX
01001019     45             INC EBP
0100101A     43             INC EBX
0100101B     6F             OUTS DX,DWORD PTR ES:[EDI]              ; I/O 命令
0100101C     6D             INS DWORD PTR ES:[EDI],DX               ; I/O 命令
0100101D     70 61          JO SHORT NOTEPADw.01001080
0100101F     637432 00      ARPL WORD PTR DS:[EDX+ESI],SI
01001023     60             PUSHAD
```

上面全是一样的，下断：`BP` `VirtualFree`

`F9`,

中断在此处，取消断点

```assembly
77E64E64 >  55             PUSH EBP
77E64E65     8BEC           MOV EBP,ESP
77E64E67     FF75 10        PUSH DWORD PTR SS:[EBP+10]
77E64E6A     FF75 0C        PUSH DWORD PTR SS:[EBP+C]
77E64E6D     FF75 08        PUSH DWORD PTR SS:[EBP+8]
77E64E70     6A FF          PUSH -1
77E64E72     E8 04000000    CALL kernel32.VirtualFreeEx
77E64E77     5D             POP EBP
77E64E78     C2 0C00        RETN 0C        F4 执行到此,F8
```

```assembly
00280330     8B46 0C        MOV EAX,DWORD PTR DS:[ESI+C]
00280333     03C7           ADD EAX,EDI
00280335     5D             POP EBP
00280336     5E             POP ESI
00280337     5F             POP EDI
00280338     5B             POP EBX
00280339     C3             RETN           F4 执行到此,F8
```

```assembly
010115A1     8985 23120010  MOV DWORD PTR SS:[EBP+10001223],EAX      ; NOTEPADw.01006420
010115A7     8BF0           MOV ESI,EAX
010115A9     59             POP ECX
010115AA     5A             POP EDX
010115AB     03CA           ADD ECX,EDX
010115AD     68 00800000    PUSH 8000
010115B2     6A 00          PUSH 0
010115B4     57             PUSH EDI
010115B5     FF11           CALL DWORD PTR DS:[ECX]
010115B7     8BC6           MOV EAX,ESI
010115B9     5A             POP EDX
010115BA     5E             POP ESI
010115BB     5F             POP EDI
010115BC     59             POP ECX
010115BD     5B             POP EBX
010115BE     5D             POP EBP
010115BF     FFE0           JMP EAX       F4 执行到此,F8
```

//飞向光明之巅！ ^O^ (`FLY` 语)

```assembly
01006420     55             PUSH EBP
01006421     8BEC           MOV EBP,ESP
01006423     6A FF          PUSH -1
01006425     68 88180001    PUSH NOTEPADw.01001888
0100642A     68 D0650001    PUSH NOTEPADw.010065D0               ; JMP to msvcrt._except_handler3
0100642F     64:A1 00000000 MOV EAX,DWORD PTR FS:[0]
01006435     50             PUSH EAX
01006436     64:8925 0000000>MOV DWORD PTR FS:[0],ESP
0100643D     83C4 98        ADD ESP,-68
01006440     53             PUSH EBX
01006441     56             PUSH ESI
01006442     57             PUSH EDI
```

万分感谢，如果不是这个断点，我转了半天了愣没转出来呀！佩服佩服~~~

---

## 【总结】

我的稍有不同，`F9` 直接运行，没出现异常！~

2004-8-13 6:16:48

`4dd466d0e1e0cc123afbe504da2462ca`
