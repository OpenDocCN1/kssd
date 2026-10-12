
### 汇编代码分析

:00408AE3 `8B4D08` mov `ecx`, `dword ptr [ebp+08]`

:00408AE6 `51` push `ecx`

\* Reference To: `USER32.MessageBoxA`, Ord:0000h

### 关键 CALL 分析

进入关键 `CALL`：`00408A66` call `004087CB`

\* Referenced by a CALL at Addresses:
`:004010CB`, `:00408A66`, `:0040A71E`

:004087CB `55` push `ebp`
:004087CC `8BEC` mov `ebp`, `esp`
:004087CE `83EC14` sub `esp`, `00000014`
:004087D1 `68648C4100` push `00418C64`
:004087D6 `6868694100` push `00416968`
:004087DB `E8B0350000` call `0040BD90`
:004087E0 `83C408` add `esp`, `00000008`
:004087E3 `85C0` test `eax`, `eax`
:004087E5 `7507` jne `004087EE`
:004087E7 `33C0` xor `eax`, `eax`
:004087E9 `E9BF000000` jmp `004088AD`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004087E5(C)`

:004087EE `6868694100` push `00416968`
====> `00416968` = `13572468`

:004087F3 `8D45F0` lea `eax`, `dword ptr [ebp-10]`
:004087F6 `50` push `eax`
:004087F7 `E8A4340000` call `0040BCA0`
:004087FC `83C408` add `esp`, `00000008`
:004087FF `8A4DF6` mov `cl`, `byte ptr [ebp-0A]`
====> `CL` = `36` 取 `13572468` 第 7 位字符的 HEX 值

:00408802 `884DEC` mov `byte ptr [ebp-14]`, `cl`
====> `[ebp-14]` = `CL`

:00408805 `C645F600` mov `[ebp-0A]`, `00`
:00408809 `8D55F0` lea `edx`, `dword ptr [ebp-10]`
====> `EDX` = `135724` 取 `13572468` 前 6 位

:0040880C `52` push `edx`
:0040880D `E8B8380000` call `0040C0CA`
====> 取 `135724` 的 16 进制值 `0002122C`

:00408812 `83C404` add `esp`, `00000004`
:00408815 `8945FC` mov `dword ptr [ebp-04]`, `eax`
====> `[ebp-04]` = `EAX` = `0002122C`

:00408818 `0FBE45EC` movsx `eax`, `byte ptr [ebp-14]`
====> `EAX` = `[ebp-14]` = `36` 即：`13572468` 第 7 位字符的 HEX 值

:0040881C `8B0C8530604100` mov `ecx`, `dword ptr [4*eax+00416030]`
====> 以第 7 位字符的 HEX 值为参数取值
====> `ECX` = `[4*36+00416030]` = `[416108]` = `00040A3F`

#### 数据表

`[4*eax+00416030]` 内存中有张不小的表：

```
00416030  C7 AA 01 00 BF 42 08 00 DF 3D 02 00 FA F5 0D 00  仟.緽.?.   .BF 4  
00416040  93 E8 0C 00 C4 FA 06 00 61 B5 03 00 1C E4 08 00  撹..您.a?.?.C4 F  
00416050  11 2D 09 00 AC 1A 03 00 0E AC 0D 00 33 FE 06 00  -..?.?.3?.C 1  
00416060  5E AC 0E 00 3D 11 02 00 4C 9D 08 00 5D EF 09 00  ^?.=.L?.]?.D 1  
00416070  CE 96 03 00 06 59 0C 00 19 95 02 00 69 1B 04 00  螙.Y..?.i. 5  
00416080  36 D8 07 00 7B 9A 0B 00 BB C2 09 00 14 4E 0C 00  6?.{?.宦..N..B 9  
00416090  DA 56 04 00 8C 80 06 00 6F C5 07 00 21 A0 04 00  赩.實.o?.!?.8C 8  
004160A0  0D D5 01 00 32 34 04 00 68 8C 02 00 95 99 04 00  .?.24.h?.暀.2 3  
004160B0  7F E4 0C 00 C6 4E 07 00 3E 1D 0E 00 8B 9D 05 00  ??.芅.>.嫕.6 4  
004160C0  B1 32 0F 00 E4 B3 02 00 D4 B5 01 00 EB 2E 06 00  ??.涑.缘.?.E4 B  
004160D0  FA 56 0D 00 DA 37 0E 00 EC 86 0E 00 D0 90 0C 00  鶹..?.靻.袗..DA 3  
004160E0  C5 49 03 00 2A 49 0A 00 2E B6 09 00 CC 32 0F 00  臝.*I...?.?.A 4  
004160F0  38 9A 05 00 5B 94 0C 00 31 68 08 00 B7 24 07 00   8?.[?.1h.?.B 9  
00416100  F8 BE 0A 00 31 92 03 00 3F 0A 04 00 A4 CE 09 00    ..1?.?..の..1 9  
00416110  7E 96 02 00 74 CB 06 00 69 3A 05 00 2B 5F 09 00  ~?.t?.i:.+_. C  
00416120  1C 50 0D 00 B2 5D 06 00 E1 08 0C 00 30 A2 02 00  P..瞉.?..0?.2 5  
```
…… 省略 ……

### 验证逻辑

:00408823 `3B4DFC` cmp `ecx`, `dword ptr [ebp-04]`
====> `ECX` = `00040A3F` (H) = `264767` (D)
====> `[ebp-04]` = `0002122C` (H) = `135724` (D)
====> 因此前 6 位注册码应是 `264767`

:00408826 `7507` jne `0040882F`

:00408828 `B801000000` mov `eax`, `00000001`
====> 置 1 则 OK！

:0040882D `EB7E` jmp `004088AD`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00408826(C)`

:0040882F `6868694100` push `00416968`
:00408834 `8D55F0` lea `edx`, `dword ptr [ebp-10]`
:00408837 `52` push `edx`
:00408838 `E863340000` call `0040BCA0`
:0040883D `83C408` add `esp`, `00000008`
:00408840 `8D45F0` lea `eax`, `dword ptr [ebp-10]`
:00408843 `50` push `eax`
:00408844 `E82A390000` call `0040C173`
:00408849 `83C404` add `esp`, `00000004`
:0040884C `8A4DF6` mov `cl`, `byte ptr [ebp-0A]`
:0040884F `884DEC` mov `byte ptr [ebp-14]`, `cl`
:00408852 `C645F600` mov `[ebp-0A]`, `00`
:00408856 `0FBE55EC` movsx `edx`, `byte ptr [ebp-14]`
:0040885A `8B049530604100` mov `eax`, `dword ptr [4*edx+00416030]`
:00408861 `3B45FC` cmp `eax`, `dword ptr [ebp-04]`
====> 还有机会

:00408864 `7507` jne `0040886D`

:00408866 `B801000000` mov `eax`, `00000001`
====> 置 1 则 OK！

:0040886B `EB40` jmp `004088AD`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00408864(C)`

:0040886D `6868694100` push `00416968`
:00408872 `8D4DF0` lea `ecx`, `dword ptr [ebp-10]`
:00408875 `51` push `ecx`
:00408876 `E825340000` call `0040BCA0`
:0040887B `83C408` add `esp`, `00000008`
:0040887E `8D55F0` lea `edx`, `dword ptr [ebp-10]`
:00408881 `52` push `edx`
:00408882 `E84E380000` call `0040C0D5`
:00408887 `83C404` add `esp`, `00000004`
:0040888A `8A45F6` mov `al`, `byte ptr [ebp-0A]`
:0040888D `8845EC` mov `byte ptr [ebp-14]`, `al`
:00408890 `C645F600` mov `[ebp-0A]`, `00`
:00408894 `0FBE4DEC` movsx `ecx`, `byte ptr [ebp-14]`
:00408898 `8B148D30604100` mov `edx`, `dword ptr [4*ecx+00416030]`
:0040889F `3B55FC` cmp `edx`, `dword ptr [ebp-04]`
====> 还有机会

:004088A2 `7507` jne `004088AB`
====> 跳则 OVER！

:004088A4 `B801000000` mov `eax`, `00000001`
====> 置 1 则 OK！

:004088A9 `EB02` jmp `004088AD`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004088A2(C)`

:004088AB `33C0` xor `eax`, `eax`
====> 清 0 则 OVER！爆破点！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:004087E9(U)`, `:0040882D(U)`, `:0040886B(U)`, `:004088A9(U)`
