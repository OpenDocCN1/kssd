
# PolyView v3.61

> 原文：[`www.pediy.com/kssd/pediy03/forum1214.htm`](https://www.pediy.com/kssd/pediy03/forum1214.htm)

下中断`bpx getwindowtexta`，拦下后`F12`两下到这里

\* Reference To: `USER32.GetWindowTextA`, Ord:`015Eh`

\----------------------------------------------------------------------

```
|
|
:00476949 8986E4030000       mov dword ptr [esi+000003E4], eax
:0047694F 8D442478          lea eax, dword ptr [esp+78]
:00476953 50               push eax
:00476954 8BCF              mov ecx, edi
:00476956 E8D42C0A00        call 0051962F
:0047695B 8B442474    mov eax, dword ptr [esp+74]<--假注册码
:0047695F 8B0F                mov ecx, dword ptr [edi]<--用户名
:00476961 50                   push eax
:00476962 51                   push ecx
:00476963 8986E0030000           mov dword ptr [esi+000003E0], eax
:00476969 E84222FEFF               call 00458BB0<---跟进去
\----------------------------------------------------------------------
```

\* Referenced by a CALL at Addresses:

```
|:004088EC  , :00408D8B  , :00409C7A  , :0046CF67  , :00472BF8  
|:004736D3  , :004742CF  , :004768D5  , :00476969  , :004784CB  
|:004790E4  , :004794E9  , :00479598  , :0049D999  , :0049DB51  
|:004BD27B  , :00532D51  
|
:00458BB0 64A100000000            mov eax, dword ptr fs:[00000000]
\* Possible Reference to Menu: MenuID_00FF

                                  |

:00458BB6 6AFF                      push FFFFFFFF
:00458BB8 68B1265400                  push 005426B1
:00458BBD 50                          push eax
:00458BBE 64892500000000              mov dword ptr fs:[00000000], esp
:00458BC5 81EC24010000                sub esp, 00000124
:00458BCB 53                          push ebx
:00458BCC 55                          push ebp
:00458BCD 56                          push esi
:00458BCE 57                          push edi
:00458BCF 8BBC2444010000              mov edi, dword ptr [esp+00000144]
:00458BD6 6880DF5900                  push 0059DF80
:00458BDB 57                          push edi
:00458BDC E87F020000                  call 00458E60<--计算核心
:00458BE1 8B9C2450010000              mov ebx, dword ptr [esp+00000150]
:00458BE8 83C408                      add esp, 00000008
:00458BEB 3BC3                  cmp eax, ebx<--| EAX 为正确的注册码
:00458BED 0F84AD000000      je 00458CA0    | EBX 为假注册码
                                        | 俱为 16 进制形式

                          至此如果跳转则注册成功

  crack by lancelot  2001.8.11  4dd466d0e1e0cc123afbe504da2462ca
```
