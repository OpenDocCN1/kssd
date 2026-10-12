
重新载入程序，在`DUMP`区`01006450`处的4个字节上下内存访问断点（不能下硬件断点!）。

`F9`，断下：

```
0028321C    8DB5 6D050000     lea esi,dword ptr ss:\[ebp+56D\]  
00283222    8B07              mov eax,dword ptr ds:\[edi\]////取地址放入 EAX=01006455。此时 EDI=284100。  
00283224    0BC0              or eax,eax  
00283226    75 02             jnz short 0028322A  
00283228    EB 1D             jmp short 00283247  
0028322A    25 FFFFFF7F        and eax,7FFFFFFF  
0028322F    0385 83060000     add eax,dword ptr ss:\[ebp+683\]  
00283235    2B85 8F060000     sub eax,dword ptr ss:\[ebp+68F\]  
0028323B    8BDE              mov ebx,esi  
0028323D    2BD8              sub ebx,eax  
0028323F    8958 FC            mov dword ptr ds:\[eax-4\],ebx////断在这里。  
00283242    83C7 08            add edi,8  
00283245  ^ EB DB              jmp short 00283222////跳回继续。  
00283247    64:FF35 30000000   push dword ptr fs:\[30\]  
```

断下时`EBX=FF27CE80``EAX=01006455``EDI=284100`。显然程序正准备将加密后的数据移入该地址。看来这里就是关键处了!

到`DUMP`区`284100`处看看：

```
00284100  55 64 00 01 54 63 A0 EF  Ud.Tc\_?
```

还记得前面的问题吗－`EAX`和`EDI`的数据是哪里来的？看到`DUMP`区该知道了吧:)

下面来`PATH`：

重新载入程序，然后`bp VirtualAlloc`，在`01010237`处断下2次后，返回，`CTRL+G`来到：

```
0028322A    25 FFFFFF7F        and eax,7FFFFFFF////从此处开始 PATH。  
0028322F    0385 83060000     add eax,dword ptr ss:\[ebp+683\]  
00283235    2B85 8F060000     sub eax,dword ptr ss:\[ebp+68F\]  
0028323B    8BDE              mov ebx,esi  
0028323D    2BD8              sub ebx,eax  
0028323F    8958 FC            mov dword ptr ds:\[eax-4\],ebx  
00283242    83C7 08            add edi,8  
00283245  ^ EB DB              jmp short 00283222  
00283247    64:FF35 30000000   push dword ptr fs:\[30\]  
0028324E    58                  pop eax  
```

修改为：

```
0028322A   /E9 D11B0000        jmp 00284E00  
```

下面是我写的憋脚`Path`代码：

```
00284E00       807F 03 01          cmp byte ptr ds:\[edi+3\],1////比较最高位是否为 1。原因不再解释。  
00284E04       75 1C               jnz short 00284E22  
00284E06       53                  push ebx///借用一下:)  
00284E07       8B47 04             mov eax,dword ptr ds:\[edi+4\]////加密数据之一移入 EAX。  
00284E0A       2B07                sub eax,dword ptr ds:\[edi\]///相当于 0028335E 处的 sub eax,edi。  
00284E0C       F7D0                not eax///取反。  
00284E0E       C1C0 10             rol eax,10///循环左移。  
00284E11       8B1F                mov ebx,dword ptr ds:\[edi\]///调用地址入 EBX。  
00284E13       66:C743 FA FF15     mov word ptr ds:\[ebx-6\],15FF///加入指令 CALL\[XXXXXXXX\]  
00284E19       8943 FC             mov dword ptr ds:\[ebx-4\],eax///正确的跳转地址移入正确的调用地址。  
00284E1C       5B                  pop ebx///归还。  
00284E1D     ^ E9 20E4FFFF         jmp 00283242////跳回继续。  

00284E22       53                  push ebx///借用一下:)  
00284E23       8B1F                mov ebx,dword ptr ds:\[edi\]///先要处理一下最高位。  
00284E25       81E3 FFFFFFFF       and ebx,7FFFFFFF////取消最高位。  
00284E2B       891F                mov dword ptr ds:\[edi\],ebx  
00284E2D       8B47 04             mov eax,dword ptr ds:\[edi+4\]  
00284E30       2B07                sub eax,dword ptr ds:\[edi\]  
00284E32       F7D0                not eax  
00284E34       C1C0 10             rol eax,10  
00284E37       8B1F                mov ebx,dword ptr ds:\[edi\]  
00284E39       66:C743 FA FF25     mov word ptr ds:\[ebx-6\],25FF////加入指令 JMP\[XXXXXXXX\]  
00284E3F       8943 FC             mov dword ptr ds:\[ebx-4\],eax  
00284E42       5B                  pop ebx  
00284E43     ^ E9 FAE3FFFF         jmp 00283242////跳回继续。  
```

二进制数据：

```
80 7F 03 01 75 1C 53 8B 47 04 2B 07 F7 D0 C1 C0 10 8B 1F 66 C7 43 FA FF 15 89 43 FC 5B E9 20 E4  
FF FF 53 8B 1F 81 E3 FF FF FF 7F 89 1F 8B 47 04 2B 07 F7 D0 C1 C0 10 8B 1F 66 C7 43 FA FF 25 89  
43 FC 5B E9 FA E3 FF FF  
```

按上述方法`PATH`后就可以到入口修复入口代码并`DUMP`了，然后用`ImportREC`修复就`OK`了。如果将`DUMP`文件的最后一个区段删除再修复`IAT`将会失去图标，但可以正常运行。

### 三、查找被偷代码

重新载入后在壳的入口处：

```
01010000 >  56                 push esi  
01010001    52                 push edx  
01010002    51                 push ecx  
01010003    53                 push ebx  
01010004    55                 push ebp  
01010005    E8 15010000        call NOTEPAD.0101011F  
0101000A    3200               xor al,byte ptr ds:\[eax\]  
```

此时观察寄存器数据，`ESP=0006FFC4`。

重复上面的步骤，在`01010237`处断下2次后，`F8`返回后，根据堆栈平衡原理，在`0006FFC0`处的四个字节下硬件访问双字节断点。然后`F9`运行。

```
002832B5    55                 push ebp///断下!被偷代码第一行。  
002832B6    8BEC               mov ebp,esp///被偷代码第二行。  
002832B8    EB 19              jmp short 002832D3  
002832BA    83FB 02            cmp ebx,2  
002832BD    75 0F              jnz short 002832CE  
002832BF    8BC5               mov eax,ebp  
002832C1    5D                 pop ebp  
002832C2    5B                 pop ebx  
002832C3    59                 pop ecx  
002832C4    5A                 pop edx  
```

找到被偷代码后就可以恢复了。

`4dd466d0e1e0cc123afbe504da2462ca`
