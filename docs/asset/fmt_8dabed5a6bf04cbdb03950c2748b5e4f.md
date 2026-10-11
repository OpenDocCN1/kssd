
# 中华通讯录 V5.8.102 Build 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-427.htm`](https://www.pediy.com/kssd/pediy07/pediy7-427.htm)

【破文标题】：中华通讯录 V5.8.102 Build 注册算法分析  
【破文作者】：KuNgBiM\[DFCG\]  
【作者邮箱】：gb\_1227@163.com  
【软件名称】：中华通讯录 V5.8.102 Build  
【软件大小】：3346 KB  
【软件类别】：国产软件 / 共享版 / 信息管理  
【更新日期】：2005-09-14  
【下载地址】：[`www1.skycn.com/soft/12563.html`](http://www1.skycn.com/soft/12563.html)  
【保护方式】：注册码 + 功能限制  
【编译语言】：Borland Delphi 4.0 - 5.0  
【加壳方式】：UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo  
【调试环境】：`WinXP`、`PEiD`、`Ollydbg`  
【破解日期】：2005-09-24  
【破解目的】：推广使用 ESP 定律脱壳，以及研究算法分析  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 破解过程

### 分离壳过程

侦壳脱壳：用 `PEiD` 查壳，`UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo` 加壳

`Ollydbg` 载入主程序：

```assembly
006A8240 >   60                    pushad                                      ; UPX 标准壳入口，F8 一次
006A8241     BE 00F05B00           mov esi,中华通讯.005BF000                   ; 来到这里，这时查看寄存器窗口
006A8246     8DBE 0020E4FF         lea edi,dword ptr ds:[esi+FFE42000]
006A824C     57                    push edi
006A824D     83CD FF              or ebp,FFFFFFFF
006A8250     EB 10                jmp short 中华通讯.006A8262
```

#### 寄存器

```text
EAX 00000000
ECX 0012FFB0
EDX 7C92EB94 ntdll.KiFastSystemCallRet
EBX 7FFDF000
ESP 0012FFA4   //esp=0012ffa4
EBP 0012FFF0
ESI FFFFFFFF
EDI 7C930738 ntdll.7C930738
EIP 006A8241 中华通讯.006A8241
```

根据 ESP 定律规则，现在在命令栏中下 `hr 0012ffa4` 命令，回车，`F9` 运行：

```assembly
006A83A3   - E9 AC49ECFF          jmp 中华通讯.0056CD54                         ; 这里断下,再 F8 一次，飞向光明之颠！^\_^
006A83A8     C083 6A00D083 6A     rol byte ptr ds:[ebx+83D0006A],6A
006A83AF     00D4                 add ah,dl
006A83B1     64:57                push edi
006A83B3     0000                 add byte ptr ds:[eax],al
006A83B5     0000                 add byte ptr ds:[eax],al
006A83B7     0000                 add byte ptr ds:[eax],al
```

```assembly
0056CD54     55                    push ebp                                    ; Borland Delphi 4.0 - 5.0 程序标准入口
0056CD55     8BEC                 mov ebp,esp
0056CD57     83C4 F4              add esp,-0C
0056CD5A     53                    push ebx
0056CD5B     B8 DCC75600          mov eax,中华通讯.0056C7DC
0056CD60     E8 EFAEE9FF          call 中华通讯.00407C54
0056CD65     8B1D E4515700        mov ebx,dword ptr ds:[5751E4]                ; 中华通讯.005767D8
0056CD6B     8B03                 mov eax,dword ptr ds:[ebx]
0056CD6D     E8 DA9EEEFF          call 中华通讯.00456C4C
0056CD72     8B0B                 mov ecx,dword ptr ds:[ebx]
0056CD74     B2 01                mov dl,1
0056CD76     A1 FCBE5600          mov eax,dword ptr ds:[56BEFC]
0056CD7B     E8 B02FEEFF          call 中华通讯.0044FD30
0056CD80     8B15 00535700        mov edx,dword ptr ds:[575300]                ; 中华通讯.005798C4
0056CD86     8902                 mov dword ptr ds:[edx],eax
0056CD88     A1 00535700          mov eax,dword ptr ds:[575300]
0056CD8D     8B00                 mov eax,dword ptr ds:[eax]
0056CD8F     E8 7C6EEEFF          call 中华通讯.00453C10
0056CD94     A1 00535700          mov eax,dword ptr ds:[575300]
0056CD99     8B00                 mov eax,dword ptr ds:[eax]
0056CD9B     8B10                 mov edx,dword ptr ds:[eax]
0056CD9D     FF92 80000000        call dword ptr ds:[edx+80]
0056CDA3     8B03                 mov eax,dword ptr ds:[ebx]
0056CDA5     BA E8CE5600          mov edx,中华通讯.0056CEE8                    ; ASCII "cnet"
```

最后命令 `hd 0012ffa4` 取消断点!

---

### 算法分析过程

注册运行有对话框提示 "注册码不正确,无法注册"

用 `OD` 字符串插件,找到 "注册码不正确,无法注册",双击

`F9` 运行，输入试炼信息：

```text
*******************************

用户机器码：6EF4-33E9

用户注册玛：9876543210

*******************************
```

```assembly
005671AD     FC                    cld
005671AE     B8 C4CAA7B0          mov eax,B0A7CAC4
005671B3     DCA3 A1000053        fsub qword ptr ds:[ebx+530000A1]
005671B9     8BD8                 mov ebx,eax
005671BB     8BC3                 mov eax,ebx
005671BD     E8 9ACBFFFF          call 中华通讯.00563D5C                       ; 这里设断,★算法 CALL,F7 跟进!★
005671C2     84C0                 test al,al                                   ; 比较 al 中的值是否为 1
005671C4     74 09                je short 中华通讯.005671CF                   ; al 不等于 1 则跳死!(★爆破方法①关键点★)
005671C6     8BC3                 mov eax,ebx
005671C8     E8 DFC8FFFF          call 中华通讯.00563AAC
005671CD     5B                    pop ebx
005671CE     C3                    retn
005671CF     6A 20                 push 20
005671D1     B9 EC715600          mov ecx,中华通讯.005671EC
005671D6     BA F8715600          mov edx,中华通讯.005671F8                    ; 双击来到这里,"注册码不正确,无法注册"
005671DB     A1 E4515700          mov eax,dword ptr ds:[5751E4]
005671E0     8B00                 mov eax,dword ptr ds:[eax]
005671E2     E8 55FCEEFF          call 中华通讯.00456E3C
005671E7     5B                    pop ebx
005671E8     C3                    retn
```

### 跟进 005671BD call 中华通讯.00563D5C

```assembly
00563D5C     55                    push ebp                                    ; 运算程序初始化
00563D5D     8BEC                 mov ebp,esp
00563D5F     33C9                 xor ecx,ecx
00563D61     51                    push ecx                                    ; 按时大苏打
00563D62     51                    push ecx
00563D63     51                    push ecx
00563D64     51                    push ecx
00563D65     51                    push ecx
00563D66     53                    push ebx                                    ; ebx=01133F54
00563D67     56                    push esi                                    ; esi=0114559C
00563D68     8945 FC              mov dword ptr ss:[ebp-4],eax                ; eax=01133F54
00563D6B     33C0                 xor eax,eax
00563D6D     55                    push ebp
00563D6E     68 383E5600          push 中华通讯.00563E38
00563D73     64:FF30              push dword ptr fs:[eax]
00563D76     64:8920              mov dword ptr fs:[eax],esp
00563D79     33C0                 xor eax,eax
00563D7B     8945 F4              mov dword ptr ss:[ebp-C],eax
00563D7E     8D55 F8              lea edx,dword ptr ss:[ebp-8]                ; edx=0114559C
00563D81     8B45 FC              mov eax,dword ptr ss:[ebp-4]                ; 堆栈 ss:[0012FBC0]=01133F54
00563D84     8B80 200A0000        mov eax,dword ptr ds:[eax+A20]              ; ds:[01134974]=011452CC, (ASCII "D3Q")
00563D8A     E8 913FEDFF          call 中华通讯.00437D20
00563D8F     8B45 F8              mov eax,dword ptr ss:[ebp-8]                ; 取机器码,堆栈 ss:[0012FBBC]=011D3A38
00563D92     E8 9904EAFF          call 中华通讯.00404230
00563D97     8BD8                 mov ebx,eax                                 ; eax=00000009
00563D99     85DB                 test ebx,ebx                                 ; 比较机器码是否为 9 位
00563D9B     7E 2E                jle short 中华通讯.00563DCB                  ; 不等则跳直接死!
00563D9D     BE 01000000          mov esi,1
00563DA2     8D45 F0              lea eax,dword ptr ss:[ebp-10]               ; eax=00000009
                                                                ; eax=00000036
                                                                ; eax=00000045
                                                                ; eax=00000046
                                                                ; eax=00000034
                                                                ; eax=0000002D
```
