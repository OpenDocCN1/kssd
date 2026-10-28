
```asm
0047484F    FFB5 D4000000   push    dword ptr [ebp+D4]               ; 父窗口句柄
00474855    68 F4010000     push    1F4
0047485A    FF35 084F4700   push    dword ptr [474F08]
00474860    FFD0            call    eax                              ; 调用DialogBoxParamA
00474862    61              popad                                    ; 调用完毕恢复现场
00474863    FF75 10         push    dword ptr [ebp+10]               ; 执行被覆盖的代码
00474866    8B07            mov     eax, [edi]
00474868  - E9 F99FFDFF     jmp     0044E866                         ; 回到挂钩的下一句
0047486D    55              push    ebp                              ; 窗口过程
0047486E    8BEC            mov     ebp, esp
00474870    8B45 0C         mov     eax, [ebp+C]
00474873    3D 10010000     cmp     eax, 110                         ; WM\_INITDIALOG分支，在这里获取一些重要API的地址
00474878    0F85 9F000000   jnz     0047491D
0047487E    68 28404700     push    00474028                         ; ASCII "GetDlgItemInt"
00474883    FF35 044F4700   push    dword ptr [474F04]
00474889    FF15 80914500   call    [<&KERNEL32.GetProcAddress>]     ; kernel32.GetProcAddress
0047488F    A3 0C4F4700     mov     [474F0C], eax
00474894    68 36404700     push    00474036                         ; ASCII "SetDlgItemInt"
00474899    FF35 044F4700   push    dword ptr [474F04]
0047489F    FF15 80914500   call    [<&KERNEL32.GetProcAddress>]     ; kernel32.GetProcAddress
004748A5    A3 104F4700     mov     [474F10], eax
004748AA    6A 00           push    0
004748AC    FF35 09C34100   push    dword ptr [41C309]               ; 将SHA-1常数读出并依次显示到5个对话框中
004748B2    68 F5010000     push    1F5
004748B7    FF75 08         push    dword ptr [ebp+8]
004748BA    FF15 104F4700   call    [474F10]                         ; SetDlgItemInt
004748C0    6A 00           push    0
004748C2    FF35 10C34100   push    dword ptr [41C310]
004748C8    68 F6010000     push    1F6
004748CD    FF75 08         push    dword ptr [ebp+8]
004748D0    FF15 104F4700   call    [474F10]
004748D6    6A 00           push    0
004748D8    FF35 17C34100   push    dword ptr [41C317]
004748DE    68 F7010000     push    1F7
004748E3    FF75 08         push    dword ptr [ebp+8]
004748E6    FF15 104F4700   call    [474F10]
004748EC    6A 00           push    0
004748EE    FF35 1EC34100   push    dword ptr [41C31E]
004748F4    68 F8010000     push    1F8
004748F9    FF75 08         push    dword ptr [ebp+8]
004748FC    FF15 104F4700   call    [474F10]
00474902    6A 00           push    0
00474904    FF35 25C34100   push    dword ptr [41C325]
0047490A    68 F9010000     push    1F9
0047490F    FF75 08         push    dword ptr [ebp+8]
00474912    FF15 104F4700   call    [474F10]
00474918    E9 35010000     jmp     00474A52
0047491D    3D 11010000     cmp     eax, 111                         ; WM\_COMMAND分支
00474922    0F85 18010000   jnz     00474A40
00474928    0FB745 10       movzx   eax, word ptr [ebp+10]
0047492C    3D 08020000     cmp     eax, 208                         ; 按下Change按钮
00474931    0F85 83000000   jnz     004749BA
00474937    6A 00           push    0
00474939    6A 00           push    0
0047493B    68 F5010000     push    1F5
00474940    FF75 08         push    dword ptr [ebp+8]
00474943    FF15 0C4F4700   call    [474F0C]                         ; GetDlgItemInt
00474949    A3 09C34100     mov     [41C309], eax                    ; 依次取得文本框中的常数并写入到程序中
0047494E    6A 00           push    0
00474950    6A 00           push    0
00474952    68 F6010000     push    1F6
00474957    FF75 08         push    dword ptr [ebp+8]
0047495A    FF15 0C4F4700   call    [474F0C]
00474960    A3 10C34100     mov     [41C310], eax
00474965    6A 00           push    0
00474967    6A 00           push    0
00474969    68 F7010000     push    1F7
0047496E    FF75 08         push    dword ptr [ebp+8]
00474971    FF15 0C4F4700   call    [474F0C]
00474977    A3 17C34100     mov     [41C317], eax
0047497C    6A 00           push    0
0047497E    6A 00           push    0
00474980    68 F8010000     push    1F8
00474985    FF75 08         push    dword ptr [ebp+8]
00474988    FF15 0C4F4700   call    [474F0C]
0047498E    A3 1EC34100     mov     [41C31E], eax
00474993    6A 00           push    0
00474995    6A 00           push    0
00474997    68 F9010000     push    1F9
0047499C    FF75 08         push    dword ptr [ebp+8]
0047499F    FF15 0C4F4700   call    [474F0C]
004749A5    A3 25C34100     mov     [41C325], eax
004749AA    6A 00           push    0
004749AC    FF75 08         push    dword ptr [ebp+8]
004749AF    FF15 20944500   call    [<&USER32.EndDialog>]            ; USER32.EndDialog
004749B5    E9 9A000000     jmp     00474A54
004749BA    3D 09020000     cmp     eax, 209                         ; 按下Use Default按钮
004749BF    75 6B           jnz     short 00474A2C
004749C1    6A 00           push    0
004749C3    68 01234567     push    67452301
004749C8    68 F5010000     push    1F5
004749CD    FF75 08         push    dword ptr [ebp+8]
004749D0    FF15 104F4700   call    [474F10]                         ; 将文本框中的数值设置为SHA-1的默认值
004749D6    6A 00           push    0
004749D8    68 89ABCDEF     push    EFCDAB89
004749DD    68 F6010000     push    1F6
004749E2    FF75 08         push    dword ptr [ebp+8]
004749E5    FF15 104F4700   call    [474F10]
004749EB    6A 00           push    0
004749ED    68 FEDCBA98     push    98BADCFE
004749F2    68 F7010000     push    1F7
004749F7    FF75 08         push    dword ptr [ebp+8]
004749FA    FF15 104F4700   call    [474F10]
00474A00    6A 00           push    0
00474A02    68 76543210     push    10325476
00474A07    68 F8010000     push    1F8
00474A0C    FF75 08         push    dword ptr [ebp+8]
00474A0F    FF15 104F4700   call    [474F10]
00474A15    6A 00           push    0
00474A17    68 F0E1D2C3     push    C3D2E1F0
00474A1C    68 F9010000     push    1F9
00474A21    FF75 08         push    dword ptr [ebp+8]
00474A24    FF15 104F4700   call    [474F10]
00474A2A    EB 26           jmp     short 00474A52
00474A2C    3D 0A020000     cmp     eax, 20A                         ; 按下Cancel按钮
00474A31    75 1F           jnz     short 00474A52
00474A33    6A 00           push    0
00474A35    FF75 08         push    dword ptr [ebp+8]
00474A38    FF15 20944500   call    [<&USER32.EndDialog>]            ; USER32.EndDialog
00474A3E    EB 14           jmp     short 00474A54
00474A40    83F8 10         cmp     eax, 10                          ; WM\_CLOSE分支
00474A43    75 0D           jnz     short 00474A52
00474A45    6A 00           push    0
00474A47    FF75 08         push    dword ptr [ebp+8]
00474A4A    FF15 20944500   call    [<&USER32.EndDialog>]            ; USER32.EndDialog
00474A50    EB 02           jmp     short 00474A54
00474A52    33C0            xor     eax, eax
00474A54    C9              leave
00474A55    C2 1000         retn    10
```

====================以上是代码==================

写完这些代码后，将改动复制到可执行文件中就行了。

需要说明的几处地方：

I)　`DialogBoxParam`需要的两个参数`hInstance`和`hWndOwner`，程序中都无法即时提供，前者还比较好办，只要用参数`NULL`再调用`GetModuleHandle`就可以了，但后者就有些麻烦了，实在想不到有哪个API可以即时获取，在实践中基于一种思想：主窗口句柄需要用做参数来建立各种控件子窗口，因此它必定会在堆栈的某个地方出现，在OD的“窗口”查看栏中查看到句柄的值后，再在堆栈中搜索这个值，结果才发现在`[ebp+0D4]`的位置保存着一份主窗口句柄。

II)　为了更改SHA-1常数，必须知道这些常数在代码段的什么位置，换言之就是需要知道往什么地方写入数据，用OD中“查找常量”功能查找特征常量如`0xC3D2E1F0`查到的位置有3处，怎么办呢，这时PEid又发挥作用了，用分析密码学算法插件查到SHA-1算法在地址`41CE56`处，于是判断正确的位置是在这个地址往上的一处。当然这需要验证。验证的方法也很简单，新程序完工后运行一份原程序对同一个字符串算Hash，其他方法算出来的值都一样，只有SHA-1不同，就证明我们的操作是正确的。

III)　获取`user32.dll`等库的句柄需要提供文件名，获取函数地址需要提供函数名，总之就是需要提供一些字符串，这些字符串也写在新建的区里，实践中是写到`474000`开始的地方（新区段的起点），到了写入文件的时候，这些改动也要写进去。

IV)　调用导入表里的函数方式是间接调用，机器码是`FF15`，不要错写成直接调用的`E8`，譬如在Stud\_PE里查到导入表中`EndDialog`的RVA是`459420`那么指令应该是`call [459420]`，实际上很好理解，导入表中怎么可能包含可执行代码呢。（VC生成的程序好象没有跳板，直接`call`向导入表中）

V)　原始汇编方式有个难点就是确定跳转的距离，如果跳转的位置无法预计，最好都先汇编成长跳转留下5个字节的空间，即使以后改短跳转，空出来的字节填些`nop`就行，总比先汇编短跳转后发现必须改长跳转好。
