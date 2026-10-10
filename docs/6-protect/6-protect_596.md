

# 【VMProtect】配合【PESpin 的 SDK 加密】的另一方法 

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-558.htm`](https://www.pediy.com/kssd/pediy08/pediy8-558.htm)

【VMProtect】配合【PESpin 的 SDK 加密】的另一方法  
  
Anskya 使用 VMProtect 的 SDK 方式配合 ASProtect 的 SDK 实施保护；我便采用 VMProtec 的另一方式  
（在工程编译选项中:Project->Options->Linker 选项卡中的 Map File 选项中，将单选框选择 Detailed，然后在 VMProtect 添加保护代码中  
会看到函数地址）配合 PESpin 的 SDK 实施保护。  
  
目标：  
    使用【VMProtect】配合【PESpin 的 SDK 加密】对一个使用 delphi 编译的例子实施保护！不知自己理解的对否，还请各位提提意见！  
  
过程：  
  
※  1。使用 delphi 打开 Project1.DPR 文件，然后点击 delphi 的 Project 菜单下的 Options...选项，在弹出的对话窗口中点击 Linker 页面，在该页面中选择 Map file 下的 Detailed 这一项，以便打开同时生成完全 MAP 文件的这一功能，因为在 VMProtect 中要使用 MAP 文件与对应的 eXe 文件做比较，以方便 VMProtect 对其进行加密。  
  
※  2。完成上面的操作后，便按 F9 键进行编译，在目录下便生成了 eXe 及 Map 文件；使用 DeDe 对该 eXe 文件进行反汇编。  
  
※  3。用 VMProtect 打开生成的这个 eXe 文件后，点击“Project”菜单下的“New procedure”或者是 点击工具栏中的“New procedure”按钮，在弹出的添加地址窗口中  
找到：TForm1.Button1Click 这一行，然后点击“OK”按钮，  
找到：TForm1.FormCreate   这一行，然后点击“OK”按钮，  
找到：TForm1.Image1Click  这一行，然后点击“OK”按钮，  
在 VMProtect 左边的窗口中便会列出以这三处为开始地址，可以被 VMProtect 保护的这三段相关代码有哪些。  
  
  
  
  
  
※  4。在 VMProtect 中，点击左边窗口中的 TForm1.FormCreate 这一行后，配合 DeDe 对 eXe 文件的反汇编，看到其相应的代码和虚拟地址为：  
。。。  
00443A04   EB24                   jmp     00443A2A       //这个 JMP 命令的机器码 EB24 便是  
00443A06   FB                     sti                    //PESpin 的 Crypt markers 的 SDK {$I clear\_start.inc}  
00443A07   FB                     sti                    //的开始标志  
00443A08   FB                     sti                    //  
00443A09   FB                     sti                    //  
00443A0A   FB                     sti                    //  
00443A0B   FB                     sti                    //  
00443A0C   FB                     sti                    //  
00443A0D   FB                     sti                    //  
00443A0E   FB                     sti                    //  
00443A0F   FB                     sti                    //  
00443A10   FB                     sti                    //  
00443A11   FB                     sti                    //  
00443A12   FB                     sti                    //  
00443A13   FB                     sti                    //  
00443A14   FB                     sti                    //  
00443A15   FB                     sti                    //  
00443A16   FB                     sti                    //  
00443A17   FB                     sti                    //  
00443A18   FB                     sti                    //  
00443A19   FB                     sti                    //  
00443A1A   FB                     sti                    //  
00443A1B   FB                     sti                    //  
00443A1C   FB                     sti                    //  
00443A1D   FB                     sti                    //  
00443A1E   FB                     sti                    //  
00443A1F   FB                     sti                    //  
00443A20   FB                     sti                    //  
00443A21   FB                     sti                    //  
00443A22   FB                     sti                    //  
00443A23   FB                     sti                    //  
00443A24   FB                     sti                    //  
00443A25   FB                     sti                    //PESpin 的 Crypt markers 的 SDK {$I clear\_start.inc}  
00443A26   66BD0000               mov     bp, $0000      //的结束标志  
  
\* Possible String Reference to: 'PESpin Markers Test'  
|  
00443A2A   BA743A4400             mov     edx, $00443A74       //× 准备在此处 ×开始× VMProtect 的保护 ×//详见【4A】  
  
\* Reference to: controls.TControl.SetText(TControl;TCaption);  
|  
00443A2F   E89C02FEFF             call    00423CD0             //× 准备在此处 ×结束× VMProtect 的保护 ×//详见【4B】  
00443A34   EB33                   jmp     00443A69       //这个 JMP 命令的机器码 EB33 便是  
00443A36   FA                     cli                    //PESpin 的 Crypt markers 的 SDK {$I clear\_end.inc}  
00443A37   FA                     cli                    //的开始标志  
00443A38   FA                     cli                    //  
00443A39   FA                     cli                    //  
00443A3A   FA                     cli                    //  
00443A3B   FA                     cli                    //  
00443A3C   FA                     cli                    //  
00443A3D   FA                     cli                    //  
00443A3E   FA                     cli                    //  
00443A3F   FA                     cli                    //  
00443A40   FA                     cli                    //  
00443A41   FA                     cli                    //  
00443A42   FA                     cli                    //  
00443A43   FA                     cli                    //  
00443A44   FA                     cli                    //  
00443A45   FA                     cli                    //  
00443A46   FA                     cli                    //  
00443A47   FA                     cli                    //  
00443A48   FA                     cli                    //  
00443A49   FA                     cli                    //  
00443A4A   FA                     cli                    //  
00443A4B   FA                     cli                    //  
00443A4C   FA                     cli                    //  
00443A4D   FA                     cli                    //  
00443A4E   FA                     cli                    //  
00443A4F   FA                     cli                    //  
00443A50   FA                     cli                    //  
00443A51   FA                     cli                    //  
00443A52   FA                     cli                    //  
00443A53   FA                     cli                    //  
00443A54   FA                     cli                    //  
00443A55   FA                     cli                    //  
00443A56   FA                     cli                    //  
00443A57   FA                     cli                    //  
00443A58   FA                     cli                    //  
00443A59   FA                     cli                    //  
00443A5A   FA                     cli                    //  
00443A5B   FA                     cli                    //  
00443A5C   FA                     cli                    //  
00443A5D   FA                     cli                    //  
00443A5E   FA                     cli                    //  
00443A5F   FA                     cli                    //  
00443A60   FA                     cli                    //  
00443A61   FA                     cli                    //  
00443A62   FA                     cli                    //  
00443A63   FA                     cli                    //  
00443A64   FA                     cli                    //PESpin 的 Crypt markers 的 SDK {$I clear\_end.inc}  
00443A65   66BD0000               mov     bp, $0000      //的结束标志  
00443A69   C3                     ret  
。。。  
    【4A】。点击“Project”菜单下的“New procedure”或者是点击工具栏中的“New procedure”按钮，在弹出的添加地址窗口中输入 00443A2A 后，点击“OK”按钮，VMProtect 便又会自动列出虚拟地址：00443A2A 后面的代码；  
  
    【4B】。在操作了【4A】之后，VMProtect 左边窗口中便多了一个以 00443A2A 这处为开始地址，可以被 VMProtect 保护的这段相关代码有哪些。点击这行后又发现，在列出的可被保护代码中，虚拟地址：00443A34 这一行的代码并不想被 VMProtect 保护；因此，在右边窗口列出的可被保护代码中，找到并选定：“00443A34   EB33                   jmp     00443A69” 这一行，单击鼠标右键，在弹出的菜单中选择：“End of procedure”或是 按：Ctrl+B 键； 这时虚拟地址：00443A34 以及它后面的代码都变成了灰色； 表示这些灰色代码不再被 VMProtect 保护；  
  
因此，TForm1.FormCreate 事件被分成了三块，其中，中间将要被 PeSpin 的 SDK 所保护的这一块，先被 VMProtect 保护了。  
  
  
  
  
  
※  5。在 VMProtect 中点击左边窗口中的 TForm1.Button1Click 这一行后，配合 DeDe 对 eXe 文件的反汇编，看到其相应的代码和虚拟地址为：  
。。。  
00443A88   55                     push    ebp  
00443A89   8BEC                   mov     ebp, esp  
00443A8B   6A00                   push    $00  
00443A8D   6A00                   push    $00  
00443A8F   6A00                   push    $00  
00443A91   53                     push    ebx  
00443A92   56                     push    esi  
00443A93   57                     push    edi  
00443A94   8BF8                   mov     edi, eax  
00443A96   33C0                   xor     eax, eax  
00443A98   55                     push    ebp  
  
\* Possible String Reference to: '殪鼷�腚\_^\[嬪\]?  
|  
00443A99   68873B4400             push    $00443B87  
  
\*\*\*\*\* TRY  
|  
00443A9E   64FF30                 push    dword ptr fs:\[eax\]  
00443AA1   648920                 mov     fs:\[eax\], esp  
00443AA4   8D55FC                 lea     edx, \[ebp\-$04\]  
00443AA7   8B87D4020000           mov     eax, \[edi+$02D4\]  
  
\* Reference to: controls.TControl.GetText(TControl):TCaption;  
|  
00443AAD   E8EE01FEFF             call    00423CA0  
00443AB2   8B45FC                 mov     eax, \[ebp\-$04\]  
  
\* Reference to: system.@LStrLen:Integer;  
|           or: system.@DynArrayLength;  
|           or: system.DynArraySize(Pointer):Integer;  
|  
00443AB5   E8A600FCFF             call    00403B60  
00443ABA   85C0                   test    eax, eax  
00443ABC   0F84AA000000           jz      00443B6C  
00443AC2   EB08                   jmp     00443ACC  //这个 JMP 命令的机器码 EB08 便是  
00443AC4   FC                     cld               //PESpin 的 Crypt markers 的 SDK {$I crypt\_start.inc}   
00443AC5   FC                     cld               //的开始标志  
00443AC6   FC                     cld               //  
00443AC7   FC                     cld               //  
00443AC8   FC                     cld               //  
00443AC9   FC                     cld               //  
00443ACA   27                     daa               //PESpin 的 Crypt markers 的 SDK {$I crypt\_start.inc}   
00443ACB   54                     push    esp       //的结束标志  
00443ACC   33DB                   xor     ebx, ebx        //× 准备在此处 ×开始× VMProtect 的保护 ×//详见【5A】  
00443ACE   33F6                   xor     esi, esi  
00443AD0   43                     inc     ebx  
00443AD1   8D55F8                 lea     edx, \[ebp\-$08\]  
00443AD4   8B87D4020000           mov     eax, \[edi+$02D4\]  
  
\* Reference to: controls.TControl.GetText(TControl):TCaption;  
|  
00443ADA   E8C101FEFF             call    00423CA0  
00443ADF   8B45F8                 mov     eax, \[ebp\-$08\]  
00443AE2   0FB64418FF             movzx   eax, byte ptr \[eax+ebx\-$01\]  
00443AE7   03F0                   add     esi, eax  
00443AE9   8D55F4                 lea     edx, \[ebp\-$0C\]  
00443AEC   8B87D4020000           mov     eax, \[edi+$02D4\]  
  
\* Reference to: controls.TControl.GetText(TControl):TCaption;  
|  
00443AF2   E8A901FEFF             call    00423CA0  
00443AF7   8B45F4                 mov     eax, \[ebp\-$0C\]  
  
\* Reference to: system.@LStrLen:Integer;  
|           or: system.@DynArrayLength;  
|           or: system.DynArraySize(Pointer):Integer;  
|  
00443AFA   E86100FCFF             call    00403B60  
00443AFF   3BD8                   cmp     ebx, eax  
00443B01   7CCD                   jl      00443AD0  
00443B03   83EE43                 sub     esi, +$43  
00443B06   83C610                 add     esi, +$10  
00443B09   81F69A020000           xor     esi, $0000029A  
00443B0F   81F609030000           xor     esi, $00000309  
00443B15   C1E602                 shl     esi, $02  
00443B18   33DB                   xor     ebx, ebx  
00443B1A   43                     inc     ebx  
00443B1B   8BC3                   mov     eax, ebx  
00443B1D   03C0                   add     eax, eax  
00443B1F   33C3                   xor     eax, ebx  
00443B21   F7EE                   imul    esi   
00443B23   8BF0                   mov     esi, eax  
00443B25   83FB06                 cmp     ebx, +$06  
00443B28   7CF0                   jl      00443B1A        //× 准备在此处 ×结束× VMProtect 的保护 ×//详见【5B】  
00443B2A   EB08                   jmp     00443B34  //这个 JMP 命令的机器码 EB08 便是  
00443B2C   FD                     std               //PESpin 的 Crypt markers 的 SDK {$I crypt\_end.inc}  
00443B2D   FD                     std               //的开始标志  
00443B2E   FD                     std               //  
00443B2F   FD                     std               //  
00443B30   FD                     std               //  
00443B31   FD                     std               //  
00443B32   54                     push    esp       //PESpin 的 Crypt markers 的 SDK {$I crypt\_end.inc}  
00443B33   37                     aaa               //的结束标志  
00443B34   81FEC0884904           cmp     esi, $044988C0  
00443B3A   7408                   jz      00443B44  
00443B3C   81FEC0884904           cmp     esi, $044988C0  
00443B42   7515                   jnz     00443B59  
00443B44   6A00                   push    $00  
  
\* Possible String Reference to: 'Example Markers'  
|  
00443B46   68983B4400             push    $00443B98  
  
\* Possible String Reference to: 'Good pass :D'  
|  
00443B4B   68A83B4400             push    $00443BA8  
00443B50   6A00                   push    $00  
  
\* Reference to: user32.MessageBoxA()  
|  
00443B52   E8192BFCFF             call    00406670  
00443B57   EB13                   jmp     00443B6C  
00443B59   6A00                   push    $00  
  
\* Possible String Reference to: 'Example Markers'  
|  
00443B5B   68983B4400             push    $00443B98  
  
\* Possible String Reference to: 'Bad password. Try: PESpin'  
|  
00443B60   68B83B4400             push    $00443BB8  
00443B65   6A00                   push    $00  
  
\* Reference to: user32.MessageBoxA()  
|  
00443B67   E8042BFCFF             call    00406670  
00443B6C   33C0                   xor     eax, eax  
00443B6E   5A                     pop     edx  
00443B6F   59                     pop     ecx  
00443B70   59                     pop     ecx  
00443B71   648910                 mov     fs:\[eax\], edx  
  
\*\*\*\*\*\* FINALLY  
|  
  
\* Possible String Reference to: '\_^\[嬪\]?  
|  
00443B74   688E3B4400             push    $00443B8E  
00443B79   8D45F4                 lea     eax, \[ebp\-$0C\]  
00443B7C   BA03000000             mov     edx, $00000003  
  
\* Reference to: system.@LStrArrayClr;  
|  
00443B81   E87EFDFBFF             call    00403904  
00443B86   C3                     ret  
  
  
\* Reference to: system.@HandleFinally;  
|  
00443B87   E9ECF7FBFF             jmp     00403378  
00443B8C   EBEB                   jmp     00443B79  
  
\*\*\*\*\*\* END  
|  
00443B8E   5F                     pop     edi  
00443B8F   5E                     pop     esi  
00443B90   5B                     pop     ebx  
00443B91   8BE5                   mov     esp, ebp  
00443B93   5D                     pop     ebp  
00443B94   C3                     ret  
。。。  
    【5A】。点击“Project”菜单下的“New procedure”或者是点击工具栏中的“New procedure”按钮，在弹出的添加地址窗口中输入 00443ACC 后，点击“OK”按钮，VMProtect 便又会自动列出虚拟地址：00443ACC 后面的代码；  
  
    【5B】。在操作了【5A】之后，VMProtect 左边窗口中便多了一个以 00443ACC 这处为开始地址，可以被 VMProtect 保护的这段相关代码有哪些。点击这行后又发现，在列出的可被保护代码中，虚拟地址：00443B2A 这一行及后面的代码并不想被 VMProtect 保护；因此，在右边窗口列出的可被保护代码中，找到并选定：“00443B2A EB08                    jmp 00443B34” 这一行，单击鼠标右键，在弹出的菜单中选择：“End of procedure”或是 按：Ctrl+B 键； 这时虚拟地址：00443B2A 以及它后面的代码都变成了灰色； 表示这些灰色代码不再被 VMProtect 保护；  
  
因此，TForm1.FormCreate 事件被分成了三块，其中，中间将要被 PeSpin 的 SDK 所保护的这一块，先被 VMProtect 保护了。  
  
  
  
  
  
※  6。在 VMProtect 中点击左边窗口中的 TForm1.Image1Click 这一行后，配合 DeDe 对 eXe 文件的反汇编，看到其相应的代码和虚拟地址为：  
。。。  
00443BD4   53                     push    ebx  
00443BD5   8BD8                   mov     ebx, eax  
00443BD7   EB08                   jmp     00443BE1  //这个 JMP 命令的机器码 EB08 便是  
00443BD9   FC                     cld               //PESpin 的 Crypt markers 的 SDK {$I crypt\_start.inc}   
00443BDA   FC                     cld               //的开始标志  
00443BDB   FC                     cld               //  
00443BDC   FC                     cld               //  
00443BDD   FC                     cld               //  
00443BDE   FC                     cld               //  
00443BDF   27                     daa               //PESpin 的 Crypt markers 的 SDK {$I crypt\_start.inc}  
00443BE0   54                     push    esp       //的结束标志  
00443BE1   6A00                   push    $00             //× 准备在此处 ×开始× VMProtect 的保护 ×//详见【6A】  
  
\* Possible String Reference to: 'Example Markers...'  
|  
00443BE3   68083C4400             push    $00443C08  
  
\* Possible String Reference to: 'Example in Delphi by reywen^htb'  
|  
00443BE8   681C3C4400             push    $00443C1C  
00443BED   8BC3                   mov     eax, ebx  
  
\* Reference to: controls.TWinControl.GetHandle(TWinControl):HWND;  
|  
00443BEF   E87C61FEFF             call    00429D70  
00443BF4   50                     push    eax  
  
\* Reference to: user32.MessageBoxA()  
|  
00443BF5   E8762AFCFF             call    00406670        //× 准备在此处 ×结束× VMProtect 的保护 ×//详见【6B】  
00443BFA   EB08                   jmp     00443C04  //这个 JMP 命令的机器码 EB08 便是  
00443BFC   FD                     std               //PESpin 的 Crypt markers 的 SDK {$I crypt\_end.inc}  
00443BFD   FD                     std               //的开始标志  
00443BFE   FD                     std               //  
00443BFF   FD                     std               //  
00443C00   FD                     std               //  
00443C01   FD                     std               //  
00443C02   54                     push    esp       //PESpin 的 Crypt markers 的 SDK {$I crypt\_end.inc}  
00443C03   37                     aaa               //的结束标志  
00443C04   5B                     pop     ebx  
00443C05   C3                     ret  
。。。  
    【6A】。点击“Project”菜单下的“New procedure”或者是点击工具栏中的“New procedure”按钮，在弹出的添加地址窗口中输入 00443BE1 后，点击“OK”按钮，VMProtect 便又会自动列出虚拟地址：00443BE1 后面的代码；  
  
    【6B】。在操作了【6A】之后，VMProtect 左边窗口中便多了一个以 00443BE1 这处为开始地址，可以被 VMProtect 保护的这段相关代码有哪些。点击这行后又发现，在列出的可被保护代码中，虚拟地址：00443BFA 这一行的代码并不想被 VMProtect 保护；因此，在右边窗口列出的可被保护代码中，找到并选定：“00443BFA   EB08                   jmp     00443C04” 这一行，单击鼠标右键，在弹出的菜单中选择：“End of procedure”或是 按：Ctrl+B 键；这时虚拟地址：00443BFA 以及它后面的代码都变成了灰色； 表示这些灰色代码不再被 VMProtect 保护；  
  
因此，TForm1.Image1Click 事件被分成了三块，其中，中间将要被 PeSpin 的 SDK 所保护的这一块，先被 VMProtect 保护了。  
  
  
  
  
  
※  7。然后在 VMProtect 的“Options”窗口中设置相应的选项，最后点击工具栏中的“Compilation (F9)”按钮，便可！  
  
  
※  8。再用 PESpin 打开被 VMProtect 保护了的 eXe 文件，对该 eXe 文件进行加密，即可！ 

另：声明一下，上面所提到的使用 DeDe 只是便于对代码反汇编后的理解和好看而用到，不用 DeDe 也没有关系的！只要你有 Map 文件，对 VMP 来说就够了！  
上面所用到的 eXe 文件，是在 PESpin v1.304 自身所带的例子，你可在 PESpin1304 的 Examples\\delphi 目录中找到它！