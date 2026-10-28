
# [原创]Dll注入NetTransport 2.25.337

【文章标题】: Dll注入NetTransport 2.25.337

【文章作者】: wynney

【软件名称】: NetTransport 2.25.337 Unicode

【软件大小】: 3.46M

【下载地址】: 打包下载

【编写语言】: Microsoft Visual C++ 6.0

【作者声明】: 冲冲看雪的气氛

--------------------------------------------------------------------------------

## 【详细过程】

前言：前天看到NetTransport出了新版，而且是Unicode版本的，遂下来看了看，于是就有了下文

1、未注册，启动就有注册提示框

2、未注册，使用过程中不定时弹出注册框

### 一、定位注册框

F7配合F8很容易就找到掉用注册框的地方

```
00443EB9     E8 4272FFFF        call NetTrans.0043B100            ; 如果是的话那这里就是关键Call了
00443EBE     8B4C24 18          mov ecx,dword ptr ss:[esp+18]
00443EC2     8B41 F8            mov eax,dword ptr ds:[ecx-8]      ; [ecx-8]=[728776CC]，记住他！
00443EC5     85C0               test eax,eax
00443EC7     75 78              jnz short NetTrans.00443F41       ; 看看/跳过的话那不就不用注册了？
00443EC9     6A 00              push 0
00443ECB     8D8C24 50010000    lea ecx,dword ptr ss:[esp+150]
00443ED2     E8 79860200        call NetTrans.0046C550
00443ED7     8D8C24 4C010000    lea ecx,dword ptr ss:[esp+14C]
00443EDE     C68424 04100000 0B mov byte ptr ss:[esp+1004],0B
00443EE6     E8 F72C0800        call <jmp.&MFC42u.#2506>          ; 调用注册框
00443EEB     3D 55050000        cmp eax,555
```

我们发现`[ecx-8]=[728776CC]=1`才可以跳过去

小插曲：在`00443EB9`下F2，再F9发现程序直接退出，图省事，不想追其原因，改在`00443EB9`下硬件执行断点吧

重载，F9，中断在`00443EB9`，F7，来看看几段代码吧

```
0043B124     57                 push edi
0043B125     8BCE               mov ecx,esi
0043B127     E8 B0BD0800        call <jmp.&MFC42u.#2606>      ; 看看下面的注册表。。
0043B12C     8BBC24 90000000    mov edi,dword ptr ss:[esp+90] ; 呵呵，这个Call还真是关键哦
0043B133     85FF               test edi,edi
0043B135     74 06              je short NetTrans.0043B13D
0043B137     C707 00000000      mov dword ptr ds:[edi],0
0043B13D     6A 00              push 0
0043B13F     68 50534F00        push NetTrans.004F5350        ; UNICODE "Key"
0043B144     68 28534F00        push NetTrans.004F5328        ; UNICODE "SOFTWARE\Xi\NetXfer"
0043B149     8D4424 20          lea eax,dword ptr ss:[esp+20]
0043B14D     68 02000080        push 80000002
0043B152     50                 push eax
0043B153     B9 38B04F00        mov ecx,NetTrans.004FB038
0043B158     E8 D39B0000        call NetTrans.00444D30        ; 看到了吧，我们找对了
0043B15D     6A 00              push 0
0043B15F     68 14534F00        push NetTrans.004F5314        ; UNICODE "Signature"
0043B164     68 28534F00        push NetTrans.004F5328        ; UNICODE "SOFTWARE\Xi\NetXfer"
0043B169     8D4C24 1C          lea ecx,dword ptr ss:[esp+1C]
0043B16D     68 02000080        push 80000002
0043B172     51                 push ecx
0043B173     B9 38B04F00        mov ecx,NetTrans.004FB038
0043B2A8     8B41 F8            mov eax,dword ptr ds:[ecx-8]
0043B2AB     85C0               test eax,eax
0043B2AD     0F85 AA000000      jnz NetTrans.0043B35D
0043B2B3     8D5424 3C          lea edx,dword ptr ss:[esp+3C]
0043B2B7     8D8424 8C000000    lea eax,dword ptr ss:[esp+8C]
0043B2BE     52                 push edx
0043B2BF     8D8C24 94000000    lea ecx,dword ptr ss:[esp+94]
0043B2C6     50                 push eax
0043B2C7     51                 push ecx
0043B2C8     E8 CBBA0800        call <jmp.&MFC42u.#925>           ; 这里又是什么？
0043B2CD     8D9424 90000000    lea edx,dword ptr ss:[esp+90]
0043B2D4     68 00534F00        push NetTrans.004F5300            ; UNICODE ".key.kqm"
0043B2D9     8D4424 24          lea eax,dword ptr ss:[esp+24]
0043B2DD     52                 push edx
0043B2DE     50                 push eax
0043B2DF     C68424 90000000 05 mov byte ptr ss:[esp+90],5
0043B2E7     E8 ACBA0800        call <jmp.&MFC42u.#925>
0043B2EC     8B18               mov ebx,dword ptr ds:[eax]
0043B2EE     8D8C24 90000000    lea ecx,dword ptr ss:[esp+90]
0043B2F5     68 F4524F00        push NetTrans.004F52F4            ; UNICODE ".key"
0043B2FA     8D5424 20          lea edx,dword ptr ss:[esp+20]     ; 看样子像是注册成功后的key备份
0043B35F     8B42 F8            mov eax,dword ptr ds:[edx-8]
0043B362     85C0               test eax,eax
0043B364     75 22              jnz short NetTrans.0043B388
0043B366     57                 push edi
0043B367     56                 push esi                          ; key的名称
0043B368     68 E4524F00        push NetTrans.004F52E4            ; UNICODE "NT2REG"
0043B36D     8D4C24 30          lea ecx,dword ptr ss:[esp+30]
0043B371     E8 BA210800        call NetTrans.004BD530
0043B376     50                 push eax
0043B377     8D4C24 40          lea ecx,dword ptr ss:[esp+40]
0043B3B5     8B41 F8            mov eax,dword ptr ds:[ecx-8]
0043B3B8     85C0               test eax,eax
0043B3BA     0F85 AA000000      jnz NetTrans.0043B46A
0043B3C0     8D9424 8C000000    lea edx,dword ptr ss:[esp+8C]
0043B3C7     68 E4524F00        push NetTrans.004F52E4            ; UNICODE "NT2REG"
0043B3CC     8D8424 94000000    lea eax,dword ptr ss:[esp+94]
0043B3D3     52                 push edx
0043B3D4     50                 push eax
0043B3D5     E8 BEB90800        call <jmp.&MFC42u.#925>
0043B3DA     8D8C24 90000000    lea ecx,dword ptr ss:[esp+90]
0043B3E1     68 00534F00        push NetTrans.004F5300            ; UNICODE ".key.kqm"
0043B3E6     8D5424 20          lea edx,dword ptr ss:[esp+20]     ; 注册表备份
0043B3EA     51                 push ecx
0043B3EB     52                 push edx
0043B3EC     C68424 90000000 08 mov byte ptr ss:[esp+90],8
0043B3F4     E8 9FB90800        call <jmp.&MFC42u.#925>
0043B3F9     8B18               mov ebx,dword ptr ds:[eax]
0043B3FB     8D8424 90000000    lea eax,dword ptr ss:[esp+90]     ; 前提是要注册成功了才有
0043B402     68 F4524F00        push NetTrans.004F52F4            ; UNICODE ".key"
0043B407     8D4C24 24          lea ecx,dword ptr ss:[esp+24]
0043B40B     50                 push eax
```

看过上面3小段大家就应该知道，这里是关键

提示：看上面看似关键的跳转不是受`[ecx-8]`就是受`[edx-8]`的控制，似乎是有关联的[是与不是分析便知]

### 二、借助`MFC42u.DLL`注入

接下来就是慢慢分析了

```
0043B127     E8 B0BD0800        call <jmp.&MFC42u.#2606>          ; F7
004C6EDC   - FF25 88524D00      jmp dword ptr ds:[<&MFC42u.#2606>>; MFC42u.#2606，F8
727A49E4 M>  8B01               mov eax,dword ptr ds:[ecx]        ; 来到这里，发现正处于MFC42u.DLL的领空
727A49E6     8378 F8 00         cmp dword ptr ds:[eax-8],0        ; ds:[728776CC]=00000000
727A49EA     74 15              je short MFC42u.727A4A01          ; 好咯，看到上面就应该灵机一动了
727A49EC     8378 F4 00         cmp dword ptr ds:[eax-C],0        ; 到system32目录下找到MFC42u.DLL
727A49F0     7C 05              jl short MFC42u.727A49F7          ; 把它拷贝到程序目录
727A49F2   ^ E9 B5FFFFFF        jmp MFC42u.#5509                  ; 关掉OD，重新载入，以便于程序调用的是本目录下的MFC42u.DLL
727A49F7     68 F4D68772        push offset MFC42u.#6408
727A49FC     E8 91FDFFFF        call MFC42u.#861
727A4A01     C3                 retn
```

拷贝`MFC42u.DLL`到程序目录后，OD载入程序，F9跑一下，Alt+E，看看程序调用的是否是本目录下的`MFC42u.DLL`

OK，一切准备妥当，Patch这个`MFC42u.DLL`吧，Patch它的目的是为了使`[ecx-8]=[728776CC]=1`，`[728776C8]=00000001`，在`MFC42u.DLL`找块空白代码

我这里找的是`72843C71`（找不会被重定位的空白）

```
727A49E4 M>  8B01               mov eax,dword ptr ds:[ecx]
727A49E6     E9 86F20900        jmp MFC42u.72843C71               ; 跳向Patch位置
727A49EB     90                 nop
727A49EC     8378 F4 00         cmp dword ptr ds:[eax-C],0        ; [eax-0C]也必须=1
727A49F0     7C 05              jl short MFC42u.727A49F7
727A49F2   ^ E9 B5FFFFFF        jmp MFC42u.#5509
727A49F7     68 F4D68772        push offset MFC42u.#6408
727A49FC     E8 91FDFFFF        call MFC42u.#861
727A4A01     C3                 retn
72843C71     C740 F8 01000000   mov dword ptr ds:[eax-8],1        ; 先赋值1
72843C78     8378 F8 00         cmp dword ptr ds:[eax-8],0        ; 原代码挪下
72843C7C   ^ 0F84 7F0DF6FF      je MFC42u.727A4A01                ; 原代码挪下
```
