
# Ardamax Keylogger 2.5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-398.htm`](https://www.pediy.com/kssd/pediy08/pediy8-398.htm)

**【破解日期】** 2006 年 11 月 5 日(传说今天是四狗齐贺月最圆--->20 时 58 分)  

**【破解作者】** 冷血书生  

**【作者邮箱】** 暂没  

**【作者主页】** `hxxp://www.126sohu.com`  

**【使用工具】** `OD`  

**【破解平台】** `Win9x/NT/2000/XP`  

**【软件名称】** `Ardamax Keylogger 2.5`  

**【下载地址】** `http://www.onlinedown.net/soft/22696.htm`  

**【软件简介】** 小巧实用方便的按键记录工具，它捕捉用户的按键信息并保存到 `LOG` 文件中，你可以以文本方式或网页形式查看该记录文件，有助于你发现自己离开时机器被动了什么手脚或监视您的孩子都干了些什么!  

**【软件大小】** `388KB`  

**【加壳方式】** 无  

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  

--------------------------------------------------------------------------------

**【破解内容】**

```
代码:

* * *

00405B38     E8 F24E0100           call TND.0041AA2F
00405B3D     51                   push ecx
00405B3E     8D85 FCFEFFFF        lea eax,dword ptr ss:[ebp-104]
00405B44     8BF4                 mov esi,esp
00405B46     50                   push eax
00405B47     E8 E34E0100          call TND.0041AA2F
00405B4C     E8 1EFFFFFF          call TND.00405A6F                       ; 关键 CALL
00405B51     85C0                 test eax,eax
00405B53     74 57                je short TND.00405BAC
00405B55     6A 40                push 40
00405B57     68 0C084600          push TND.0046080C                      ; Ardamax Keylogger
00405B5C     68 240F4600          push TND.00460F24                      ; Registration code 
is accepted. Thank you for registration!
00405B61     FF77 04              push dword ptr ds:[edi+4]
00405B64     FF15 6C954500        call dword ptr ds:[<&USER32.MessageBo>; 
USER32.MessageBoxA
00405B6A     8D85 FCFEFFFF        lea eax,dword ptr ss:[ebp-104]
00405B70     50                   push eax
00405B71     FF77 20              push dword ptr ds:[edi+20]
00405B74     E8 7B4E0100          call TND.0041A9F4
00405B79     6A 30                push 30
00405B7B     E8 C8060200          call TND.00426248
00405B80     85C0                 test eax,eax
00405B82     59                   pop ecx
00405B83     74 11                je short TND.00405B96
00405B85     8BCF                 mov ecx,edi
00405B87     F7D9                 neg ecx
00405B89     8D57 04              lea edx,dword ptr ds:[edi+4]
00405B8C     1BC9                 sbb ecx,ecx
00405B8E     23CA                 and ecx,edx
00405B90     51                   push ecx
00405B91     E8 6AF0FFFF          call TND.00404C00
00405B96     0FB745 08            movzx eax,word ptr ss:[ebp+8]
00405B9A     50                   push eax
00405B9B     FF77 04              push dword ptr ds:[edi+4]
00405B9E     FF15 AC934500        call dword ptr ds:[<&USER32.EndDialog>; USER32.EndDialog
00405BA4     5E                   pop esi
00405BA5     33C0                 xor eax,eax
00405BA7     5B                   pop ebx
00405BA8     C9                   leave
00405BA9     C2 0400              retn 4
00405BAC     6A 30                push 30
00405BAE     68 0C084600          push TND.0046080C                      ; Ardamax Keylogger
00405BB3     68 600F4600          push TND.00460F60                      ; Registration code 
or name is invalid. Please check all fields.
00405BB8   ^ E9 66FFFFFF          jmp TND.00405B23

///////////////////////////////////////////////////////////////////////////////////////////////////////////
跟进关键 CALL
///////////////////////////////////////////////////////////////////////////////////////////////////////////

00405A80     E8 DF490100          call TND.0041A464
00405A85     50                   push eax
00405A86     8D9D FCFEFFFF        lea ebx,dword ptr ss:[ebp-104]
00405A8C     E8 65FFFFFF          call TND.004059F6                       ; 算法 CALL
00405A91     8B7D 0C              mov edi,dword ptr ss:[ebp+C]
00405A94     8BC3                 mov eax,ebx                             ; 真码 
00405A96     50                   push eax
00405A97     57                   push edi
00405A98     FF15 C8904500        call dword ptr ds:[<&KERNEL32.lstrcmp>; kernel32.lstrcmpA
00405A9E     8BF0                 mov esi,eax
00405AA0     8B45 08              mov eax,dword ptr ss:[ebp+8]
00405AA3     F7DE                 neg esi

///////////////////////////////////////////////////////////////////////////////////////////////////////////
跟进算法 CALL
///////////////////////////////////////////////////////////////////////////////////////////////////////////

004059FD     8B35 C0904500        mov esi,dword ptr ds:[<&KERNEL32.lstr>; kernel32.lstrlenA
00405A03     57                   push edi
00405A04     BF 34234600          mov edi,TND.00462334                    ; 207144FA6AD570E
00405A09     57                   push edi
00405A0A     FFD6                 call esi
00405A0C     FF75 08              push dword ptr ss:[ebp+8]
00405A0F     8945 F8              mov dword ptr ss:[ebp-8],eax
00405A12     FFD6                 call esi
00405A14     8BF0                 mov esi,eax
00405A16     85F6                 test esi,esi
00405A18     8975 F4              mov dword ptr ss:[ebp-C],esi
00405A1B     75 04                jnz short TND.00405A21
00405A1D     8803                 mov byte ptr ds:[ebx],al
00405A1F     EB 48                jmp short TND.00405A69
00405A21     57                   push edi
00405A22     53                   push ebx
00405A23     FF15 BC904500        call dword ptr ds:[<&KERNEL32.lstrcpy>; kernel32.lstrcpyA
00405A29     8B45 F8              mov eax,dword ptr ss:[ebp-8]
00405A2C     3BF0                 cmp esi,eax
00405A2E     8975 FC              mov dword ptr ss:[ebp-4],esi
00405A31     7F 03                jg short TND.00405A36
00405A33     8945 FC              mov dword ptr ss:[ebp-4],eax
00405A36     33F6                 xor esi,esi
00405A38     3975 FC              cmp dword ptr ss:[ebp-4],esi
00405A3B     7E 2C                jle short TND.00405A69
00405A3D     8BC6                 mov eax,esi
00405A3F     99                   cdq
00405A40     F77D F8              idiv dword ptr ss:[ebp-8]               ; /固定字符串长度
00405A43     8BC6                 mov eax,esi
00405A45     6A 19                push 19
00405A47     5F                   pop edi
00405A48     8D0C1A               lea ecx,dword ptr ds:[edx+ebx]          ; 固定字符串" ")
00405A4B     99                   cdq
00405A4C     F77D F4              idiv dword ptr ss:[ebp-C]               ; /用户名长度
00405A4F     8B45 08              mov eax,dword ptr ss:[ebp+8]
00405A52     0FB60402             movzx eax,byte ptr ds:[edx+eax]         ; 取用户名 ASCII 值
00405A56     0FB611               movzx edx,byte ptr ds:[ecx]             ; 取固定字符串
00405A59     33C2                 xor eax,edx                             ; eax xor edx
00405A5B     99                   cdq
00405A5C     F7FF                 idiv edi                                ; /19
00405A5E     80C2 41              add dl,41                               ; dl +41
00405A61     46                   inc esi
00405A62     3B75 FC              cmp esi,dword ptr ss:[ebp-4]
00405A65     8811                 mov byte ptr ds:[ecx],dl                ; 保存结果
00405A67   ^ 7C D4                jl short TND.00405A3D
00405A69     5F                   pop edi
00405A6A     5E                   pop esi
00405A6B     C9                   leave
00405A6C     C2 0400              retn 4

* * *
```

**总结:**  

1) 用户名不足 `15` 位，循环至 `15` 次计算  

2) 用户名 `ASCII` 值 `XOR` 固定字符串 `207144FA6AD570E` 的 `ASCII` 值 `mod 19 + 41` =注册码  

--------------------------------------------------------------------------------

**【版权声明】** 本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢! 

`4dd466d0e1e0cc123afbe504da2462ca`
