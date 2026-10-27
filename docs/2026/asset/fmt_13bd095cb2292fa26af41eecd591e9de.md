
```asm
0063746A      8B45 14                    mov eax,dword ptr ss:[ebp+14]
0063746D      50                         push eax
0063746E      8B4D 10                    mov ecx,dword ptr ss:[ebp+10]
00637471      51                         push ecx
00637472      8B55 0C                    mov edx,dword ptr ss:[ebp+C]
00637475      52                         push edx
00637476      8B45 08                    mov eax,dword ptr ss:[ebp+8]
00637479      50                         push eax
0063747A      E8 0D000000                call vsgu2\_0.0063748C
0063747F      83C4 10                    add esp,10
00637482      8945 FC                    mov dword ptr ss:[ebp-4],eax
00637485      8B45 FC                    mov eax,dword ptr ss:[ebp-4]
00637488      8BE5                       mov esp,ebp
0063748A      5D                         pop ebp
0063748B      C3                         retn
```

而`call vsgu2\_0.0063748C`函数如下:

```asm
0063748C      55                         push ebp
0063748D      8BEC                       mov ebp,esp
0063748F      81EC AC090000              sub esp,9AC
00637495      57                         push edi
00637496      C685 C4F6FFFF 00           mov byte ptr ss:[ebp-93C],0
0063749D      B9 4D020000                mov ecx,24D
006374A2      33C0                       xor eax,eax
006374A4      8DBD C5F6FFFF              lea edi,dword ptr ss:[ebp-93B]
006374AA      F3:AB                      rep stos dword ptr es:[edi]
006374AC      66:AB                      stos word ptr es:[edi]
006374AE      C785 ACF6FFFF 00000000     mov dword ptr ss:[ebp-954],0
006374B8      8D85 C4F6FFFF              lea eax,dword ptr ss:[ebp-93C]
```

```asm
-----------00637427      E8 51B2FFFF                call vsgu2\_0.l\_zcp
-----------0063743E      E8 012AFEFF                call vsgu2\_0.l\_err\_info\_cp
-----------0063744A      E8 9B29FEFF                call vsgu2\_0.l\_free\_err\_info
--------0063E255      E8 E6960100                call vsgu2\_0.l\_supersede
--------0063E261      E8 5E880100                call vsgu2\_0.l\_post\_pkg
--------0063E26F      E8 CC960100                call vsgu2\_0.l\_supersede
--------0063E2E0      E8 2126FFFF                call vsgu2\_0.l\_lfclose关闭文件
----005C5E67      E8 24EDFFFF                call vsgu2\_0.vsgu::checkOutRTL
-----005C4BC0      E8 1BFFFFFF                call vsgu2\_0.vsgu::licenseIsAllowed
-----005C4C05      E8 D6FEFFFF                call vsgu2\_0.vsgu::licenseIsAllowed
-----005C4C1F      E8 CCFDFFFF                call vsgu2\_0.vsgu::checkOutLicense

  堆栈数据：0012EE3C    0012EE48     ASCII "vsg\_rtl"
            0012EE40    0069A030     ASCII "2.0"
            0012EE44    0012EEF0     ASCII "C:\Program Files\MultiGen-Paradigm\bin\vprun.exe"

------005C4A26      E8 D5C20400                call vsgu2\_0.CLicense::CLicense构造相应的Featrue和版本号的CLicense类
------005C4A35      E8 A6CD0400                call vsgu2\_0.CLicense::Checkout
-------00611808      E8 53EBFFFF                call vsgu2\_0.CLicenseManager::AttachLicense
```

下面这段代码我不能不先提一下了:

```asm
00611843      3BF8                       cmp edi,eax
00611845      7F 18                      jg short vsgu2\_0.0061185F  ;注意这里,关键跳转（1）
00611847      8B86 A4160000              mov eax,dword ptr ds:[esi+16A4]
0061184D      0FAFC7                     imul eax,edi
00611850      5F                         pop edi
00611851      5E                         pop esi
00611852      C1E0 02                    shl eax,2
00611855      5D                         pop ebp
00611856      81C4 54040000              add esp,454
0061185C      C2 0400                    retn 4
0061185F      57                         push edi
00611860      8BCE                       mov ecx,esi
```

程序在这里关键跳转（1）要跳了,所以我们暂时不管了.让它跳吧。

```asm
-------00611862      E8 F9FAFFFF                call vsgu2\_0.CLicense::DoDisableLicensingCallback
-------0061196E      E8 AD280000                call vsgu2\_0.MPITrim
-------00611993      E8 08F8FFFF                call vsgu2\_0.CLicense::Checkout注意此处与005C4A35 进入的地方不一样，这就是C++的特点吧，允许形参不同相同函数名称了。
--------006111CC      E8 EF660000                call vsgu2\_0.lc\_set\_attr设置FEATURE
--------006111ED      E8 7E500100                call vsgu2\_0.lc\_checkout
---------006262AC      E8 04880000                call vsgu2\_0.l\_mt\_lock
---------006262C0      E8 B11B0700                call vsgu2\_0.\_setjmp3
---------006262F3      E8 B3000000                call vsgu2\_0.l\_checkout

  堆栈数据：0012E944    09029AD8
            0012E948    0903BF1C     ASCII "vsg\_rtl"
            0012E94C    0903BEF4     ASCII "2.0"

----------006263E3      E8 95C20000                call vsgu2\_0.l\_zcp
----------006263F1      E8 7A6C0100                call vsgu2\_0.l\_getattr
----------00626520      E8 689D0100                call vsgu2\_0.l\_ckout\_borrow
-----------00640342      E8 BE35FFFF                call vsgu2\_0.l\_getenv
-----------00640353      E8 18C70000                call vsgu2\_0.l\_baddate
-----------0064047B      FF15 F4916900              call dword ptr ds:[<&MSVCR71.sprintf>]

  函数调用后的结果堆栈如下：

0012E664    0012E6DC     ASCII "borrow-d8f349a2-vsg\_rtl"
0012E668    006D868C     ASCII "borrow-%s-%s"
0012E66C    0012E71C     ASCII "d8f349a2"
0012E670    0903BF1C     ASCII "vsg\_rtl"

----------00626548      FF15 24366E00              call dword ptr ds:[6E3624]                   ; vsgu2\_0.00626AD5
-----------00626B4B      E8 33050100                call vsgu2\_0.l\_valid\_version
-----------00626BF9      E8 7FBA0000                call vsgu2\_0.l\_zcp
-----------00626C58      E8 2700FFFF                call vsgu2\_0.l\_next\_conf\_or\_marker
-----------00626CA7      E8 5C0A0000                call vsgu2\_0.l\_local\_verify\_conf
-----------00626E61      E8 89110000                call vsgu2\_0.l\_good\_lic\_key 小心了,如果是正确的license将返回1,错误的License返回0
------------00628096      E8 6B570100                call vsgu2\_0.l\_xorname
------------006280B3      E8 531D0000                call vsgu2\_0.l\_sg           ;第二次调用L\_SG
-------------00629E64      FF15 98396E00              call dword ptr ds:[l\_n36\_buff]               ; vsgu2\_0.00624600
-------------00628267      E8 E52B0000                call vsgu2\_0.0062AE51      ;小心了
--------------0062AE75      E8 38020000                call vsgu2\_0.0062B0B2     ;小心了
```

`F7`进去看一下,看到没有,发现是不是与前面介绍的`0063740C  call vsgu2\_0.l\_crypt\_private`代码基本一样啊！！！

```asm
0062B0B2      55                         push ebp
0062B0B3      8BEC                       mov ebp,esp
0062B0B5      81EC AC090000              sub esp,9AC
0062B0BB      57                         push edi
0062B0BC      C685 C4F6FFFF 00           mov byte ptr ss:[ebp-93C],0
0062B0C3      B9 4D020000                mov ecx,24D
0062B0C8      33C0                       xor eax,eax
0062B0CA      8DBD C5F6FFFF              lea edi,dword ptr ss:[ebp-93B]
0062B0D0      F3:AB                      rep stos dword ptr es:[edi]
0062B0D2      66:AB                      stos word ptr es:[edi]
0062B0D4      C785 ACF6FFFF 00000000     mov dword ptr ss:[ebp-954],0
0062B0DE      8D85 C4F6FFFF              lea eax,dword ptr ss:[ebp-93C]
```

呵呵,发现了吧,其实这里也是`l\_crypt\_private`传说中的检查license文件是否正确的地方了,但是如果你用"搜索-当前模块中的名称"的方法,是不能发现这处参考的.

由于这个函数太重要了,因此我将其关键代码列在此了:

```asm
---------------0062B3D9      E8 45ADFFFF                call vsgu2\_0.l\_good\_bin\_date
```

经过一系列对日期和序列号,Vendor Info的判断之后

然后关键比较出现了

```asm
---------------0062BC24      E8 46110000                call vsgu2\_0.0062CD6F

0062BBFB      8B95 C0F6FFFF              mov edx,dword ptr ss:[ebp-940]
0062BC01      52                         push edx                                     ; SN
0062BC02      8B85 A4F6FFFF              mov eax,dword ptr ss:[ebp-95C]
0062BC08      50                         push eax                                     ; 66D8B337
0062BC09      8B4D 14                    mov ecx,dword ptr ss:[ebp+14]
0062BC0C      51                         push ecx
```
