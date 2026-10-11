
## 反汇编代码分析

```asm
0049CE87   |.   EB 49                 |jmp short Delphi 编.0049CED2  

0049CE89   |>   8BCB                |mov ecx,ebx       计算完机器码的 6 位后到这里继续。N1 入 ECX  

0049CE8B   |.   C1E1 02             |shl ecx,2         N1 左移 2 次记为 N3  

0049CE8E   |.   8BC1                |mov eax,ecx       N3 入 EAX  

0049CE90   |.   83C0 08             |add eax,8         N3+8 结果记为 N4  

0049CE93   |.   8B55 FC             |mov edx,dword ptr ss:[ebp-4]        预置数入 EDX  

0049CE96   |.   0FB6541A FF         |movzx edx,byte ptr ds:[edx+ebx-1]   取预置数的一位的十六进制记为 D1  

0049CE9B   |.   8BFA                |mov edi,edx                   D1 入 EDI  

0049CE9D   |.   C1E2 03             |shl edx,3                       D1 左移 3 次记为 D2  

0049CEA0   |.   2BD7                |sub edx,edi                       D2-D1 结果记为 D3  

0049CEA2   |.   03C2                |add eax,edx                       N4+D3 结果记为 D4  

0049CEA4   |.   8B55 F4             |mov edx,dword ptr ss:[ebp-C]        预置数的累加结果入 EDX 记为 D5  

0049CEA7   |.   C1E2 03             |shl edx,3                       D5 左移 3 次结果记为 D6  

0049CEAA   |.   03C2                |add eax,edx                       D4+D6 结果记为 D7  

0049CEAC   |.   8B55 F0             |mov edx,dword ptr ss:[ebp-10]       机器码的累加结果入 EDX 记为 D8  

0049CEAF   |.   8D14D2              |lea edx,dword ptr ds:[edx+edx*8]    D8+D8*8 结果记为 D9  

0049CEB2   |.   03C2                |add eax,edx                       D7+D9 结果记为 D10  

0049CEB4   |.   8BD3                |mov edx,ebx                       N1 入 EDX  

0049CEB6   |.   0FAFD3              |imul edx,ebx                      N1*N1 结果记为 D11  

0049CEB9   |.   03C2                |add eax,edx                       D10+D11 结果记为 D12  

0049CEBB   |.   0FAF4D EC           |imul ecx,dword ptr ss:[ebp-14]      N3*B1 结果记为 D13  

0049CEBF   |.   83C1 17             |add ecx,17                      D13+17 结果记为 D14  

0049CEC2   |.   F7E9                |imul ecx                      D12*D14 结果记为 D15  

0049CEC4   |.   BA 77000000         |mov edx,77                      77 入 EDX  

0049CEC9   |.   2BD3                |sub edx,ebx                       77-N1 结果记为 D16  

0049CECB   |.   8BCA                |mov ecx,edx  

0049CECD   |.   99                    |cdq  

0049CECE   |.   F7F9                |idiv ecx                      D15/D16 商记为 D17,余数记为 D18  

0049CED0   |.   8BFA                |mov edi,edx                   D18 入 EDI  

0049CED2   |>   83FF 23             |cmp edi,23            比较 S19 是否大于 23。比较 D18 是否大于 23  

0049CED5   |.   7D 06                 |jge short Delphi 编.0049CEDD  大于则跳，否则不跳  

0049CED7   |.   8D443B 22           |lea eax,dword ptr ds:[ebx+edi+22] N1+S19+22 记为 S20,N1+D18+22 记为 D19  

0049CEDB   |.   8BF8                |mov edi,eax  

0049CEDD   |>   8D45 E4             |lea eax,dword ptr ss:[ebp-1C]  

0049CEE0   |.   8BD7                |mov edx,edi  

0049CEE2   |.   E8 FD6DF6FF         |call Delphi 编.00403CE4  

0049CEE7   |.   8B55 E4             |mov edx,dword ptr ss:[ebp-1C]  

0049CEEA   |.   8D45 E8             |lea eax,dword ptr ss:[ebp-18]  

0049CEED   |.   E8 D26EF6FF         |call Delphi 编.00403DC4  

0049CEF2   |.   43                    |inc ebx                 记数器加一  

0049CEF3   |.   4E                    |dec esi                  长度减一  

0049CEF4   |.^ 0F85 33FFFFFF         \jnz Delphi 编.0049CE2D          没有计算完则继续。  

0049CEFA   |>   8B45 08             mov eax,dword ptr ss:[ebp+8]  

0049CEFD   |.   8B55 E8             mov edx,dword ptr ss:[ebp-18]  最后这里的内存地址存放的就是阅读密码  

0049CF00   |.   E8 8B6CF6FF         call Delphi 编.00403B90  

0049CF05   |.   33C0                xor eax,eax  

0049CF07   |.   5A                    pop edx  

0049CF08   |.   59                    pop ecx  

0049CF09   |.   59                    pop ecx  

0049CF0A   |.   64:8910             mov dword ptr fs:[eax],edx  

0049CF0D   |.   68 34CF4900         push Delphi 编.0049CF34  

0049CF12   |>   8D45 E4             lea eax,dword ptr ss:[ebp-1C]  

0049CF15   |.   BA 02000000         mov edx,2  

0049CF1A   |.   E8 416CF6FF         call Delphi 编.00403B60  

0049CF1F   |.   8D45 F8             lea eax,dword ptr ss:[ebp-8]  

0049CF22   |.   BA 02000000         mov edx,2  

0049CF27   |.   E8 346CF6FF         call Delphi 编.00403B60  

0049CF2C   |\   C3                    retn  

0049CF2D    .^ E9 A266F6FF           jmp Delphi 编.004035D4  

0049CF32    .^ EB DE                jmp short Delphi 编.0049CF12  

0049CF34    .   5F                    pop edi  

0049CF35    .   5E                    pop esi  

0049CF36    .   5B                    pop ebx  

0049CF37    .   8BE5                mov esp,ebp  

0049CF39    .   5D                    pop ebp  

0049CF3A    .   C2 0400             retn 4  
```

## 破解总结

算法挺简单的，呵呵。现附上它的 `VB` 注册机源码，高手见笑了。

```vb
Private Sub Command1_Click() 

Dim A, B, C, D, E, F, G, H, I, J, K 

If Text1.Text = "" Or Len(Text1.Text) < 6 Then 

MsgBox "sorry!please input the serial" 

Exit Sub 

End If 

For A = 1 To Len(Text1.Text) 

B = B + Asc(Mid(Text1.Text, A, 1)) 

Next A 

C = "wsy54321`" 

K = Len(C) 

For D = 1 To Len(C) 

E = E + Asc(Mid(C, D, 1)) 

Next D 

For F = 1 To Len(Text1.Text) - 6 

If F = 6 Then 

For I = F To Len(C) 

H = ((((((I * 4 + 8) + ((Asc(Mid(C, I, 1)) * 8) - Asc(Mid(C, I, 1)))) + (B * 8)) + (E + E * 8)) + (I * I)) * (I * 4 * 68 + 23)) Mod (119 - I) 

If H > 23 Then 

J = J & Chr(H) 

Else 

J = J & Chr(I + H + 34) 

End If 

K = K - 1 

If K = 0 Then 

Text2.Text = J 

Exit Sub 

End If 

Next I 

Else 

G = ((((Asc(Mid(Text1.Text, F + 4, 1)) * ((F + 5) * Asc(Mid(C, F, 1)) + F)) + (F * 2 * 68)) + (((F * 2 * F) + 13) * B)) + (((F + F * 2) + 12) * E)) Mod (114 - F) 

K = K - 1 

If G > 23 Then 

J = J & Chr(G) 

Else 

J = J & Chr(F + G + 34) 

End If 

End If 

Next F 

End Sub 
```

本注册机在 `Windows XP SP2` `VB6` 下运行通过。

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
