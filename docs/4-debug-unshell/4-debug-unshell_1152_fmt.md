
# 北京预算 2002 版及北京房修 2003

> 原文：[`www.pediy.com/kssd/pediy06/pediy6421.htm`](https://www.pediy.com/kssd/pediy06/pediy6421.htm)

## 巧脱 VFP&EXE 内存型 1.72 壳——造价大师：北京预算 2002 版及北京房修 2003 估价表

### 基本信息

- **下载页面**：`http://www.yusuan.com/down/SoftView.asp?SoftID=44`
- **运行环境**：`WinXP`, `Win2000`, `NT`, `WinME`, `Win9X`
- **授权方式**：共享软件
- **软件大小**：7.32 MB
- **软件等级**：*****
- **整理时间**：2003-5-1 (`admin`)

**【软件简介】**：依据北京市 2002 年发布的最新的工程预算定额及 2003 北京房修估价表，能进行建筑、装饰、水电安装、煤气、空调工程、市政、桥梁、园林绿化及房修、改造工程概预算的编制，定额手册查询，工程量计算规则查询、工料机换算，混凝土换算，未计价材料、设备自动提示，钢筋模版自动换算，工料机分析统计，主材、设备统计，分项工程统计，三材统计等各种统计报表，并能进行报表打印，报表设计等。既能按照传统预算编制方法，又能按照量价分离和清单报价的方法进行新定额预算编制，同时可以兼容其他软件数据文件,读取工程量计算软件数据。

**【作者声明】**：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：`Win98`、`Ollydbg` 1.09、`PEiD`、`LordPE`、`WinFile`

---

## 脱壳过程

### 初步分析

`BJGS.EXE` 用 `PEiD` 看是 `UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo` 壳。用 `UPX` 脱壳提示：

```
upx: E:试炼场北京预~1BJGS.EXE: CantUnpackException: file is modified/hacked/protected; take care!!!
```

提示是变形 `UPX`，经过 `modified/hacked/protected` 处理啦。于是手脱，上当啦！:-(

用 `Hex Workshop` 看才发现是 老王的 `VFP&EXE` 内存型 `1.72` 加密的！呵呵，老王的 `VFP&EXE` 给许多 `VFP` 程序提供了加密。说起 `VFP` 程序的脱壳与解密绝对不能不提起一个人：`Aming`！大侠 `Aming` 可谓是 `VFP` 程序杀手！他的 文件型加密 `VFP` 程序终结者 `1.50` 精简版（`Build 960`）是 `VFP&EXE` 的克星！

### UNVFP&EXE 1.50 Simple Version 使用说明

可以成功还原由 `VFP&EXE` 所有版本加密的 `VFP` 程序。程序使用 `Microsoft Visual Basic 6.0 [SP5]` 中文版编写，在 `Windows 98/ME` 下测试通过。您可以免费地使用、传播它，但必须保持软件的完整。在使用本软件解密时请备份好原程序，以免受到不必要的数据损失。

但是偶还想自己手动试试呀，于是有了这点笔记。请求 老王 老师的谅解！:-( 如果您不太高兴就麻烦斑竹删了这个吧，先说声对不起。偶以后没多少时间 `CRACK` 了，人生一世真的不易，白驹过隙白云苍狗，该做的事情太多了。

### 手动脱壳分析

老王的 `VFP&EXE` 使用了 `UPX` 压缩，所以我们首先进入的是 `UPX` 加壳的代码部分。

用 `Ollydbg` 手动脱壳,老规矩：载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

```
00464F80 B>   60                     pushad
                              ===>进入 OD 后断在这！

00464F81     BE 00104400           mov esi,BJGS.00441000
00464F86     8DBE 0000FCFF        lea edi,dword ptr ds:[esi+FFFC0000]
00464F8C     C787 B0E40400 D8CF89>mov dword ptr ds:[edi+4E4B0],5289CFD8
00464F96     57                   push edi
00464F97     83CD FF             or ebp,FFFFFFFF
00464F9A     EB 0E               jmp short BJGS.00464FAA
```

脱 `UPX` 可谓是相当简单啦，不用跟踪，向下看，找到 `popad`、`jmp` 就行了。:-)

```
00464FAA     8B1E                 mov ebx,dword ptr ds:[esi]
00464FAC     83EE FC             sub esi,-4
00464FAF     11DB                adc ebx,ebx
00464FB1     8A07                mov al,byte ptr ds:[edi]
00464FB3   ^ 72 EB               jb short BJGS.00464FA0
00464FB5     B8 01000000         mov eax,1
00464FBA     01DB                add ebx,ebx
00464FBC     75 07               jnz short BJGS.00464FC5
00464FBE     8B1E                mov ebx,dword ptr ds:[esi]
00464FC0     83EE FC             sub esi,-4
00464FC3     11DB                adc ebx,ebx
00464FC5     11C0                adc eax,eax
00464FC7     01DB                add ebx,ebx
00464FC9   ^ 73 EF               jnb short BJGS.00464FBA
00464FCB     75 09               jnz short BJGS.00464FD6
00464FCD     8B1E                mov ebx,dword ptr ds:[esi]
00464FCF     83EE FC             sub esi,-4
00464FD2     11DB                adc ebx,ebx
00464FD4   ^ 73 E4               jnb short BJGS.00464FBA
00464FD6     31C9                xor ecx,ecx
00464FD8     83E8 03             sub eax,3
00464FDB     72 0D               jb short BJGS.00464FEA
00464FDD     C1E0 08             shl eax,8
00464FE0     8A06                mov al,byte ptr ds:[esi]
00464FE2     46                   inc esi
00464FE3     83F0 FF             xor eax,FFFFFFFF
00464FE6     74 76               je short BJGS.0046505E
00464FE8     89C5                mov ebp,eax
00464FEA     01DB                add ebx,ebx
00464FEC     75 07               jnz short BJGS.00464FF5
00464FEE     8B1E                mov ebx,dword ptr ds:[esi]
00464FF0     83EE FC             sub esi,-4
00464FF3     11DB                adc ebx,ebx
00464FF5     11C9                adc ecx,ecx
00464FF7     01DB                add ebx,ebx
00464FF9     75 07               jnz short BJGS.00465002
00464FFB     8B1E                mov ebx,dword ptr ds:[esi]
00464FFD     83EE FC             sub esi,-4
00465000     11DB                adc ebx,ebx
00465002     11C9                adc ecx,ecx
00465004     75 20               jnz short BJGS.00465026
00465006     41                   inc ecx
00465007     01DB                add ebx,ebx
00465009     75 07               jnz short BJGS.00465012
0046500B     8B1E                mov ebx,dword ptr ds:[esi]
0046500D     83EE FC             sub esi,-4
00465010     11DB                adc ebx,ebx
00465012     11C9                adc ecx,ecx
00465014     01DB                add ebx,ebx
00465016   ^ 73 EF               jnb short BJGS.00465007
00465018     75 09               jnz short BJGS.00465023
0046501A     8B1E                mov ebx,dword ptr ds:[esi]
0046501C     83EE FC             sub esi,-4
0046501F     11DB                adc ebx,ebx
00465021   ^ 73 E4               jnb short BJGS.00465007
00465023     83C1 02             add ecx,2
00465026     81FD 00F3FFFF       cmp ebp,-0D00
0046502C     83D1 01             adc ecx,1
0046502F     8D142F             lea edx,dword ptr ds:[edi+ebp]
00465032     83FD FC             cmp ebp,-4
00465035     8A040F             mov al,byte ptr ds:[edi+ecx]
00465038     76 0E               jbe short BJGS.00465048
0046503A     8A02                mov al,byte ptr ds:[edx]
0046503C     42                   inc edx
0046503D     8807                mov byte ptr ds:[edi],al
0046503F     47                   inc edi
00465040     49                   dec ecx
00465041   ^ 75 F7               jnz short BJGS.0046503A
00465043   ^ E9 5EFFFFFF         jmp BJGS.00464FA6
00465048     8B02                mov eax,dword ptr ds:[edx]
0046504A     83C2 04             add edx,4
0046504D     8907                mov dword ptr ds:[edi],eax
0046504F     83C7 04             add edi,4
00465052     83E9 04             sub ecx,4
00465055   ^ 77 F1               ja short BJGS.00465048
00465057     01CF                add edi,ecx
00465059   ^ E9 48FFFFFF         jmp BJGS.00464FA6
0046505E     5E                   pop esi
0046505F     89F7                mov edi,esi
00465061     B9 4F250000         mov ecx,254F
00465066     8A07                mov al,byte ptr ds:[edi]
00465068     47                   inc edi
00465069     2C E8               sub al,0E8
0046506B     3C 01               cmp al,1
0046506D   ^ 77 F7               ja short BJGS.00465066
0046506F     803F 0D             cmp byte ptr ds:[edi],0D
00465072   ^ 75 F2               jnz short BJGS.00465066
00465074     8B07                mov eax,dword ptr ds:[edi]
00465076     8A5F 04             mov bl,byte ptr ds:[edi+4]
00465079     66:C1E8 08          shr ax,8
0046507D     C1C0 10             rol eax,10
00465080     86C4                xchg ah,al
00465082     29F8                sub eax,edi
00465084     80EB E8             sub bl,0E8
00465087     01F0                add eax,esi
00465089     8907                mov dword ptr ds:[edi],eax
0046508B     83C7 05             add edi,5
0046508E     89D8                mov eax,ebx
00465090   ^ E2 D9               loopd short BJGS.0046506B
00465092     8DBE 00100600      lea edi,dword ptr ds:[esi+61000]
00465098     8B07                mov eax,dword ptr ds:[edi]
0046509A     09C0                or eax,eax
```



```
0046509C     74 3C                je short BJGS.004650DA

0046509E     8B5F 04             mov ebx,dword ptr ds:[edi+4]

004650A1     8D8430 A8610600     lea eax,dword ptr ds:[eax+esi+661A8]

004650A8     01F3                add ebx,esi

004650AA     50                 push eax

004650AB     83C7 08            add edi,8

004650AE     FF96 5C620600       call dword ptr ds:[esi+6625C]

004650B4     95                 xchg eax,ebp

004650B5     8A07               mov al,byte ptr ds:[edi]

004650B7     47                 inc edi

004650B8     08C0               or al,al

004650BA   ^ 74 DC              je short BJGS.00465098

004650BC     89F9               mov ecx,edi

004650BE     57                 push edi

004650BF     48                 dec eax

004650C0     F2:AE              repne scas byte ptr es:[edi]

004650C2     55                 push ebp

004650C3     FF96 60620600       call dword ptr ds:[esi+66260]

004650C9     09C0               or eax,eax

004650CB     74 07              je short BJGS.004650D4

004650CD     8903               mov dword ptr ds:[ebx],eax

004650CF     83C3 04            add ebx,4

004650D2   ^ EB E1              jmp short BJGS.004650B5

004650D4     FF96 64620600       call dword ptr ds:[esi+66264]

004650DA     61                 popad

                              ====>看到这里！直接 F4 到这！

004650DB   ^ E9 147EFEFF         jmp BJGS.0044CEF4

                              ====>跳向 UPX 加壳的 OEP！

————————————————————————

0044CEF4     55                 push ebp

                              ====>如果仅仅是 UPX 就可以在这儿 DUMP 了。继续跟踪！

0044CEF5     8BEC               mov ebp,esp

0044CEF7     83C4 F4            add esp,-0C

0044CEFA     B8 ACCD4400        mov eax,BJGS.0044CDAC

0044CEFF     E8 2488FBFF        call BJGS.00405728

0044CF04     A1 64E84400        mov eax,dword ptr ds:[44E864]

0044CF09     8B00               mov eax,dword ptr ds:[eax]

0044CF0B     E8 78D9FDFF        call BJGS.0042A888

0044CF10     8B0D FCE84400      mov ecx,dword ptr ds:[44E8FC] ; BJGS.0044F7D0

0044CF16     A1 64E84400        mov eax,dword ptr ds:[44E864]

0044CF1B     8B00               mov eax,dword ptr ds:[eax]

0044CF1D     8B15 78AF4400      mov edx,dword ptr ds:[44AF78] ; BJGS.0044AFB8

0044CF23     E8 78D9FDFF        call BJGS.0042A8A0

                              ====>F7 进入！

————————————————————————

0042A8A0     55                 push ebp

0042A8A1     8BEC               mov ebp,esp

0042A8A3     83C4 F8            add esp,-8

0042A8A6     53                 push ebx

0042A8A7     56                 push esi

0042A8A8     894D FC            mov dword ptr ss:[ebp-4],ecx

0042A8AB     8BF2               mov esi,edx

0042A8AD     8BD8               mov ebx,eax

0042A8AF     8BC6               mov eax,esi

0042A8B1     FF50 F4            call dword ptr ds:[eax-C]

0042A8B4     8945 F8            mov dword ptr ss:[ebp-8],eax

0042A8B7     8B45 FC            mov eax,dword ptr ss:[ebp-4]

0042A8BA     8B55 F8            mov edx,dword ptr ss:[ebp-8]

0042A8BD     8910               mov dword ptr ds:[eax],edx

0042A8BF     33C0               xor eax,eax

0042A8C1     55                 push ebp

0042A8C2     68 E3A84200        push BJGS.0042A8E3

0042A8C7     64:FF30            push dword ptr fs:[eax]

0042A8CA     64:8920            mov dword ptr fs:[eax],esp

0042A8CD     8BCB               mov ecx,ebx

0042A8CF     33D2               xor edx,edx

0042A8D1     8B45 F8            mov eax,dword ptr ss:[ebp-8]

0042A8D4     8B30               mov esi,dword ptr ds:[eax]

0042A8D6     FF56 24            call dword ptr ds:[esi+24]

                              ====>F7 进入！

————————————————————————

00425A54     55                 push ebp

   …… …… 省 略 …… ……

00425ADB     75 4A              jnz short BJGS.00425B27

                              ====>跳

00425B27     33C0               xor eax,eax

00425B29     5A                 pop edx

00425B2A     59                 pop ecx

00425B2B     59                 pop ecx

00425B2C     64:8910            mov dword ptr fs:[eax],edx

00425B2F     68 465B4200        push BJGS.00425B46

00425B34     8B45 FC            mov eax,dword ptr ss:[ebp-4]

00425B37     80A0 3D010000 FE   and byte ptr ds:[eax+13D],0FE

00425B3E     C3                 retn

                              ====>返回到 00425B46

00425B46     33C0               xor eax,eax

00425B48     55                 push ebp

00425B49     68 7D5B4200        push BJGS.00425B7D

00425B4E     64:FF30            push dword ptr fs:[eax]

00425B51     64:8920            mov dword ptr fs:[eax],esp

00425B54     8B45 FC            mov eax,dword ptr ss:[ebp-4]

00425B57     66:83B8 CE010000 00 cmp word ptr ds:[eax+1CE],0

00425B5F     74 12              je short BJGS.00425B73

00425B61     8B5D FC            mov ebx,dword ptr ss:[ebp-4]

00425B64     8B55 FC            mov edx,dword ptr ss:[ebp-4]

00425B67     8B83 D0010000      mov eax,dword ptr ds:[ebx+1D0]

00425B6D     FF93 CC010000      call dword ptr ds:[ebx+1CC]     ; BJGS.0044B104

                              ====>F7 进入！

————————————————————————

0044B104     55                 push ebp

   …… …… 省 略 …… ……

0044B178     75 67              jnz short BJGS.0044B1E1

                              ====>跳

0044B1E1     A1 ECF74400        mov eax,dword ptr ds:[44F7EC]

0044B1E6     A3 F0F74400        mov dword ptr ds:[44F7F0],eax

0044B1EB     C605 D4F74400 02   mov byte ptr ds:[44F7D4],2

0044B1F2     B8 FC094500        mov eax,BJGS.004509FC

0044B1F7     E8 9087FBFF        call BJGS.0040398C

0044B1FC     E8 EB76FBFF        call BJGS.004028EC

0044B201     8BF8               mov edi,eax

0044B203     85FF               test edi,edi

0044B205     7E 3B              jle short BJGS.0044B242

                              ====>跳

0044B242     8D95 C8F7FFFF      lea edx,dword ptr ss:[ebp-838]

   …… …… 省 略 …… ……

0044B28A   ^ 75 E2              jnz short BJGS.0044B26E

                              ====>F4 跳出循环！

0044B28C     8D95 C8F7FFFF      lea edx,dword ptr ss:[ebp-838]

   …… …… 省 略 …… ……

0044B2C1     7D 45              jge short BJGS.0044B308

                              ====>跳

0044B308     B9 02000000        mov ecx,2

   …… …… 省 略 …… ……

0044B385     E8 86F9FDFF        call BJGS.0042AD10

0044B38A     8B95 C8F7FFFF      mov edx,dword ptr ss:[ebp-838]

0044B390     B8 F8094500        mov eax,BJGS.004509F8

0044B395     E8 4686FBFF        call BJGS.004039E0

                              ====>取得系统目录路径： C:WINDOWS

0044B39A     68 00010000        push 100

0044B39F     68 F4F74400        push BJGS.0044F7F4           ; ASCII "C:WINDOWS"

0044B3A4     E8 63A5FBFF        call BJGS.0040590C

0044B3A9     8D45 D8            lea eax,dword ptr ss:[ebp-28]

0044B3AC     BA F4F74400        mov edx,BJGS.0044F7F4        ; ASCII "C:WINDOWS"

0044B3B1     B9 00020000        mov ecx,200

0044B3B6     E8 FD87FBFF        call BJGS.00403BB8

0044B3BB     8D45 D4            lea eax,dword ptr ss:[ebp-2C]

0044B3BE     50                 push eax

0044B3BF     B9 02000000        mov ecx,2

0044B3C4     BA 01000000        mov edx,1

0044B3C9     8B45 D8            mov eax,dword ptr ss:[ebp-28]
```



```
0044B3CC     E8 3B8AFBFF          call BJGS.00403E0C   

0044B3D1     8B45 FC             mov eax,dword ptr ss:[ebp-4]   

0044B3D4     E8 57140000         call BJGS.0044C830   

0044B3D9     85C0                test eax,eax   

0044B3DB   ^ 0F85 FAFEFFFF       jnz BJGS.0044B2DB   

0044B3E1     8D85 C8F7FFFF       lea eax,dword ptr ss:[ebp-838]   

0044B3E7     B9 54C04400         mov ecx,BJGS.0044C054         ; ASCII "Recycled"   

0044B3EC     8B55 D4             mov edx,dword ptr ss:[ebp-2C]   

0044B3EF     E8 6088FBFF         call BJGS.00403C54   

                             ==== > 取得 回收站 路径：C:Recycled   

0044B3F4     8B85 C8F7FFFF       mov eax,dword ptr ss:[ebp-838]   

0044B3FA     E8 A5C4FBFF         call BJGS.004078A4   

0044B3FF     84C0                test al,al   

0044B401     74 24               je short BJGS.0044B427   

0044B403     B8 000A4500         mov eax,BJGS.00450A00   

0044B408     B9 54C04400         mov ecx,BJGS.0044C054          ; ASCII "Recycled"   

0044B40D     8B55 D4             mov edx,dword ptr ss:[ebp-2C]   

0044B410     E8 3F88FBFF         call BJGS.00403C54   

0044B415     8D45 D0             lea eax,dword ptr ss:[ebp-30]   

0044B418     BA 68C04400         mov edx,BJGS.0044C068 ; ASCII "{645FF040-5081-101B-9F08-00AA002F954E}"   

                             ==== > 注册表：PropertySheetHandlers   

0044B41D     E8 0286FBFF         call BJGS.00403A24   

0044B422     E9 84000000         jmp BJGS.0044B4AB   

0044B4AB     B8 080A4500         mov eax,BJGS.00450A08   

  …………  省 略  …………   

0044B4F6     0F85 F4000000       jnz BJGS.0044B5F0   

                             ==== > 跳   

0044B5F0     FF35 000A4500       push dword ptr ds:[450A00]   

  …………  省 略  …………   

0044B648     E8 FFBABFF          call BJGS.0040714C   

                             ==== > 在 C:WINDOWSTemp 下生成：fs270.tmp   

0044B64D     8945 F8             mov dword ptr ss:[ebp-8],eax   

0044B650     837D F8 00          cmp dword ptr ss:[ebp-8],0   

0044B654     7D 4C               jge short BJGS.0044B6A2   

                             ==== > 跳   

0044B6A2     33C9                xor ecx,ecx   

  …………  省 略  …………   

0044B70C     0F85 FA000000       jnz BJGS.0044B80C   

                             ==== > 跳   

0044B80C     8B45 FC             mov eax,dword ptr ss:[ebp-4]   

0044B80F     8B80 DC010000       mov eax,dword ptr ds:[eax+1DC]   

0044B815     83C0 38             add eax,38   

0044B818     8B15 9CE74400       mov edx,dword ptr ds:[44E79C]   ; BJGS.0044F7C4   

0044B81E     8B12                mov edx,dword ptr ds:[edx]   

0044B820     E8 BB81FBFF         call BJGS.004039E0   

0044B825     8A85 CCFBFFFF       mov al,byte ptr ss:[ebp-434]   

0044B82B     3C 40               cmp al,40   

0044B82D     74 58               je short BJGS.0044B887   

0044B82F     3C 60               cmp al,60   

0044B831     74 54               je short BJGS.0044B887   

                             ==== > 跳   

0044B887     8B55 F4             mov edx,dword ptr ss:[ebp-C]   

  …………  省 略  …………   

0044B8DF     E8 A880FBFF         call BJGS.0040398C   

0044B8E4     BF E8030000         mov edi,3E8   

0044B8E9     8D9D B4FBFFFF       lea ebx,dword ptr ss:[ebp-44C]   

                             ==== > EBX=y062007.exe     很有意思的一个文件 :-)   

0044B8EF     8D85 C8F7FFFF       lea eax,dword ptr ss:[ebp-838]   

0044B8F5     8A13                mov dl,byte ptr ds:[ebx]   

0044B8F7     E8 3482FBFF         call BJGS.00403B30   

0044B8FC     8B95 C8F7FFFF       mov edx,dword ptr ss:[ebp-838]   

0044B902     B8 F8094500         mov eax,BJGS.004509F8   

0044B907     E8 0483FBFF         call BJGS.00403C10   

0044B90C     47                  inc edi   

0044B90D     43                  inc ebx   

0044B90E     81FF F3030000       cmp edi,3F3   

0044B914   ^ 75 D9               jnz short BJGS.0044B8EF   

                             ==== > F4 跳出循环！   

0044B916     C705 D8F74400 F1D8FF>mov dword ptr ds:[44F7D8],-270F   

  …………  省 略  …………   

0044B998     0F8D 4E010000       jge BJGS.0044BAEC   

                             ==== > 跳   

0044BAEC     833D D8F74400 00     cmp dword ptr ds:[44F7D8],0   

  …………  省 略  …………   

0044BB67     0F85 E3010000       jnz BJGS.0044BD50   

                             ==== > 跳   

0044BD50     8B45 F8             mov eax,dword ptr ss:[ebp-8]   

  …………  省 略  …………   

0044BDF3     EB 35               jmp short BJGS.0044BE2A   

0044BE2A     C605 E8F74400 00     mov byte ptr ds:[44F7E8],0   

0044BE31     8B45 FC             mov eax,dword ptr ss:[ebp-4]   

0044BE34     8B80 E4010000       mov eax,dword ptr ds:[eax+1E4]   

0044BE3A     E8 A1E3FFFF         call BJGS.0044A1E0   

                             ==== > F7 进入！   
```

```
0044A1E0     53                  push ebx   

0044A1E1     56                  push esi   

0044A1E2     57                  push edi   

0044A1E3     8BF0                mov esi,eax   

0044A1E5     33DB                xor ebx,ebx   

0044A1E7     33C0                xor eax,eax   

0044A1E9     8986 A8000000       mov dword ptr ds:[esi+A8],eax   

0044A1EF     807E 45 00          cmp byte ptr ds:[esi+45],0   

0044A1F3     75 6D               jnz short BJGS.0044A262   

0044A1F5     B2 01               mov dl,1   

0044A1F7     8BC6                mov eax,esi   

0044A1F9     E8 A2010000         call BJGS.0044A3A0   

                             ==== > F7 进入！   
```

```
0044A3A0     55                  push ebp   

  …………  省 略  …………   

0044A490     E8 37B3FBFF         call BJGS.004057CC   

                             ==== > F7 进入！   

004057CC   - FF25 4C134500       jmp dword ptr ds:[45134C]   

                             ==== > 程序运行！   
```

看看堆栈： 会有发现的！

```
0069F20C   0044A495    /CALL 到 CreateProcessA 来自 BJGS.0044A490   

0069F210   00000000    |ModuleFileName = NULL   

0069F214   0069F252    |CommandLine = "c:   

ecycled.............y062007.exe "   

0069F218   00000000    |pProcessSecurity = NULL   

0069F21C   00000000    |pThreadSecurity = NULL   

0069F220   00000000    |InheritHandles = FALSE   

0069F224   00000030    |CreationFlags = CREATE_NEW_CONSOLE|NORMAL_PRIORITY_CLASS   

0069F228   00000000    |pEnvironment = NULL   

0069F22C   00000000    |CurrentDir = NULL   

0069F230   00BC48D0    |pStartupInfo = 00BC48D0   

0069F234   00BC4914    pProcessInfo = 00BC4914   
```

看到：`c:\ecycled\y062007.exe`  呵呵，这个就是原程序啦！:-)

## WIN+R 打开“运行”窗口，填入：`winfile`  呵呵，古老的“文件管理器”又要发挥余热啦！

定位到 C 盘，文件—>搜索：`y062007.exe`  不负所望,找到：`C:RECYCLED\Y062007.EXE`

选中搜索到的文件,F8 把 `Y062007.EXE` 复制到：`E:DUMP.EXE`

OK,看到隐藏属性的 `DUMP.EXE` 露出了本来的“狐狸头”的面目。正常运行！:-) 2.02M

## 【VFP&EXE 脱壳小结】：

偶 篇名的 巧脱 是“偷巧”的意思呀，看看如何偷巧吧：

运行用 `VFP&EXE` 内存型加密的主程序，再运行某个进程查看工具，我直接用 `LordPE` 啦。会发现某个特殊的进程，如这个《造价大师：北京预算 2002 版及北京房修 2003 估价表》是“`C:RECYCLED\Y062007.EXE`”。《新资料大师》是 “`C:RECYCLED\Y317710.EXE`”。呵呵，看见目标啦。

OK，这时用 `LordPE` 结束原先运行的 `E:试炼场北京预算 2002 清单版 Bjgs.exe` 这个进程！

运行 Win98 下的 文件管理器，在 C 盘搜索你看到的那个 EXE 程序，找到后复制出来就是脱壳后的程序啦。

把得到的程序和 `Aming` 大侠的文件型加密 VFP 程序终结者 1.50 精简版脱壳后的文件比较，发觉是一样的。

完全是偷巧的手法，没有技术含量。最后再请 `老王` 老师海涵！

```
         ,     \_/   
       /| \_.-~/         \_   ,        青春都一饷
      
      ( /~   /            ~-\_ |
       \`\\  \_/                ~ )           忍把浮名  
  \_-~~~-.)  )\_\/;;,.         \_  //'    
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂 
  \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`   
   /~~//'   /' \`~            ) /--..\_, )\_  \`~    
   "  \`~"  "       \`"        /~'\`    \`\\~~   
    
                   "     "   "~'  ""   
```



Cracked By 巢水工作坊——fly [OCN][FCG]

2003-10-10 18:00

`4dd466d0e1e0cc123afbe504da2462ca`
