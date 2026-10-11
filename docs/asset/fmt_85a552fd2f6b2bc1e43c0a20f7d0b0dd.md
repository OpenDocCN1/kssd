
:00404310 52 push edx   
:00404311 683F000F00 push 000F003F   
:00404316 6A00 push 00000000   
:00404318 8D45A0 lea eax, dword ptr [ebp-60]   
:0040431B 50 push eax   
:0040431C 6802000080 push 80000002   

*:

*:

*:

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\|:00404485(C)

\|

:00404491 833D7020420000 cmp dword ptr [00422070], 00000000   
:00404498 740E je 004044A8   
:0040449A C605C9FD410001 mov byte ptr [0041FDC9], 01（第二条）   
:004044A1 C605CAFD410001 mov byte ptr [0041FDCA], 01（第三条）   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\|:00404498(C)

\|

:004044A8 6A03 push 00000003   
:004044AA E861F3FFFF call 00403810   
:004044AF 83C404 add esp, 00000004   
:004044B2 85C0 test eax, eax   
:004044B4 7434 je 004044EA （不 nop 掉只有三条鱼）   
改：90 nop   
90 nop   
:004044B6 C7057020420003000000 mov dword ptr [00422070], 00000003   
:004044C0 C605C9FD410001 mov byte ptr [0041FDC9], 01 （第二条）   
:004044C7 C605CAFD410001 mov byte ptr [0041FDCA], 01 （第三条）   
:004044CE C605CBFD410001 mov byte ptr [0041FDCB], 01 （第四条）   
:004044D5 C605CCFD410001 mov byte ptr [0041FDCC], 01 （第五条）   
:004044DC C605CDFD410001 mov byte ptr [0041FDCD], 01 （第六条）   
:004044E3 C605CEFD410001 mov byte ptr [0041FDCE], 01 （第七条）至于第一条吗？你想想吧！！！   

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

\|:00404477(C), :004044B4(C)

\|

:004044EA 833D7020420003 cmp dword ptr [00422070], 00000003   
:004044F1 0F8511010000 jne 00404608   
:004044F7 8D459C lea eax, dword ptr [ebp-64]   
:004044FA 50 push eax   
:004044FB 6880324200 push 00423280   
:00404500 8D4D88 lea ecx, dword ptr [ebp-78]   
:00404503 51 push ecx   
:00404504 6A00 push 00000000

Sachs Marine Aquarium 0.99G Beta 破解要点

1、输入任意注册码，但要求格式为 6 个大字字母+6 位数字，我输入的是：`AAAAAA123456`

打 `bpx hmemcpy` 下断点，中断后，小心地跟踪，一直来到：

\* Possible StringData Ref from Data Obj ->"TESTFISH"

\|

:0040DABF 6834064200 push 00420634   
:0040DAC4 684C1E4200 push 00421E4C   

\* Reference To: KERNEL32.lstrcmpiA, Ord:02FFh

\|

:0040DAC9 FF1598B04100 Call dword ptr [0041B098] ; 比较临时注册码'TESTFISH'   
:0040DACF 85C0 test eax, eax   
:0040DAD1 750A jne 0040DADD   
:0040DAD3 C7057020420001000000 mov dword ptr [00422070], 00000001   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\|:0040DAD1(C)

\|

:0040DADD 6A02 push 00000002   
:0040DADF E82C5DFFFF call 00403810 ; 进入后，可找到 BASIC 注册码   
:0040DAE4 83C404 add esp, 00000004   
:0040DAE7 85C0 test eax, eax   
:0040DAE9 7414 je 0040DAFF   
:0040DAEB C7057020420002000000 mov dword ptr [00422070], 00000002   
:0040DAF5 6A01 push 00000001   
:0040DAF7 E8B964FFFF call 00403FB5   
:0040DAFC 83C404 add esp, 00000004   

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\|:0040DAE9(C)

\|

:0040DAFF 6A03 push 00000003   
:0040DB01 E80A5DFFFF call 00403810 ; 进入后，可找到 DELUXE 注册码   
:0040DB06 83C404 add esp, 00000004   
:0040DB09 8945F8 mov dword ptr [ebp-08], eax   
:0040DB0C 837DF800 cmp dword ptr [ebp-08], 00000000   
:0040DB10 7414 je 0040DB26   
:0040DB12 C7057020420003000000 mov dword ptr [00422070], 00000003   
:0040DB1C 6A01 push 00000001   
:0040DB1E E89264FFFF call 00403FB5   
:0040DB23 83C404 add esp, 00000004

2、进入 `call 00403810` ，来到下边时，可找到真正注册码：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

\|:0040398E(C)

\|

:004039A1 8D45E8 lea eax, dword ptr [ebp-18]   
:004039A4 50 push eax ; d eax，可看到真注册码   
:004039A5 684C1E4200 push 00421E4C ; d 00421E4C，可看到假注册码   

\* Reference To: KERNEL32.lstrcmpiA, Ord:02FFh

\|

:004039AA FF1598B04100 Call dword ptr [0041B098] ; 注册码比较   
:004039B0 85C0 test eax, eax   
:004039B2 7507 jne 004039BB   
:004039B4 B801000000 mov eax, 00000001   
:004039B9 EB02 jmp 004039BD

3、如果只需要注册码的话，可以直接下断点 `bpx 004039A4`，然后打 `d eax`，即可得到正确的注册码。第 1 次中断得到 `BASIC REGISTRATION CODE`，第 2 次中断得到 `DELUXE REGISTRATION CODE`。

`4dd466d0e1e0cc123afbe504da2462ca`
