
* Reference to: `user32.MessageBoxA()`

|
```
00443BF5   E8762AFCFF             call    00406670        //× 准备在此处 ×结束× VMProtect的保护 ×//详见【6B】
00443BFA   EB08                   jmp     00443C04  //这个JMP命令的机器码 EB08 便是
00443BFC   FD                     std               //PESpin的 Crypt markers 的 SDK {$I crypt_end.inc}
00443BFD   FD                     std               //的开始标志
00443BFE   FD                     std               //
00443BFF   FD                     std               //
00443C00   FD                     std               //
00443C01   FD                     std               //
00443C02   54                     push    esp       //PESpin的 Crypt markers 的 SDK {$I crypt_end.inc}
00443C03   37                     aaa               //的结束标志
00443C04   5B                     pop     ebx
00443C05   C3                     ret
```
。。。

【6A】。点击“`Project`”菜单下的“`New procedure`”或者是点击工具栏中的“`New procedure`”按钮，在弹出的添加地址窗口中输入 `00443BE1` 后，点击“`OK`”按钮，VMProtect便又会自动列出虚拟地址：`00443BE1` 后面的代码；

【6B】。在操作了【6A】之后，VMProtect左边窗口中便多了一个以 `00443BE1` 这处为开始地址，可以被VMProtect保护的这段相关代码有哪些。点击这行后又发现，在列出的可被保护代码中，虚拟地址：`00443BFA` 这一行的代码并不想被VMProtect保护；因此，在右边窗口列出的可被保护代码中，找到并选定：“`00443BFA   EB08                   jmp     00443C04`” 这一行，单击鼠标右键，在弹出的菜单中选择：“`End of procedure`”或是 按：`Ctrl+B` 键；这时虚拟地址：`00443BFA` 以及它后面的代码都变成了灰色； 表示这些灰色代码不再被VMProtect保护；

因此，`TForm1.Image1Click` 事件被分成了三块，其中，中间将要被PeSpin的SDK所保护的这一块，先被VMProtect保护了。

※ 7。然后在VMProtect的“`Options`”窗口中设置相应的选项，最后点击工具栏中的“`Compilation (F9)`”按钮，便可！

※ 8。再用PESpin打开被VMProtect保护了的eXe文件，对该eXe文件进行加密，即可！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
