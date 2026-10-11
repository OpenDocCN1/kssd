
# 用脚本、补区段脱 Asprotect SKE 2.x 壳笔记【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-562.htm`](https://www.pediy.com/kssd/pediy08/pediy8-562.htm)

【文章标题】: 用脚本、补区段脱 Asprotect SKE 2.x 壳笔记  
【作者】: machenglin  
【软件名称】: ccproje 5.13  
【下载地址】: 自己搜索下载  
【加壳方式】: Asprotect SKE 2.x  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

首先感谢 [一蓑烟雨] 论坛 fly 兄弟，[看雪论坛] 给我们提供了这个学习平台。

Volx 大侠的脚本确实是非常成功的，非常感谢他的辛勤劳动，这是我用他的脚本练习过程中的笔记，结合 SYSCOM 大侠的补区段大法，完成 Asprotect SKE 2.x 壳的过程，我尽量的记录详细一点，试图使初学脱壳者能理解容易些，请各位大侠不要笑话我罗嗦。

```
00401000 c>  68 01D0DE00        push ccproje.00DED001               ; OD 入口处
00401005      E8 01000000        call ccproje.0040100B
0040100A      C3                 retn
0040100B      C3                 retn
0040100C      2B02               sub eax,dword ptr ds:[edx]
0040100E      37                 aaa
0040100F      8D77 F6            lea esi,dword ptr ds:[edi-A]
00401012  -  E1 E8              loopde short ccproje.00400FFC
-------------------------------------------------------------------------------------------------
```

`Aspr2.XX_IATfixer` 脚本停在 OEP 了。

```
016C039C      E8 5FFC0600        call 01730000                        ; OEP=00040190C
016C03A1      50                 push eax
016C03A2      E9 3B060000        jmp 016C09E2
016C03A7      68 85056C01        push 16C0585
016C03AC      E8 4FFC0600        call 01730000
016C03B1      52                 push edx
016C03B2      E9 98000000        jmp 016C044F
016C03B7      43                 inc ebx
016C03B8      E9 5B010000        jmp 016C0518
016C03BD      F2:                prefix repne:
016C03BE      EB 01              jmp short 016C03C1
016C03C0      F0:83C4 08         lock add esp,8                       ; 不允许锁定前缀
```

按 `Alt+l`，查看硬件断点 1 位于 `014FCCB4`。

`IATstartaddr`: `00A4115C`
`IATsize`: `11F8`
`OEP`: `016C039C` - `400000` = `012C039C`
`RVA`: `00A4115C` - `400000` = `0064115C`

Volx 大侠的脚本已经解决了 Asprotect SKE 2.x 壳的诸多问题，手动修复 `stolen code`，确实很完美，但本人功底太浅，OEP 的前面部分基本还能蒙上，后来就陷在壳里了，还跳不出来，还是等修炼 N 段时间再尝试吧。

SYSCOM 大侠的教程，采用补区段的方法很适合我们菜鸟，用 `lordpe` 区域转存，`dump` 出程序中 `stolen code` 和 `VM` 区段，如果不清楚那些是 `stolen code` 和 `VM` 区段，一看 OEP 部分在哪个区段，二看壳用到哪几个区段，不妨多 `dump` 些区段备用。

### VM Address Size 列表

```
014D0000          47000    <-ASProtect 解密 CODE 区段
01520000          14000    <-ASProtect 资料 DATA 区段
016C0000           2000    <-OEP   Stolen Code
016D0000           2000    <-M1    Stolen Code
016E0000           2000    <-M1    Stolen Code
016F0000           1000    <-M2    Stolen Code
01700000           1000    <-M2    Stolen Code
01710000           1000    <-M2    Stolen Code
01720000           1000    <-M3    Stolen Code
01730000           1000    <-M3    Stolen Code
01740000           1000    <-M3    Stolen Code
```

这里提醒一下，把 `dump` 出的区段附加在 `dumped_` 后面时，所有 `VM Address` 要减去基址，如：

`014D0000` - `400000` = `10D0000`。

把 `dump` 出的区段附加完成后，用 `lordpe` 修复 `PE Header`。

重新在 `OD` 中载入程序，开始处理壳的自校验。

### 处理壳自校验

感谢 SYSCOM 大侠的教程，他是这样描述：

```
//////////////////////////////////////////////////////////////////////////////////////////////
Route CHECK,算是壳的自我检查，它是由 A,B 两数值,作互减运算。
A=GetCurrentThreadID
B=CALL Route Address
运算后 ...
B=B-A
当你脱壳之后,B=会发生错误 ERROR 111 ，所以我们只要,抓出正确的 CALL Route Address，就可以通过 CHECK SUM ,也就是在 [ESP+58],的 STACK 位址。所以我们使用
MOV EAX,[ESP+58] ,来还原 B 值 +5 后修正 B 值 Address
9F70CE-MOV  EAX,[EAX+34]
9F70D1-CALL EAX               =>  GetCurrentThreadID
9FD0D3-SUB  [EBP+C],EAX      =>  B=B-A
9FD0D6-MOV  EAX,[ENP+C]
//////////////////////////////////////////////////////////////////////////////////////////////
```

```
014F8A58    /EB 01             jmp short dumped_1.014F8A5B            ; ①↓014F8A5B
014F8A5A    |698B 73308B7B 14A1>imul ecx,dword ptr ds:[ebx+7B8B3073],37>
014F8A64     50                 push eax
014F8A65     018B 4034FFD0      add dword ptr ds:[ebx+D0FF3440],ecx
014F8A6B     2945 0C            sub dword ptr ss:[ebp+C],eax
014F8A6B     2945 0C            sub dword ptr ss:[ebp+C],eax
014F8A6E     8B45 0C            mov eax,dword ptr ss:[ebp+C]
014F8A71     2B43 18            sub eax,dword ptr ds:[ebx+18]
014F8A74     2B43 68            sub eax,dword ptr ds:[ebx+68]
014F8A77     8945 FC            mov dword ptr ss:[ebp-4],eax
014F8A7A     8D43 24            lea eax,dword ptr ds:[ebx+24]
014F8A7D     8945 F8            mov dword ptr ss:[ebp-8],eax
014F8A80     85FF               test edi,edi
014F8A82     76 38              jbe short dumped_1.014F8ABC
014F8A84     EB 01              jmp short dumped_1.014F8A87
014F8A86     C7                 ???                                   ; 未知命令
014F8A87     8B45 F8            mov eax,dword ptr ss:[ebp-8]
014F8A8A     0FB600             movzx eax,byte ptr ds:[eax]
014F8A8D     8B5483 40          mov edx,dword ptr ds:[ebx+eax*4+40]
014F8A91     8BC6               mov eax,esi
014F8A93     FFD2               call edx
014F8A95     3B45 FC            cmp eax,dword ptr ss:[ebp-4]
014F8A98     75 1A              jnz short dumped_1.014F8AB4
014F8A9A     8B45 10            mov eax,dword ptr ss:[ebp+10]
014F8A9D     50                 push eax
014F8A9E     8B45 14            mov eax,dword ptr ss:[ebp+14]
014F8AA1     50                 push eax
014F8AA2     E8 19FAFFFF        call dumped_1.014F84C0
014F8AA7     50                 push eax
014F8AA8     8BCE               mov ecx,esi
014F8AAA     8B55 18            mov edx,dword ptr ss:[ebp+18]
014F8AAD     8BC3               mov eax,ebx
014F8AAF     E8 D4FDFFFF        call dumped_1.014F8888
014F8AB4     4F                 dec edi
014F8AB5     0373 6C            add esi,dword ptr ds:[ebx+6C]
014F8AB8     85FF               test edi,edi
014F8ABA   ^ 77 CB             ja short dumped_1.014F8A87
014F8ABC     68 D88A4F01        push dumped_1.014F8AD8                ; ASCII "111"
014F8AC1     E8 66C3FEFF        call dumped_1.014E4E2C
```

由 `014F8A58` 跳到这里。

```
014F8A5B     8B73 30            mov esi,dword ptr ds:[ebx+30]          ; dumped_1.016C10F9
014F8A5E     8B7B 14            mov edi,dword ptr ds:[ebx+14]
014F8A61     A1 F0375001        mov eax,dword ptr ds:[15037F0]
014F8A66     8B40 34            mov eax,dword ptr ds:[eax+34]          ; 从这里开始修改。
014F8A69     FFD0               call eax
014F8A6B     2945 0C            sub dword ptr ss:[ebp+C],eax
014F8A6E     8B45 0C            mov eax,dword ptr ss:[ebp+C]
014F8A71     2B43 18            sub eax,dword ptr ds:[ebx+18]
014F8A74     2B43 68            sub eax,dword ptr ds:[ebx+68]
014F8A77     8945 FC            mov dword ptr ss:[ebp-4],eax
```

二进制：

```
90 90 8B 44 24 58 83 E8 05 90 90
```

修改后的代码，保存文件。

```
014F8A5B     8B73 30            mov esi,dword ptr ds:[ebx+30]          ; dumped_1.016C10F9
014F8A5E     8B7B 14            mov edi,dword ptr ds:[ebx+14]
014F8A61     A1 F0375001        mov eax,dword ptr ds:[15037F0]
014F8A66     90                 nop
014F8A67     90                 nop
014F8A68     8B4424 58          mov eax,dword ptr ss:[esp+58]
014F8A6C     83E8 05            sub eax,5
014F8A6F     90                 nop
014F8A70     90                 nop
014F8A71     2B43 18            sub eax,dword ptr ds:[ebx+18]
014F8A74     2B43 68            sub eax,dword ptr ds:[ebx+68]
```
