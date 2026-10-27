
**SoftWrap 6.x之二：Inline Patch SoftWrap壳的方法**

``` 
004277F4 C705 3B994500>mov dword ptr [45993B], FCDECAE9 ; 把0045993B Patch
004277FE C605 3F994500>mov byte ptr [45993F], 0FF ; 成jmp 0042780A
00427805 - E9 CA1F0300 jmp 004597D4 ; 跳回原流程
0042780A 05 40FDC304 add eax, 4C3FD40 ; 原代码挪移
0042780F 81FE 54BB4500 cmp esi, 0045BB54 ; esi=45BB54的时候0045BB31才解码开
00427815 75 13 jnz short 0042782A ; 0045BB31没解码就跳回原流程
00427817 C705 31BB4500>mov dword ptr [45BB31], FCBCF9E9 ; 0045BB31解码就Patch 0045BB31
00427821 66:C705 35BB4>mov word ptr [45BB35], 90FF ; 成jmp 0042782F
0042782A - E9 11210300 jmp 00459940 ; 跳回原流程
0042782F C705 E88B0410>mov dword ptr [10048BE8], 1 ; 赋1给[10048BE8],INC后[10048BE8]=2
00427839 - E9 F8420300 jmp 0045BB36 ; 跳回原流程
```

```
BA D2 67 30 00 8D B5 BF 03 00 00 E9 69 19 03 00 BF 76 1C CD AB C7 05 3B 99 45 00 E9 CA DE FC C6 05 3F 99 45 00 FF
E9 CA 1F 03 00 05 40 FD C3 04 81 FE 54 BB 45 00 75 13 C7 05 31 BB 45 00 E9 F9 BC FC 66 C7 05 35 BB 45 00 FF 90 E9 11
21 03 00 C7 05 E8 8B 04 10 01 00 00 00 E9 F8 42 03 00
```

**图文版PDF下载**

[SoftWrap 6.x之二：Inline Patch SoftWrap壳的方法.pdf](https://bbs.kanxue.com/attach-download-5390.htm)

--------------------------------------------------------------------------------

## 【经验总结】

Inline Patch的大致流程就在下面，但是需要大家有足够的耐心和一定的汇编基础，Enjoy！

![](https://bbs.kanxue.com/upload/attach/200704/10794_t5elafp36dwzuhu.png)

**下篇：SoftWrap 6.x之三：IAT修复之Jmp dword ptr ds:[API]**

--------------------------------------------------------------------------------

## 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年04月23日

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.PNG](javascript:void(0);)
  （21.66kb，374次下载）
- [2.PNG](javascript:void(0);)
  （31.08kb，369次下载）
- [3.PNG](javascript:void(0);)
  （14.54kb，370次下载）
- [4.PNG](javascript:void(0);)
  （6.94kb，366次下载）
- [5.PNG](javascript:void(0);)
  （61.57kb，365次下载）
- [6.PNG](javascript:void(0);)
  （3.21kb，361次下载）
- [7.PNG](javascript:void(0);)
  （61.48kb，361次下载）
- [SoftWrap 6.x之二：Inline Patch SoftWrap壳的方法.pdf](javascript:void(0);)
  （197.56kb，61次下载）
- [CodeCaver.rar](javascript:void(0);)
  （275.72kb，106次下载）
- [FractalPC30r4u.Inline Patch.rar](javascript:void(0);)
  （291.89kb，104次下载）
