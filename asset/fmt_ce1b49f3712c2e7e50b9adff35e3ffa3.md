
```asm
00594DBA   .  8B45 FC       mov     eax, [ebp-4]

00594DBD   .  05 2F050000  add     eax, 52F

00594DC2 > .  E8 C5FEE6FF  call    00404C8C    -- ->System.@LStrAsg(void;void;void;void);

00594DC7   .  8B45 FC       mov     eax, [ebp-4]

00594DCA > .  8B80 2F050000 mov     eax, [eax+52F] -------->每一次启动计算出的 Dede,SHA1 散列值

00594DD0   .  8B15 34C85A00 mov     edx, [5AC834] ------->ini 文件中的 SHA1 Version 变量数值

00594DD6   .  8B12          mov     edx, [edx]

00594DD8 > .  E8 5702E7FF  call    00405034   ------>关键比较 System.@LStrCmp;

00594DDD   .  0F95C0        setne   al------>确定 Boolean 变量 al 的状态

00594DE0   .  8B55 FC       mov     edx, [ebp-4]

00594DE3 > .  8882 80050000 mov     [edx+580], al                  ;  *TDeDeMainForm.OFFS_0580:Byte

00594DE9   .  8B45 FC       mov     eax, [ebp-4]

00594DEC > .  80B8 80050000>cmp     byte ptr [eax+580], 0          ;  *TDeDeMainForm.OFFS_0580:Byte

00594DF3   .  74 3D         je      short 00594E32--------------->关键跳,跳就不产生 Nag,否相反!

00594DF5   .  8B0D ECF55A00 mov     ecx, [5AF5EC]

00594DFB   .  B2 01         mov     dl, 1

00594DFD   .  A1 2C204400   mov     eax, [44202C]----->TTimer 类,下面将产生 TTimer 对象

00594E02 > .  E8 6DFCEAFF  call    00444A74  ------------->ExtCtrls.TTimer.Create(TTimer;boolean;TComponent);

00594E07   .  8945 EC       mov     [ebp-14], eax

00594E0A   .  BA B80B0000   mov     edx, 0BB8------------>设置 0BB8 毫秒

00594E0F   .  8B45 EC       mov     eax, [ebp-14]

00594E12 > .  E8 F5FDEAFF  call    00444C0C  ----------->ExtCtrls.TTimer.SetInterval(TTimer;Cardinal);

00594E17   .  8B45 FC       mov     eax, [ebp-4]

00594E1A   .  50            push    eax

00594E1B   .  68 1C485900   push    0059481C ---------------->关键的 TTimer 回调事件,它是产生 Nag 方法句柄!

00594E20   .  8B45 EC       mov     eax, [ebp-14]

00594E23 > .  E8 F4FDEAFF  call    00444C1C  -------------->:TSplitter._PROC_00444C1C()

00594E28   .  B2 01         mov     dl, 1

00594E2A   .  8B45 EC       mov     eax, [ebp-14]

00594E2D > .  E8 CAFDEAFF  call    00444BFC                  ;  ->ExtCtrls.TTimer.SetEnabled(TTimer;Boolean);

00594E32   > .  8B45 FC       mov     eax, [ebp-4]

00594E35 > .  8B80 F0030000 mov     eax, [eax+3F0]             ;  *TDeDeMainForm.ClearTimer:TTimer

00594E3B   .  BA 88130000   mov     edx, 1388

00594E40 > .  E8 C7FDEAFF  call    00444C0C                  ;  ->ExtCtrls.TTimer.SetInterval(TTimer;Cardinal);

00594E45   .  8B45 FC       mov     eax, [ebp-4]

00594E48 > .  8B80 F0030000 mov     eax, [eax+3F0]             ;  *TDeDeMainForm.ClearTimer:TTimer

00594E4E   .  33D2          xor     edx, edx

00594E50 > .  E8 A7FDEAFF  call    00444BFC                  ;  ->ExtCtrls.TTimer.SetEnabled(TTimer;Boolean);

00594E55   .  C605 E8BF5A00>mov     byte ptr [5ABFE8], 0

00594E5C   .  B2 01         mov     dl, 1

00594E5E   .  A1 7CF64E00   mov     eax, [4EF67C]

00594E63 > .  E8 90EFE6FF  call    00403DF8                  ;  ->System.TObject.Create(TObject;Boolean);

00594E68   .  A3 F4F55A00   mov     [5AF5F4], eax

00594E6D   .  33C0          xor     eax, eax

00594E6F   .  5A            pop     edx

00594E70   .  59            pop     ecx

00594E71   .  59            pop     ecx

00594E72   .  64:8910       mov     fs:[eax], edx

00594E75   .  68 9A4E5900   push    00594E9A

00594E7A   > .  8D85 E8FDFFFF lea     eax, [ebp-218]

00594E80   .  BA 0D000000   mov     edx, 0D

00594E85 > .  E8 D2FDE6FF  call    00404C5C                  ;  ->System.@LStrArrayClr(void;void;Integer);

00594E8A   .  8D45 F8       lea     eax, [ebp-8]

00594E8D > .  E8 A6FDE6FF  call    00404C38                  ;  ->System.@LStrClr(void;void);

00594E92   .  C3            retn

00594E93  > ^.  E9 24F7E6FF jmp     004045BC                  ;  ->System.@HandleFinally;

00594E98   .^.  EB E0       jmp     short 00594E7A

00594E9A   .  8BE5          mov     esp, ebp

00594E9C   .  5D            pop     ebp

00594E9D   .  C3            retn
```

如果是重装了系统或者是改变了文件路径或者是文件名此时候 `Dede` 的 `Nag` 一定会再次询问 `License Statement` 声明，你同意与否。

在 `TDeDeMainForm@FormCreate` 事件中动态产生了 `TTimer` 组件，并且注册了 `TTimer` 对象的回调事件地址：`0059481C`。注意这是用 `Dede` 反编译自己得不出来的！

下面看：`TTimer` 对象的回调事件地址：`0059481C`，这个事件将会产生 `Nag` 窗体的程序。

```asm
0059481C  /.  55            push    ebp

0059481D  |.  8BEC          mov     ebp, esp

0059481F  |.  83C4 F0       add     esp, -10

00594822  |.  8955 F8       mov     [ebp-8], edx

00594825  |.  8945 FC       mov     [ebp-4], eax

00594828  |.  8B45 F8       mov     eax, [ebp-8]

0059482B  |.  8B15 2C204400 mov     edx, [44202C]  ------->TTimer

00594831  |.  E8 A2F7E6FF   call    00403FD8  ---->类检测 System.@AsClass(TObject;TClass):TObject;

00594836  |.  33D2          xor     edx, edx----->Boolean:=False;

00594838  |.  E8 BF03EBFF  call    00444BFC    ---->注销这个 Timer ExtCtrls.TTimer.SetEnabled(TTimer;Boolean);

0059483D  |.  33C9          xor     ecx, ecx

0059483F  |.  B2 01         mov     dl, 1

00594841  |.  A1 ECAA5500   mov     eax, [55AAEC]---->Nag 窗体的 TLSForm 类.下面产生 Nag 对象

00594846  |.  E8 F958EDFF   call    0046A144 ------->TCustomForm.Create(TCustomForm;boolean;TComponent);

0059484B  |.  8945 F4       mov     [ebp-C], eax

0059484E  |.  33C0          xor     eax, eax

00594850  |.  55            push    ebp

00594851  |.  68 89485900   push    00594889

00594856  |.  64:FF30       push    dword ptr fs:[eax]

00594859  |.  64:8920       mov     fs:[eax], esp

0059485C  |.  8B45 F4       mov     eax, [ebp-C]

0059485F  |.  8B10          mov     edx, [eax]

00594861  |.  FF92 E8000000 call    [edx+E8]  ------------->显示 Nag 窗体 TLSForm.ShowModal

00594867  |.  8B45 F4       mov     eax, [ebp-C]

0059486A  |.  8B80 4C020000 mov     eax, [eax+24C] ---->Nag 窗体的 TModalResult

00594870  |.  8945 F0       mov     [ebp-10], eax

00594873  |.  33C0          xor     eax, eax

00594875  |.  5A            pop     edx

00594876  |.  59            pop     ecx

00594877  |.  59            pop     ecx

00594878  |.  64:8910       mov     fs:[eax], edx

0059487B  |.  68 90485900   push    00594890

00594880  |>  8B45 F4       mov     eax, [ebp-C]

00594883  |.  E8 A0F5E6FF   call    00403E28                  ;  System.TObject.Free

00594888  \\.  C3            retn

00594889   .^.  E9 2EFDE6FF jmp     004045BC

0059488E   .^.  EB F0       jmp     short 00594880

00594890   .  837D F0 01    cmp     dword ptr [ebp-10], 1---------------->关键比较

00594894   .  75 1E         jnz     short 005948B4------->关键跳:如果 Nag 返回 ModalResult 不相等 1,就会结束程序的运行.

00594896   .  A1 34C85A00   mov     eax, [5AC834]------>关键全局变量,保存 Dede SHA1 散列值

0059489B   .  8B55 FC       mov     edx, [ebp-4]

0059489E   .  8B92 2F050000 mov     edx, [edx+52F]             ;  * Reference to: System.@LStrAsg(void;void;void;void);

005948A4   .  E8 E303E7FF   call    00404C8C

005948A9   .  B2 01         mov     dl, 1

005948AB   .  33C0          xor     eax, eax

005948AD   .  E8 6AE3F5FF   call    004F2C1C---------------->把新计算 Dede SHA1 string_[5AC834]散列值写回 ini 文件中.

005948B2   .  EB 0C         jmp     short 005948C0

005948B4   > A1 D0CA5A00   mov     eax, [5ACAD0]

005948B9   .  8B00         mov     eax, [eax]

005948BB   .  E8 E0D0EDFF  call    004719A0------>如果不同意其 Licence,就会结束主程序.调用 Forms.TApplication.Terminate(TApplication);

005948C0   > 8BE5          mov     esp, ebp

005948C2   .  5D            pop     ebp

005948C3   .  C3            retn
```

到这里重要关键的地方分析完毕，你可以动一下鼠标就 `game over` 了。剩下的 `Nag` 等窗体上的事件流程用 `dede` 比较容易分析出来。在这里不分析了留给大家。
