
# 注册机

```asm
:004355CD 8917                    mov dword ptr [edi], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004355B5(U), :004355BF(U), :004355C9(U)

 |

 :004355CF 83C704                  add edi, 00000004

:004355D2 33C0                    xor eax, eax

:004355D4 49                      dec ecx

:004355D5 740A                    je 004355E1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00435569(C)

 |

 :004355D7 33C0                    xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004355DF(C)

 |

 :004355D9 8907                    mov dword ptr [edi], eax

:004355DB 83C704                  add edi, 00000004

:004355DE 49                      dec ecx

:004355DF 75F8                    jne 004355D9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004355D5(C)

 |

 :004355E1 83E303                  and ebx, 00000003

:004355E4 7585                    jne 0043556B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00435556(C)

 |

 :004355E6 8B442410                mov eax, dword ptr [esp+10]

:004355EA 5B                      pop ebx

:004355EB 5E                      pop esi

:004355EC 5F                      pop edi

:004355ED C3                      ret
```

转载本文请注明出处，并保持文章的完整性。本文不得用于任何商业场合，仅供学习和参考。

作者： Fpc（感谢直接和间接提供帮助的所有朋友）

工具： BC++ Builder 5.0(对ASM的兼容性非常不错，可惜我用的不熟)

写教程真是很累的一件事，想起众多前辈的妙文，就知道他们付出了什么。不过在这过程中是学到了一些，尤其是那些自己原来不太明白之处，要去想清楚，否则错误连连，走在街上肯定被痛扁。有意把这一篇写的很罗嗦，只是希望大家从中学到知识，如果再写，一定会是很简练。当然水平还是基础级的，需要的只是耐心！！

下面是注册机：

```cpp
//---------------------------------------------------------------------------
// KeyGen for WinAmp 2.11 by Fpc
 // edtName为输入注册姓名的文本框，edtCode为显示注册码的文本框（只读）
void __fastcall TForm1::edtNameChange(TObject *Sender)
{
 char name[255], code[255] = " ";
int i;
 long value = 0;
 for(i=0;i<255;i++)
        name[i]='\0x0';
if(edtName->Text.IsEmpty()){
         edtCode->Text = "";
        return;
         }
strcpy(name, edtName->Text.c_str());
_asm
 {
     lea esi, name
    xor edi, edi
     xor ebx, ebx
loop1:
     movzx eax, byte ptr [Esi+2]
    movzx ecx, byte ptr [Esi]
    movzx edx, byte ptr [Esi+1]
    xor eax, ecx
     movzx ecx, byte ptr [Esi+3]
    not ecx
     not eax
    xor ecx, edx
     add eax, ecx
    mov ecx, edi
     and ecx, 0x0000000F
    shl eax, cl
     test eax, eax
    jl iflittle
     add ebx, eax
    jmp nextchar
iflittle:
     sub ebx, eax
nextchar:
     inc edi
     inc edi
    inc esi
     cmp byte ptr [esi], 00
    jne loop1            // the first loop
mov eax, ebx
    mov ecx, 0x000F4240
     imul eax, 0x00000025
    cdq
     idiv ecx
    mov ecx, edx
     mov esi, edx
    imul ecx, 0x00000064
     xor edi, edi
loop2:
     test esi, esi
     je over
    mov eax, esi
    push 0x00000064
     cdq
    pop ebx
    idiv ebx
     mov eax, esi
    push ebx
     pop esi
    add edi, edx
     cdq
    idiv esi
     mov esi, eax
    jmp loop2                // the second loop
over:
     lea eax, dword ptr [edi+2*edi]
    push 0x00000004
     cdq
    pop esi
     idiv esi
    push 0x00000064
     pop esi
    cdq
     idiv esi
    mov eax, edx
     add eax, ecx
      mov value,eax
 }
   wsprintf(code, "%ld", value);
  edtCode->Text = code;
 }
 //---------------------------------------------------------------------------
// 以上经编译通过，没有问题
```

Fpc @2001/03/26

<DIV ALIGN='right'>----------------------------------------------

没有最好，只有最差！

希望FCG不会最差！DIV>
