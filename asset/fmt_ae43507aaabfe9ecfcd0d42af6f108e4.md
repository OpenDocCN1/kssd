
```
:004ACB38 8B55FC                 mov edx, dword ptr [ebp-04]

:004ACB3B 8BC7                   mov eax, edi

:004ACB3D E8E26BF7FF             call 00423724

:004ACB42 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACB45 8B06                   mov eax, dword ptr [esi]

:004ACB47 8BB8F4010000           mov edi, dword ptr [eax+000001F4]

:004ACB4D 8BC7                   mov eax, edi

:004ACB4F E8A06BF7FF             call 004236F4

:004ACB54 8B45F8                 mov eax, dword ptr [ebp-08]

:004ACB57 8D55FC                 lea edx, dword ptr [ebp-04]

:004ACB5A E829B0F5FF             call 00407B88====>注册码[进行预处理，处理同用户名的处理]

:004ACB5F 8B55FC                 mov edx, dword ptr [ebp-04]

:004ACB62 8BC7                   mov eax, edi

:004ACB64 E8BB6BF7FF             call 00423724

:004ACB69 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACB6C 8B06                   mov eax, dword ptr [esi]

:004ACB6E 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACB74 E87B6BF7FF             call 004236F4

:004ACB79 8B45F8                 mov eax, dword ptr [ebp-08]

:004ACB7C E8CF8AFEFF             call 00495650====>用户名黑名单检查

:004ACB81 84C0                   test al, al========>al 必须为 0

:004ACB83 743C                   je 004ACBC1

\==============>Black List[黑名单]<=================

'Asteriks Asteriks'
'Ruslan Raymond Morris'
'Elizabeth Wilcox'
'kathleen pelikan'
'LEUNG KAM FUNG EVA'
'BRYAN MCBRIDE'
'Wravldtrebra CMeafnvnsianwg'
'BRUSZAZ'
'Mfagrftdisne AVtatcyeuc'
'vcivrbgnimneihac shratawg'
'Duayntireelw AHtidlvl'
'Dmanvbivdc XCsadurdtlye'
'GAAESGPCATRHIUNJ TSGPYAHREIDN'
'Esldafignhe'
'Dsatvhivdt sStcnhumdesn'
'Ttihm7odtbhiys AWheclylss'
'Cclrafusdcee JPuoatyvviun'
'Wtavrurdesnt nScaedgltedr'
'Gcefotrgghej kTjhhogmfpdssorn'
'srybfytjusmc tsnyifkthuvm'

\=============>End Of Black List<================

\* Possible StringData Ref from Code Obj ->"Sorry, your registration name "
                                        ->"("
                                        |

:004ACB85 68F4CD4A00             push 004ACDF4

:004ACB8A 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACB8D 8B06                   mov eax, dword ptr [esi]

:004ACB8F 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACB95 E85A6BF7FF             call 004236F4

:004ACB9A FF75F8                 push [ebp-08]

\* Possible StringData Ref from Code Obj ->") is found on the "Black List".

                                        |

:004ACB9D 681CCE4A00             push 004ACE1C

\* Possible StringData Ref from Code Obj ->"If you have any questions, please, "
                                        ->"contact KS-Soft: line1@ks-soft.net; "
                                        ->"line2@ks-soft.net"
                                        |

:004ACBA2 6848CE4A00             push 004ACE48

:004ACBA7 8D45FC                 lea eax, dword ptr [ebp-04]

:004ACBAA BA04000000             mov edx, 00000004

:004ACBAF E85474F5FF             call 00404008

:004ACBB4 8B45FC                 mov eax, dword ptr [ebp-04]

:004ACBB7 E8E480F9FF             call 00444CA0

:004ACBBC E9EF010000             jmp 004ACDB0

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004ACB83(C)

\========>Come Here,Here is right!!!======>

:004ACBC1 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACBC4 8B06                   mov eax, dword ptr [esi]

:004ACBC6 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACBCC E8236BF7FF             call 004236F4

:004ACBD1 837DF800               cmp dword ptr [ebp-08], 00000000

:004ACBD5 0F84CB010000           je 004ACDA6=====>用户名是否存在

:004ACBDB 8D55F4                 lea edx, dword ptr [ebp-0C]

:004ACBDE 8B06                   mov eax, dword ptr [esi]

:004ACBE0 8B80F4010000           mov eax, dword ptr [eax+000001F4]

:004ACBE6 E8096BF7FF             call 004236F4

:004ACBEB 837DF400               cmp dword ptr [ebp-0C], 00000000

:004ACBEF 0F84B1010000           je 004ACDA6=====>注册码是否存在

:004ACBF5 8D55F0                 lea edx, dword ptr [ebp-10]

:004ACBF8 8B06                   mov eax, dword ptr [esi]

:004ACBFA 8B80F0010000           mov eax, dword ptr [eax+000001F0]

:004ACC00 E8EF6AF7FF             call 004236F4

:004ACC05 8B45F0                 mov eax, dword ptr [ebp-10]

:004ACC08 E80713FFFF             call 0049DF14====>用户名[有效长度 0x40]，并产生 Buf2

:004ACC0D 8BF8                   mov edi, eax

\===========================
$50=length(Name)*8;
$00=length(Name) div 32;
\===========================

:004ACC0F 8D55F0                 lea edx, dword ptr [ebp-10]

:004ACC12 8B06                   mov eax, dword ptr [esi]

:004ACC14 8B80F4010000           mov eax, dword ptr [eax+000001F4]

:004ACC1A E8D56AF7FF             call 004236F4

:004ACC1F 8B45F0                 mov eax, dword ptr [ebp-10]

:004ACC22 E88913FFFF             call 0049DFB0====>注册码[必须是十六进制的字符串]
                                        ========>中间不能有空格
                                        ========>并产生 Buf1

:004ACC27 663BF8                 cmp di, ax========>??是不是长度比较呢??

:004ACC2A 0F8576010000           jne 004ACDA6

:004ACC30 A198224E00             mov eax, dword ptr [004E2298]

:004ACC35 BAFF010000             mov edx, 000001FF

:004ACC3A E8B912FFFF             call 0049DEF8=====>累加和 Buf1[0..$1FF]

:004ACC3F 8BF8                   mov edi, eax

:004ACC41 A19C214E00             mov eax, dword ptr [004E219C]

:004ACC46 BAFF010000             mov edx, 000001FF

:004ACC4B E8A812FFFF             call 0049DEF8=====>累加和 Buf2[0..$1FF]

:004ACC50 3BF8                   cmp edi, eax

:004ACC52 0F854E010000           jne 004ACDA6

:004ACC58 A198224E00             mov eax, dword ptr [004E2298]

:004ACC5D 83C007                 add eax, 00000007

:004ACC60 8B00                   mov eax, dword ptr [eax]

:004ACC62 8B159C214E00           mov edx, dword ptr [004E219C]

:004ACC68 83C207                 add edx, 00000007

:004ACC6B 3B02                   cmp eax, dword ptr [edx]====>要求第 7 开始的双字必须相等

:004ACC6D 0F8533010000           jne 004ACDA6======

:004ACC73 8D55F8                 lea edx, dword ptr [ebp-08]

:004ACC76 8B06                   mov eax, dword ptr [esi]
```
