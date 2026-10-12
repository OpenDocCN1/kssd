
# 让酷狗音乐双开

> 原文：[`www.pediy.com/kssd/pediy11/107091.html`](https://www.pediy.com/kssd/pediy11/107091.html)

对于听音乐来说，酷狗这个软件第一选择，前几天买了一个耳机，很贵哦，据说为了让耳机达到最好的效果再使用前要煲机，可是一首一首播放太浪费时间了，现在 `cpu` 都多核心了，为什么酷狗不能呢？好，现在让我们用逆向来突破这个限制。

下面看代码

```
00726CC3    E8 6013CEFF      call    00408028  
00726CC8    8B15 FCB67400   mov     edx, dword ptr [74B6FC]          ; KuGoo.0074A500  
00726CCE    8902            mov     dword ptr [edx], eax  
00726CD0    6A 64           push    64  
00726CD2    A1 7CAC7400     mov     eax, dword ptr [74AC7C]  
00726CD7    8B00            mov     eax, dword ptr [eax]  
00726CD9    50              push    eax  
00726CDA    E8 D917CEFF     call    <jmp.&kernel32.WaitForSingleObje>  
00726CDF    3D 02010000     cmp     eax, 102  
00726CE4    0F85 25010000   jnz     00726E0F                         ; 没运行就跳  
00726CEA    68 10270000     push    2710  
00726CEF    A1 78AF7400     mov     eax, dword ptr [74AF78]  
00726CF4    8B00            mov     eax, dword ptr [eax]  
00726CF6    50              push    eax  
00726CF7    E8 BC17CEFF     call    <jmp.&kernel32.WaitForSingleObje>  
00726CFC    3D 02010000     cmp     eax, 102  
00726D01    74 0D           je      short 00726D10  
00726D03    A1 78AF7400     mov     eax, dword ptr [74AF78]  
00726D08    8B00            mov     eax, dword ptr [eax]  
00726D0A    50              push    eax  
00726D0B    E8 C016CEFF     call    <jmp.&kernel32.ReleaseMutex>  
00726D10    6A 00           push    0  
00726D12    A1 FCB67400     mov     eax, dword ptr [74B6FC]  
00726D17    8B00            mov     eax, dword ptr [eax]  
00726D19    50              push    eax  
00726D1A    E8 9917CEFF     call    <jmp.&kernel32.WaitForSingleObje>  
00726D1F    3D 02010000     cmp     eax, 102  
00726D24    75 3F           jnz     short 00726D65                   ; 打开原来程序的运行的窗口  
00726D26    68 88130000     push    1388  
00726D2B    A1 FCB67400     mov     eax, dword ptr [74B6FC]  
00726D30    8B00            mov     eax, dword ptr [eax]  
00726D32    50              push    eax  
00726D33    E8 8017CEFF     call    <jmp.&kernel32.WaitForSingleObje>  
00726D38    3D 02010000     cmp     eax, 102  
00726D3D    75 05           jnz     short 00726D44  
00726D3F    E8 00B9FFFF     call    00722644  
00726D44    6A FF           push    -1  
00726D46    A1 FCB67400     mov     eax, dword ptr [74B6FC]  
00726D4B    8B00            mov     eax, dword ptr [eax]  
00726D4D    50              push    eax  
00726D4E    E8 6517CEFF     call    <jmp.&kernel32.WaitForSingleObje>  
00726D53    A1 FCB67400     mov     eax, dword ptr [74B6FC]  
00726D58    8B00            mov     eax, dword ptr [eax]  
00726D5A    50              push    eax  
00726D5B    E8 7016CEFF     call    <jmp.&kernel32.ReleaseMutex>  
00726D60  ^ E9 6BFFFFFF     jmp     00726CD0  
00726D65    A1 FCB67400     mov     eax, dword ptr [74B6FC]  
00726D6A    8B00            mov     eax, dword ptr [eax]  
00726D6C    50              push    eax  
00726D6D    E8 5E16CEFF     call    <jmp.&kernel32.ReleaseMutex>  
00726D72    66:BB EC13      mov     bx, 13EC  
```

`00726CE4` `0F85 25010000` `jnz` `00726E0F`   很明显，这个就是对比之处，           那是不是吧 `jnz` 改成 `jmp` 就可以呢?

恩，我们来试试，可是实际告诉我们，这样是不行的，因为程序后面还有校验。

看来这个是一个陷阱哈   

那怎么办？我们再仔细看，

```
00726CD0    6A 64           push    64  
00726CD2    A1 7CAC7400     mov     eax, dword ptr [74AC7C]  
00726CD7    8B00            mov     eax, dword ptr [eax]  
00726CD9    50              push    eax  
00726CDA    E8 D917CEFF     call    <jmp.&kernel32.WaitForSingleObje>   检测是否运行  
00726CDF    3D 02010000     cmp     eax, 102  
00726CE4    0F85 25010000   jnz     00726E0F                         ; 没运行就跳  
00726CEA    68 10270000     push    2710  
```

呵呵 ，这一次我们 `nop` 掉 `00726CDA` 函数  然后保存，

恩 运行一下，可以了 。。![](img/962324edb48d57effae0c0571850a66b.jpg "酷") 

这次可以同时用酷狗同时播放好几个音乐了，如果你的大脑是多线程的。哈哈。
