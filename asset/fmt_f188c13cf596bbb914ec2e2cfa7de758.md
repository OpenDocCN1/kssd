
```
004455AC      6. ----->  |.  E8 6587FEFF      call winimage.0042DD16                ;  -----> show dialogue  

004455B1             |.  83C4 10          add esp,10  

004455B4             |.  56               push esi                              ; /Result  

004455B5             |.  FF75 08          push dword ptr ss:[ebp+8]              ; |hWnd  

004455B8             |.  FF15 38354700    call dword ptr ds:[<&USER32.EndDialog>]; \EndDialog  

004455BE             |.  8BC6             mov eax,esi  

004455C0             |.  EB 39            jmp short winimage.004455FB  

004455C2            >  FF75 08          push dword ptr ss:[ebp+8]              ;  Case 110 of switch 004454A3  

004455C5             |.  E8 4882FEFF      call winimage.0042D812  

004455CA             |.  833D F8E54800 00 cmp dword ptr ds:[48E5F8],0  

004455D1             |.  59               pop ecx  

004455D2             |.  74 24            je short winimage.004455F8  

004455D4             |.  8B35 70354700    mov esi,dword ptr ds:[<&USER32.SetDlgI>;  USER32.SetDlgItemTextA  

004455DA             |.  68 B8DE4800      push winimage.0048DEB8                 ; /Text = "sadf"  

004455DF             |.  68 16080000      push 816                              ; |ControlID = 816 (2070.)  

004455E4             |.  FF75 08          push dword ptr ss:[ebp+8]              ; |hWnd  

004455E7             |.  FFD6             call esi                              ; \SetDlgItemTextA  

004455E9             |.  68 38E24800      push winimage.0048E238                 ; /Text = "dffdf"  

004455EE             |.  68 17080000      push 817                              ; |ControlID = 817 (2071.)  

004455F3             |.  FF75 08          push dword ptr ss:[ebp+8]              ; |hWnd  

004455F6             |.  FFD6             call esi                              ; \SetDlgItemTextA  

004455F8            >  33C0             xor eax,eax  

004455FA             |.  40               inc eax  

004455FB            >  5E               pop esi  

004455FC             |.  5D               pop ebp  

004455FD           \  C2 1000          ret 10  
```

这段子程序中有六个值得注意的地方，分别以箭头 1-6 标注。箭头 1 和箭头 2 都是调用 `GetDlgItemTextA()`，分别是读取用户名（`name`）和注册号（`Registration Code`）。反复调试之后，发现箭头 3 处非常可疑，不出预料应该是判断序列号的地方（ye！^\_^）。箭头 6 是一个函数调用，经证明，是信息对话框的显示。箭头 4 是一个跳转，他出现在信息对话框的显示的前面不远处，应该是决定信息框的内容的，即注册成功或失败。经过我的反复多次调试，我的猜测是正确的。而箭头 5 就是注入注册失败的信息，“`Wrong Registration Key！`”。找到大方向之后就要在细节上有所发现了。

进入箭头 3 的函数里，先是两个函数，测量用户名和注册号的长度，再把用户名和注册号字母转换大写，因此程序是对字母大小写不敏感的！。接着用用户名来生成注册号，这应该是比较传统的序列号生成方法。生成方法如下：

```
0044C6D7       |> /8BC1            /mov eax,ecx  
0044C6D9       |. |6A 0E           |push 0E  
0044C6DB       |. |99             |cdq  
0044C6DC       |. |5B             |pop ebx  
0044C6DD       |. |F7FB           |idiv ebx  
0044C6DF       |. |85D2           |test edx,edx  
0044C6E1       |. |75 03          |jnz short winimage.0044C6E6  
0044C6E3       |. |6A 27          |push 27  
0044C6E5       |. |5E            |pop esi  
0044C6E6       |> |8D41 03        |lea eax,dword ptr ds:[ecx+3]  
0044C6E9 1.---->   |. |0FB61407      |movzx edx,byte ptr ds:[edi+eax]                ;  MOVE CHARACTER  
0044C6ED       |. |0FAFD6         |imul edx,esi  
0044C6F0 2.---->   |. |0155 FC        |add dword ptr ss:[ebp-4],edx  
0044C6F3       |. |6A 0E          |push 0E  
0044C6F5       |. |99            |cdq  
0044C6F6       |. |5B            |pop ebx  
0044C6F7       |. |F7FB          |idiv ebx  
0044C6F9       |. |85D2          |test edx,edx  
0044C6FB       |. |74 05         |je short winimage.0044C702  
0044C6FD       |. |8D3476        |lea esi,dword ptr ds:[esi+esi*2]  
0044C700       |. |EB 03         |jmp short winimage.0044C705  
0044C702       |> |6BF6 07       |imul esi,esi,7  
0044C705       |> |41            |inc ecx  
0044C706       |. |3B4D F8       |cmp ecx,dword ptr ss:[ebp-8]  
0044C709       |.^ \7C CC        \jl short winimage.0044C6D7  
```

关键在于箭头 1 和箭头 2 的地方，箭头 1 处是把用户名中的每一个字母移动到寄存器，和 `ESI` 的值做带符号乘法，再把结果和内存中的一个值相加，经查，该处内存在 `Stack ss:[0012E9E4]` 处，其值为 `0x0047694C`，而且这个值是固定的，是常量！如此循环，就在该内存处，得到了由用户名生成的第一个注册号，我叫他原始序列号。所以这是注册的初始阶段。

注册号生成以后，他只是保存在内存中的十六进制数，程序又把他转化为字符串，今妙之处就在这里，转换过程中，把 `0x0B` 和 `0x08` 的值对换了一下，即原来为 `0x0B` 的地方换成了 `0x08`，原来为 `0x08` 的地方换成了 `0x0B`，这可能是程序作者刻意的吧。这难不倒我，在写注册机的时候，我精心写了一个函数，就完成这项工作。主要部分如下：

```c
for ( i=0; i<8; i++, copy>>=4, clear<<=4 )
{
    simple = (char)(copy&0x000f);
    if ( simple==0x0B )
    {
        flag = 0x88888888 & clear;
        clear ^= 0xffffffff;
        SerialNumber &= clear;
        SerialNumber |= flag;   
        clear ^= 0xffffffff;        
    }
    else if ( simple==0x08 )
    {
        flag = 0xBBBBBBBB & clear;
        clear ^= 0xffffffff;
        SerialNumber &= clear;
        SerialNumber |= flag;
        clear ^= 0xffffffff;     
    }
    else;
}
```

用位操作，完成了数值的互换，得意一下！^\_^而原来地程序是这样的：

```
0044C713          /$  55              push ebp  
0044C714          |.  8BEC            mov ebp,esp  
0044C716          |.  83EC 10         sub esp,10  
0044C719          |.  56              push esi                              ;  |  THE RESULT JUST NOW  
0044C71A          |.  FF75 0C         push dword ptr ss:[ebp+C]             ; /<%lX>  
0044C71D          |.  8B75 08         mov esi,dword ptr ss:[ebp+8]          ; |  
0044C720          |.  8D45 F0         lea eax,dword ptr ss:[ebp-10]         ; |  
0044C723          |.  68 14DB4700     push winimage.0047DB14                ; |Format = "%lX"  
0044C728          |.  50              push eax                              ; |s  
0044C729          |.  FF15 14354700   call dword ptr ds:[<&USER32.wsprintfA>]; \wsprintfA  
0044C72F          |.  8A45 F0         mov al,byte ptr ss:[ebp-10]  
0044C732          |.  83C4 0C         add esp,0C  
0044C735          |.  84C0            test al,al  
0044C737          |.  74 1D           je short winimage.0044C756  
0044C739          |.  8D4D F0         lea ecx,dword ptr ss:[ebp-10]  
0044C73C          |.  2BCE            sub ecx,esi  
0044C73E         >  3C 38            /cmp al,38  
0044C740          |.  75 04          |jnz short winimage.0044C746  
0044C742          |.  04 0A          |add al,0A  
0044C744          |.  EB 06          |jmp short winimage.0044C74C  
0044C746         >  3C 42          |cmp al,42  
0044C748          |.  75 02          |jnz short winimage.0044C74C  
0044C74A          |.  04 F6          |add al,0F6  
0044C74C         >  8806          |mov byte ptr ds:[esi],al  
0044C74E          |.  46             |inc esi  
0044C74F          |.  8A0431        |mov al,byte ptr ds:[ecx+esi]  
0044C752          |.  84C0          |test al,al  
0044C754          |.^ 75 E8         \jnz short winimage.0044C73E  
0044C756         >  8B45 08         mov eax,dword ptr ss:[ebp+8]  
0044C759          |.  C606 00        mov byte ptr ds:[esi],0  
0044C75C          |.  5E             pop esi  
0044C75D          |.  C9             leave  
0044C75E          \  C3             ret  
```

然后就开始序列号的比对，具体代码如下：
