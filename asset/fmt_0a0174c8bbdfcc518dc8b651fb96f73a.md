
# 加密文件的软件

> 原文：[`www.pediy.com/kssd/pediy05/pediy50307.htm`](https://www.pediy.com/kssd/pediy05/pediy50307.htm)

破解软件:一个加密文件的软件,电脑爱好者上曾经有过介绍

破解人:powerboy

软件保护介绍:软件属于注册码保护但是特别的是注册码比较是在网上进行的认证,所以进行注册码破解显然是不可能,只能爆破.

首先,查看软件是否加壳(可能是作者对自己软件的保护很有信心所以连壳都没加,VC 编译)

然后,进入软件进行一次注册,输入:

```
NAME:powerboy

SN:1234567890123456
```

(软件告诉你了注册码要 16 位)并且有很多提示.

查找有关的注册提示,很有帮助.

首先看到"注册失败"有门看看!

```
:00418226 E8C5820000                       Call 004204F0
:0041822B 6A01                             push 00000001
:0041822D 8D4E64                           lea ecx, dword ptr [esi+64]
:00418230 5F                               pop edi
:00418231 50                               push eax
:00418232 897DFC                           mov dword ptr [ebp-04], edi

* Reference To: MFC42.Ordinal:035A, Ord:035Ah
                                                |
:00418235 E8D0800000                       Call 0042030A
:0041823A 834DFCFF                         or dword ptr [ebp-04], FFFFFFFF
:0041823E 8D4DF0                           lea ecx, dword ptr [ebp-10]

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                                                |
:00418241 E80C7E0000                       Call 00420052
:00418246 8BCE                             mov ecx, esi
:00418248 E840020000                       call 0041848D----------->这里是个关键的 CALL(要跟进去看看一会你就知道了)
:0041824D 85C0                             test eax, eax
:0041824F 752F                             jne 00418280------------>既然不跳是错误,那跳呢!!!!
:00418251 50                               push eax
:00418252 50                               push eax

* Possible StringData Ref from Data Obj ->"注册失败"
                                                |
:00418253 68201F4300                       push 00431F20

* Reference To: MFC42.Ordinal:04B0, Ord:04B0h
                                                |
:00418258 E8CB800000                       Call 00420328
:0041825D E9DA000000                       jmp 0041833C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004181E9(C), :004181EE(C)
:00418262 6A00                             push 00000000
:00418264 6A00                             push 00000000

* Possible StringData Ref from Data Obj ->"注册码类型错误"
                                                |
:00418266 68101F4300                       push 00431F10

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00418185(U), :00418196(U)
|
* Reference To: MFC42.Ordinal:04B0, Ord:04B0h
                                                |
:0041826B E8B8800000                       Call 00420328
:00418270 8D8E14010000                     lea ecx, dword ptr [esi+00000114]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00418170(U)
|
* Reference To: MFC42.Ordinal:175D, Ord:175Dh
                                                |
:00418276 E8ED7F0000                       Call 00420268
:0041827B E9BC000000                       jmp 0041833C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041824F(C)
|
:00418280 8BCE                             mov ecx, esi\------------------------->跳到这里了,向下看看!
:00418282 E8E2060000                       call 00418969
:00418287 85C0                             test eax, eax
:00418289 0F84AD000000                     je 0041833C
:0041828F 8BCE                             mov ecx, esi
:00418291 E884040000                       call 0041871A
:00418296 85C0                             test eax, eax
:00418298 0F849E000000                     je 0041833C
:0041829E 89BBC8080000                     mov dword ptr [ebx+000008C8], edi
:004182A4 8DBEA4020000                     lea edi, dword ptr [esi+000002A4]
:004182AA 57                               push edi
:004182AB 8D45F0                           lea eax, dword ptr [ebp-10]

* Possible StringData Ref from Data Obj ->"***** - "  软件的名称啊!
                                                |
:004182AE 68CC104300                       push 004310CC
:004182B3 50                               push eax

* Reference To: MFC42.Ordinal:039E, Ord:039Eh
                                                |
:004182B4 E857800000                       Call 00420310
:004182B9 FF30                             push dword ptr [eax]
:004182BB 8BCB                             mov ecx, ebx
:004182BD C745FC02000000                   mov [ebp-04], 00000002

* Reference To: MFC42.Ordinal:1837, Ord:1837h
                                                |
:004182C4 E8B77F0000                       Call 00420280
:004182C9 834DFCFF                         or dword ptr [ebp-04], FFFFFFFF
:004182CD 8D4DF0                           lea ecx, dword ptr [ebp-10]

* Reference To: MFC42.Ordinal:0320, Ord:0320h
                                                |
:004182D0 E87D7D0000                       Call 00420052

* Possible Reference to Dialog: DialogID_0092, CONTROL_ID:03E9, ""
                                                |
:004182D5 68E9030000                       push 000003E9
:004182DA FF7320                           push [ebx+20]

* Reference To: USER32.KillTimer, Ord:0195h
                                                |
:004182DD FF1520584200                     Call dword ptr [00425820]
:004182E3 57                               push edi
:004182E4 8D45E4                           lea eax, dword ptr [ebp-1C]

* Possible StringData Ref from Data Obj ->"恭喜你 "--------------------->看到这个了吗!软件替你说了^\_^
                                                |

==========================================================================

跟入上面的关键 CALL........

* Referenced by a CALL at Address:
|:00418248 
|
:0041848D B87C3D4200                       mov eax, 00423D7C

* Reference To: MSVCRT._EH_prolog, Ord:0042h
                                                |
:00418492 E8A9820000                       Call 00420740
:00418497 81EC80000000                      sub esp, 00000080
:0041849D 53                               push ebx
:0041849E 56                               push esi
:0041849F 57                               push edi
:004184A0 6A40                              push 00000040

* Possible StringData Ref from Data Obj ->"用户注册通知"
                                                |
:004184A2 6828224300                       push 00432228
:004184A7 8BD9                             mov ebx, ecx

* Possible StringData Ref from Data Obj ->"亲爱的用户! 3.0 以上版本必须进行网上激活,    
请稍"
                                                ->"等片刻既可完成整个注册验证过程."
                                                |
:004184A9 68D8214300                       push 004321D8

* Reference To: MFC42.Ordinal:1080, Ord:1080h
                                                |
:004184AE E8AF7D0000                       Call 00420262
:004184B3 E8B29FFEFF                       call 0040246A
:004184B8 8BF8                             mov edi, eax

* Possible StringData Ref from Data Obj ->"\\"
                                                |
:004184BA 6844014300                       push 00430144
:004184BF 8D87D43A0000                     lea eax, dword ptr [edi+00003AD4]
:004184C5 50                               push eax
:004184C6 8D45E0                           lea eax, dword ptr [ebp-20]
:004184C9 50                               push eax

* Reference To: MFC42.Ordinal:039C, Ord:039Ch
                                                |
:004184CA E82F7E0000                       Call 004202FE
:004184CF 8365FC00                         and dword ptr [ebp-04], 00000000
:004184D3 8D4B60                           lea ecx, dword ptr [ebx+60]
:004184D6 51                               push ecx
:004184D7 50                               push eax
:004184D8 8D45E4                           lea eax, dword ptr [ebp-1C]
:004184DB 50                               push eax

* Reference To: MFC42.Ordinal:039A, Ord:039Ah
                                                |
:004184DC E8177E0000                       Call 004202F8

* Possible StringData Ref from Data Obj ->".dat"
                                                |
:004184E1 BED0214300                       mov esi, 004321D0
:004184E6 C645FC01                         mov [ebp-04], 01
:004184EA 56                               push esi
:004184EB 50                               push eax
:004184EC 8D45EC                           lea eax, dword ptr [ebp-14]
:004184EF 50                               push eax
```
