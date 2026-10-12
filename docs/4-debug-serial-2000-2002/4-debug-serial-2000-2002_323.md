
# FISH 精美屏保

> 原文：[`www.pediy.com/kssd/pediy03/forum602.htm`](https://www.pediy.com/kssd/pediy03/forum602.htm)

FISH 精美屏保暴力破解---`WD32ASM893` 版

作者：孙锋

Email：`sffs@263.net`

主页：`http://sffs.6to23.com`

先用 `WD32ASM8.93` 超级中文版进行反汇编，然后查找，会看到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D097(C)
|
:0040D0AD 6A03                     push 00000003
:0040D0AF E82C67FFFF               call 004037E0
:0040D0B4 83C404                   add esp, 00000004
:0040D0B7 8945F8                   mov dword ptr [ebp-08], eax
:0040D0BA 837DF800                 cmp dword ptr [ebp-08], 00000000
:0040D0BE 7414                     je 0040D0D4               //----->修改 74-75 即 jne->je
:0040D0C0 C70500FE410003000000     mov dword ptr [0041FE00], 00000003
:0040D0CA 6A01                     push 00000001
:0040D0CC E8A86EFFFF               call 00403F79
:0040D0D1 83C404                   add esp, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D0BE(C)  //---------->修改处，往上找。
|
:0040D0D4 833D00FE410000           cmp dword ptr [0041FE00], 00000000
:0040D0DB 7458                     je 0040D135              //---------------修改 74->75 即 jne->je
:0040D0DD C605B1DD410001           mov byte ptr [0041DDB1], 01
:0040D0E4 C605B2DD410001           mov byte ptr [0041DDB2], 01
:0040D0EB 833D00FE410001           cmp dword ptr [0041FE00], 00000001
:0040D0F2 7514                     jne 0040D108              //-------------------修改 74->75 即 jne->je

* Possible StringData Ref from Data Obj ->"TEMP registration OK"  //-----2 条鱼！

                                 |
:0040D0F4 6834E54100             push 0041E534
:0040D0F9 6892000000             push 00000092
:0040D0FE 8B4D08                 mov ecx, dword ptr [ebp+08]
:0040D101 51                     push ecx

* Reference To: USER32.SetDlgItemTextA, Ord:022Ch

                                 |
:0040D102 FF1514A24100           Call dword ptr [0041A214]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D0F2(C)        //-------------修改的地方，往上找
|
:0040D108 833D00FE410002           cmp dword ptr [0041FE00], 00000002
:0040D10F 7514                     jne 0040D125

* Possible StringData Ref from Data Obj ->"BASIC registration OK" //-------3 条鱼！

                                 |
:0040D111 684CE54100             push 0041E54C
:0040D116 6892000000             push 00000092
:0040D11B 8B5508                 mov edx, dword ptr [ebp+08]
:0040D11E 52                     push edx

* Reference To: USER32.SetDlgItemTextA, Ord:022Ch

                                 |
:0040D11F FF1514A24100           Call dword ptr [0041A214]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D10F(C)
|
:0040D125 833D00FE410003           cmp dword ptr [0041FE00], 00000003  //----->
:0040D12C 7D07                     jge 0040D135  //------> 比较大于则跳，所以改为小于 7D->7E
:0040D12E 33C0                     xor eax, eax
:0040D130 E91B010000             jmp 0040D250

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0040D0DB(C), :0040D12C(C)
|
:0040D135 833D00FE410003           cmp dword ptr [0041FE00], 00000003
:0040D13C 7545                     jne 0040D183  //--------修改 75-74 即 jne->je

* Possible StringData Ref from Data Obj ->"DELUXE registration OK"  //-----7 条鱼！

                                 |
:0040D13E 6864E54100             push 0041E564
:0040D143 6892000000             push 00000092
:0040D148 8B4508                 mov eax, dword ptr [ebp+08]
:0040D14B 50                     push eax

* Reference To: USER32.SetDlgItemTextA, Ord:022Ch

                                 |
:0040D14C FF1514A24100           Call dword ptr [0041A214]
:0040D152 C605B1DD410001           mov byte ptr [0041DDB1], 01  //----
:0040D159 C605B2DD410001           mov byte ptr [0041DDB2], 01  //----
:0040D160 C605B3DD410001           mov byte ptr [0041DDB3], 01  //----
:0040D167 C605B4DD410001           mov byte ptr [0041DDB4], 01  //----
:0040D16E C605B5DD410001           mov byte ptr [0041DDB5], 01  //----
:0040D175 C605B6DD410001           mov byte ptr [0041DDB6], 01  //----
:0040D17C 33C0                     xor eax, eax
:0040D17E E9CD000000             jmp 0040D250

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D13C(C)  //-------------------------根据上面的值变化，也要修改。
|
:0040D183 6878104200             push 00421078
:0040D188 6892000000             push 00000092
:0040D18D 8B4D08                 mov ecx, dword ptr [ebp+08]
:0040D190 51                     push ecx

* Reference To: USER32.SetDlgItemTextA, Ord:022Ch

                                 |
:0040D191 FF1514A24100           Call dword ptr [0041A214]
:0040D197 33C0                     xor eax, eax
:0040D199 E9B2000000             jmp 0040D250

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040CFD1(C)
```

下面是参考 `dyiyd` 兄的提示写的：感谢 `dyiyd` 兄。

修改了上面的内容注册就是任意注册码了。而且是 `DELUXE`，7 条鱼！！！但是还有启动注册框 `NAG`，下面来去掉他！

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00404194(C)
|
:004041A0 833D00FE410000           cmp dword ptr [0041FE00], 00000000  //----0041FE00 是否为 0
:004041A7 740E                     je 004041B7  //-----等于 0 就跳，玩完了。 74->75 即 je->jne
:004041A9 C605B1DD410001           mov byte ptr [0041DDB1], 01
:004041B0 C605B2DD410001           mov byte ptr [0041DDB2], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004041A7(C)
|
:004041B7 6A03                     push 00000003
:004041B9 E822F6FFFF               call 004037E0
:004041BE 83C404                   add esp, 00000004
:004041C1 85C0                     test eax, eax
:004041C3 7434                     je 004041F9  //------>eax=0 就跳，玩完了。74->75 即 je->jne
:004041C5 C70500FE410003000000     mov dword ptr [0041FE00], 00000003  //-----使 0041FE00=3
:004041CF C605B1DD410001           mov byte ptr [0041DDB1], 01
:004041D6 C605B2DD410001           mov byte ptr [0041DDB2], 01
:004041DD C605B3DD410001           mov byte ptr [0041DDB3], 01
:004041E4 C605B4DD410001           mov byte ptr [0041DDB4], 01
:004041EB C605B5DD410001           mov byte ptr [0041DDB5], 01
:004041F2 C605B6DD410001           mov byte ptr [0041DDB6], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00404186(C), :004041C3(C)
|
:004041F9 833D00FE410000           cmp dword ptr [0041FE00], 00000000
:00404200 7410                     je 00404212
:00404202 837DFC00                 cmp dword ptr [ebp-04], 00000000
:00404206 740A                     je 00404212
:00404208 6A01                     push 0000001
:0040420A E86AFDFFFF               call 00403F79
```

所以只要使 `0041FE00`=`3` 就可以了，所以我们可以这么修改。

<------------完--------->

## 再论 FISH 精美屏保（`VER:0.99G`）暴力破解---`WD32ASM893` 版

请孙锋给提点意见！！！

作者：李海涛

Email：`lihaitao@xaonline.com`

先用 `WD32ASM8.93` 超级中文版进行反汇编，然后查找，会看到：

```
* Possible StringData Ref from Data Obj ->"SOFTWARE\Sachs\SachsAquarium"

                                 |
:004042F2 68D4024200             push 004202D4
:004042F7 8D4DA0                 lea ecx, dword ptr [ebp-60]
:004042FA 51                     push ecx
:004042FB E800DB0000             call 00411E00
:00404300 83C408                 add esp, 00000008
:00404303 C7057020420000000000   mov dword ptr [00422070], 00000000

  改：C7057020420003000000     mov dword ptr [00422070], 00000003

:0040430D 8D5590                 lea edx, dword ptr [ebp-70]
```



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
