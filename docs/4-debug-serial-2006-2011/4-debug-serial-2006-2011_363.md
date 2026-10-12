
# 处理系统 5.7 注册算法分析+注册机源代码

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-188.htm`](https://www.pediy.com/kssd/pediy09/pediy09-188.htm)
>
> 【文章标题】: 友锋图像处理系统 5.7 注册算法分析+注册机源代码
>
> 【文章作者】: `bxm`
>
> 【作者邮箱】: `bxm78@163.com`
>
> 【软件名称】: 友锋图像处理系统
>
> 【下载地址】: 自己搜索下载
>
> 【编写语言】: `Borland Delphi 6.0 - 7.0`
>
> 【使用工具】: `OD`, `winhex`, 计算器
>
> 【操作平台】: `winxp_sp2`

> --------------------------------------------------------------------------------
>
> 【详细过程】
>
> 应一个朋友之要求破解这个软件，顺便把破文发出来，有错误之处，请各位批评指正。
>
> 拿到软件，试运行，发现有两种注册方式，一是通过网络下载注册码，二是从文件中读取注册码。决定从文件验证入手。
>
> 注册文件名不固定，扩展名为 `.dat`。用记事本建了个 `bxm.dat`，输入 `1234567890` 十个字符，然后读入注册文件，确定以后发现没什么反应，好像假死一样，决定用 `OD` 调试。
>
> 既然是文件验证，就下个断点 `bp ReadFile`，断下后，`Alt+F9` 返回程序领空。

```
00403276  |.  6A 00         push    0                                ; /pOverlapped = NULL
00403278  |.  50           push    eax                              ; |pBytesRead
00403279  |.  FF73 08      push    dword ptr [ebx+8]                ; |BytesToRead
0040327C  |.  52           push    edx                              ; |Buffer
0040327D  |.  FF33         push    dword ptr [ebx]                  ; |hFile
0040327F  |.  E8 B0E0FFFF  call    <jmp.&kernel32.ReadFile>        ; \ReadFile
00403284  |.  5A           pop     edx
00403285  |.  48           dec     eax
00403286  |.  75 13        jnz     short 0040329B                   ;  如果文件不存在，跳。
00403288  |.  3B53 08      cmp     edx, dword ptr [ebx+8]           ;  文件大小与 0x328 比较
0040328B  |.  74 18        je      short 004032A5                   ;  相等，跳
0040328D  |.  B8 64000000  mov     eax, 64
00403292  |.  EB 0C        jmp     short 004032A0
00403294  |>  B8 67000000  mov     eax, 67
00403299  |.  EB 05        jmp     short 004032A0
0040329B  |>  E8 14E1FFFF  call    <jmp.&kernel32.GetLastError>     ; [GetLastError
004032A0  |>  E8 17F7FFFF  call    004029BC
004032A5  |>  5B           pop     ebx
004032A6  \.  C3           retn
```

> 由上可知，文件必须为 `0x328` 字节。重新建个 `0x328` 大小的文件，`004032A6` 返回到如下地址。

```
005D3B22  |.  E8 75EEE2FF  call    0040299C
005D3B27  |.  8D85 B4FEFFFF lea     eax, dword ptr [ebp-14C]
005D3B2D  |.  E8 4AF5E2FF  call    0040307C
005D3B32  |.  E8 65EEE2FF  call    0040299C
005D3B37  |.  66:81BD B0FEF>cmp     word ptr [ebp-150], 300          ;  检查注册文件的第 805、806 字节是否为 0x0003
005D3B40  |.  0F84 99000000 je      005D3BDF                         ;  如果相等，说明版本正确，否则弹出版本不符对话框
```

> 强行跳过此处。

```
005D3BDF  |> \8D85 68FBFFFF lea     eax, dword ptr [ebp-498]
005D3BE5  |.  8D95 90FBFFFF lea     edx, dword ptr [ebp-470]         ;  注册文件的第 5 个字节地址入 EDX
005D3BEB  |.  E8 4810E3FF  call    00404C38                           ;  根据注册文件内容截取一些数据作为注册用户名，比较复杂，懒得跟了
005D3BF0  |.  8B85 68FBFFFF mov     eax, dword ptr [ebp-498]
005D3BF6  |.  8D95 6CFBFFFF lea     edx, dword ptr [ebp-494]
005D3BFC  |.  E8 7BF4FFFF  call    005D307C                           ;  加密用户名，懒得跟了
005D3C01  |.  8B8D 6CFBFFFF mov     ecx, dword ptr [ebp-494]
005D3C07  |.  BA 483E5D00  mov     edx, 005D3E48                      ;  value
005D3C0C  |.  B8 583E5D00  mov     eax, 005D3E58                      ;  user
005D3C11  |.  E8 5E92F7FF  call    0054CE74                           ;  在 HKEY_CURRENT_USER\Software\友锋图像处理系统\User 下写入 Value 的值为加密后用户名
005D3C16  |.  8B8D 8CFBFFFF mov     ecx, dword ptr [ebp-474]
005D3C1C  |.  BA 683E5D00  mov     edx, 005D3E68                      ;  order
005D3C21  |.  B8 783E5D00  mov     eax, 005D3E78                      ;  ASCII "Application"
005D3C26  |.  E8 6D93F7FF  call    0054CF98                           ;  在 HKEY_CURRENT_USER\Software\友锋图像处理系统\Application 下写入 Order 的值为注册文件的第 1 至 4 字节所转换成的字符串
005D3C2B  |.  8D85 60FBFFFF lea     eax, dword ptr [ebp-4A0]
005D3C31  |.  8D95 90FBFFFF lea     edx, dword ptr [ebp-470]
005D3C37  |.  E8 FC0FE3FF  call    00404C38
005D3C3C  |.  8B85 60FBFFFF mov     eax, dword ptr [ebp-4A0]
005D3C42  |.  8D95 64FBFFFF lea     edx, dword ptr [ebp-49C]
005D3C48  |.  E8 2FF4FFFF  call    005D307C
005D3C4D  |.  8B8D 64FBFFFF mov     ecx, dword ptr [ebp-49C]
005D3C53  |.  BA 8C3E5D00  mov     edx, 005D3E8C                      ;  ASCII "Hint"
005D3C58  |.  B8 783E5D00  mov     eax, 005D3E78                      ;  ASCII "Application"
005D3C5D  |.  E8 1292F7FF  call    0054CE74                           ;  写入 Hint 的值为加密后的用户名
005D3C62  |.  33C9           xor     ecx, ecx
005D3C64  |.  8A8D AFFBFFFF mov     cl, byte ptr [ebp-451]           ;  注册文件的第 36 字节入 cl
005D3C6A  |.  BA 9C3E5D00  mov     edx, 005D3E9C                      ;  ASCII "Setx"
005D3C6F  |.  B8 783E5D00  mov     eax, 005D3E78                      ;  ASCII "Application"
005D3C74  |.  E8 1F93F7FF  call    0054CF98                           ;  写入 Setx 的值为 cl 所转换成的十进制字符串
005D3C79  |.  B2 01          mov     dl, 1
005D3C7B  |.  A1 8C1C4400   mov     eax, dword ptr [441C8C]
005D3C80  |.  E8 07E1E6FF   call    00441D8C
005D3C85  |.  8BD8           mov     ebx, eax
005D3C87  |.  BA 01000080   mov     edx, 80000001
005D3C8C  |.  8BC3           mov     eax, ebx
005D3C8E  |.  E8 99E1E6FF   call    00441E2C
005D3C93  |.  B1 01          mov     cl, 1
005D3C95  |.  BA AC3E5D00   mov     edx, 005D3EAC
005D3C9A  |.  8BC3           mov     eax, ebx
005D3C9C  |.  E8 CBE2E6FF   call    00441F6C
005D3CA1  |.  68 FF000000   push    0FF                              ; /Arg1 = 000000FF
005D3CA6  |.  8D8D B0FBFFFF lea     ecx, dword ptr [ebp-450]         ; |
005D3CAC  |.  BA DC3E5D00   mov     edx, 005D3EDC                    ; |ASCII "Set1"
005D3CB1  |.  8BC3           mov     eax, ebx                         ; |
005D3CB3  |.  E8 28E5E6FF   call    004421E0                         ; \写入 Set1 的值为注册文件的第 37 字节以后的 255 字节
005D3CB8  |.  68 FF000000   push    0FF                              ; /Arg1 = 000000FF
005D3CBD  |.  8D8D B0FCFFFF lea     ecx, dword ptr [ebp-350]         ; |
005D3CC3  |.  BA EC3E5D00   mov     edx, 005D3EEC                    ; |ASCII "Set2"
005D3CC8  |.  8BC3           mov     eax, ebx                         ; |
005D3CCA  |.  E8 11E5E6FF   call    004421E0                         ; \写入 Set2 的值为注册文件的第 292 字节以后的 255 字节
005D3CCF  |.  68 FF000000   push    0FF                              ; /Arg1 = 000000FF
005D3CD4  |.  8D8D B0FDFFFF lea     ecx, dword ptr [ebp-250]         ; |
005D3CDA  |.  BA FC3E5D00   mov     edx, 005D3EFC                    ; |ASCII "Set3"
005D3CDF  |.  8BC3           mov     eax, ebx                         ; |
005D3CE1  |.  E8 FAE4E6FF   call    004421E0                         ; \写入 Set3 的值为注册文件的第 37 字节以后的 545 字节
005D3CE6  |.  8BC3           mov     eax, ebx
005D3CE8  |.  E8 0FE1E6FF   call    00441DFC
005D3CED  |.  B1 01          mov     cl, 1
005D3CEF  |.  BA 0C3F5D00   mov     edx, 005D3F0C
005D3CF4  |.  8BC3           mov     eax, ebx
005D3CF6  |.  E8 71E2E6FF   call    00441F6C
005D3CFB  |.  8D85 5CFBFFFF lea     eax, dword ptr [ebp-4A4]
005D3D01  |.  8D95 90FBFFFF lea     edx, dword ptr [ebp-470]
005D3D07  |.  E8 2C0FE3FF  call    00404C38
005D3D0C  |.  8B8D 5CFBFFFF mov     ecx, dword ptr [ebp-4A4]
005D3D12  |.  BA 583E5D00   mov     edx, 005D3E58                    ;  ASCII "User"
005D3D17  |.  8BC3           mov     eax, ebx
005D3D19  |.  E8 0AE4E6FF   call    00442128
005D3D1E  |.  BA 2C3F5D00   mov     edx, 005D3F2C                    ;  ASCII "State"
```



```asm
005D3D23  |.  8BC3             mov     eax, ebx
005D3D25  |.  E8 0AE6E6FF      call    00442334
005D3D2A  |.  84C0             test    al, al
005D3D2C  |.  74 0C            je      short 005D3D3A
005D3D2E  |.  BA 2C3F5D00      mov     edx, 005D3F2C                     ;  ASCII "State"
005D3D33  |.  8BC3             mov     eax, ebx
005D3D35  |.  E8 56E3E6FF      call    00442090
005D3D3A  |>  8BC3             mov     eax, ebx
005D3D3C  |.  E8 BBE0E6FF      call    00441DFC
005D3D41  |.  8BC3             mov     eax, ebx
005D3D43  |.  E8 78FEE2FF      call    00403BC0
005D3D48  |.  6A 40            push    40
005D3D4A  |.  8D95 58FBFFFF    lea     edx, dword ptr [ebp-4A8]
005D3D50  |.  A1 D4066B00      mov     eax, dword ptr [6B06D4]
005D3D55  |.  8B00             mov     eax, dword ptr [eax]
005D3D57  |.  E8 A430ECFF      call    00496E00
005D3D5C  |.  8B85 58FBFFFF    mov     eax, dword ptr [ebp-4A8]
005D3D62  |.  E8 2D11E3FF      call    00404E94
005D3D67  |.  50               push    eax
005D3D68  |.  68 343F5D00      push    005D3F34
005D3D6D  |.  8BC6             mov     eax, esi
005D3D6F  |.  E8 447FEAFF      call    0047BCB8
005D3D74  |.  50               push    eax                               ; |hOwner
005D3D75  |.  E8 FE3CE3FF      call    <jmp.&user32.MessageBoxA>         ; \\弹出重启验证对话框
```

重启后，下断点 `bpx regopenkeyExA`，断下后一步步找到关键处。

```asm
00649E62   .  C645 FB 9E      mov     byte ptr [ebp-5], 9E               ;  [ebp-5]置初值 0x9E
00649E66   .  8B45 F0         mov     eax, dword ptr [ebp-10]            ;  Order 入 EAX
00649E69   .  E8 26AEDBFF     call    00404C94                           ;  EAX 返回 Order 的长度
00649E6E   .  85C0            test    eax, eax
00649E70   .  7E 13           jle     short 00649E85
00649E72   .  BB 01000000     mov     ebx, 1
00649E77   >  8B55 F0         mov     edx, dword ptr [ebp-10]
00649E7A   .  8A541A FF       mov     dl, byte ptr [edx+ebx-1]          ;  dl 为 Order 的每一位
00649E7E   .  3055 FB        xor     byte ptr [ebp-5], dl               ;  dl 异或[ebp-5],[ebp-5]的初始值为 0x9E
00649E81   .  43             inc     ebx
00649E82   .  48             dec     eax
00649E83   .^ 75 F2          jnz     short 00649E77                     ;  最终[ebp-5]得 A6(我的结果)
```

此段代码相当于以下 C 程序：

```c
for(i=0;i<10;i++)  //temp[0]为[ebp-5]
  temp[0]^=Order[i];
```

```asm
00649E85   > \\C645 FA 9E    mov     byte ptr [ebp-6], 9E
00649E89   .  8B45 F4        mov     eax, dword ptr [ebp-C]             ;  注册用户名入 EAX
00649E8C   .  E8 03AEDBFF    call    00404C94                           ;  EAX 返回注册用户名的长度
00649E91   .  85C0           test    eax, eax
00649E93   .  7E 13          jle     short 00649EA8
00649E95   .  BB 01000000    mov     ebx, 1
00649E9A   >  8B55 F4        mov     edx, dword ptr [ebp-C]
00649E9D   .  8A541A FF      mov     dl, byte ptr [edx+ebx-1]          ;  用户名的每一位入 dl
00649EA1   .  3055 FA       xor     byte ptr [ebp-6], dl               ;  dl 异或[ebp-6],[ebp-6]的初始值为 0x9E
00649EA4   .  43             inc     ebx
00649EA5   .  48             dec     eax
00649EA6   .^ 75 F2          jnz     short 00649E9A                     ;  最终[ebp-6]得 A8
```

此段代码相当于以下 C 程序：

```c
for(i=0;i<54;i++)  //temp[1]为[ebp-6]，name 为用户名
  temp[1]^=name[i];
```

```asm
00649EA8   > \\8D45 F4       lea     eax, dword ptr [ebp-C]
00649EAB   .  E8 24ABDBFF    call    004049D4
00649EB0   .  BB 01000000    mov     ebx, 1                             ;  ebx 置初值 1
00649EB5   .  8DBD E0FEFFFF  lea     edi, dword ptr [ebp-120]
00649EBB   .  8DB5 E0FCFFFF  lea     esi, dword ptr [ebp-320]           ;  set3 入 ESI
00649EC1   .  8D85 E0FDFFFF  lea     eax, dword ptr [ebp-220]           ;  set2 入 EAX
00649EC7   .  8945 E0        mov     dword ptr [ebp-20], eax
00649ECA   >  8BC3           mov     eax, ebx
00649ECC   .  B9 03000000    mov     ecx, 3
00649ED1   .  99             cdq
00649ED2   .  F7F9           idiv    ecx
00649ED4   .  83EA 01        sub     edx, 1                             ;  Switch (cases 0..2)
00649ED7   .  72 07          jb      short 00649EE0
00649ED9   .  74 0E          je      short 00649EE9
00649EDB   .  4A             dec     edx
00649EDC   .  74 19          je      short 00649EF7
00649EDE   .  EB 1F          jmp     short 00649EFF
00649EE0   >  8A07           mov     al, byte ptr [edi]                 ;  set1 的第 n 位(循环次数位)入 al; Case 0 of switch 00649ED4
00649EE2   .  3245 FB       xor     al, byte ptr [ebp-5]               ;  al 异或[ebp-5]
00649EE5   .  3006           xor     byte ptr [esi], al                 ;  set3 的第 n 位异或 al
00649EE7   .  EB 16          jmp     short 00649EFF
00649EE9   >  8A06           mov     al, byte ptr [esi]                 ;  set3 的第 1 位入 al; Case 1 of switch 00649ED4
00649EEB   .  3245 FA       xor     al, byte ptr [ebp-6]               ;  al 与[ebp-6]异或
00649EEE   .  8B55 E0        mov     edx, dword ptr [ebp-20]
00649EF1   .  3202           xor     al, byte ptr [edx]                 ;  al 与 set2 的第 1 位异或
00649EF3   .  8806           mov     byte ptr [esi], al                 ;  结果保存在 set3 的第 1 位
00649EF5   .  EB 08          jmp     short 00649EFF
00649EF7   >  8A45 FB       mov     al, byte ptr [ebp-5]               ;  [ebp-5]入 al; Case 2 of switch 00649ED4
00649EFA   .  3245 FA       xor     al, byte ptr [ebp-6]               ;  al 异或[ebp-6]
00649EFD   .  3006           xor     byte ptr [esi], al                 ;  set3 异或 al
00649EFF   >  8D85 D8FCFFFF  lea     eax, dword ptr [ebp-328]           ;  Default case of switch 00649ED4
00649F05   .  8A16           mov     dl, byte ptr [esi]                 ;  set3 的第 n 位(循环次数位）入 dl
00649F07   .  E8 B0ACDBFF    call    00404BBC
00649F0C   .  8B95 D8FCFFFF  mov     edx, dword ptr [ebp-328]
00649F12   .  8D45 F4        lea     eax, dword ptr [ebp-C]
00649F15   .  E8 82ADDBFF    call    00404C9C                           ;  结果复制到新位置
00649F1A   .  43             inc     ebx
00649F1B   .  FF45 E0        inc     dword ptr [ebp-20]                 ;  set2+1
00649F1E   .  46             inc     esi                                  ;  set3+1
00649F1F   .  47             inc     edi                                  ;  set1+1
00649F20   .  81FB FB000000  cmp     ebx, 0FB
00649F26   .^ 75 A2          jnz     short 00649ECA
```

此段代码相当于以下 C 程序：

```c
for(i=0;i<30;i++)
  switch(i%3)
  {
    case 0:
      al=set3[i];
      al^=temp[1];
      al^=set2[i];
      set3[i]=al;
      break;
    case 1:
      al=temp[0];
      al^=temp[1];
      set3[i]^=al;
      break;
    case 2:
      al=set1[i];
      al^=temp[0];
      set3[i]^=al;
  }
```

至此，`set3` 中已存放了运算后的注册码，在下面的程序中将和机器码有多处比较，代码就不贴了。

附注册机的 C++ 代码，以 VC6 中编译通过。

```cpp
void CYouFeng_keygenDlg::OnButton1()
{
  // TODO: Add your control notification handler code here
  unsigned char Order[]="875770417";
  unsigned char temp[3]={0x9e,0x9e,0x9e};
  unsigned char name[54]={0x62, 0x78, 0x6D, 0x00, 0x30, 0x41, 0x42, 0x43, 0x44, 0x45,
                         0x46, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39,
                         0x30, 0x41, 0x42, 0x43, 0x44, 0x45, 0x46, 0x31, 0x32, 0x33,
                         0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x30, 0x41, 0x42, 0x43,
};
```



```
  0x44, 0x45, 0x46, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37,  
  0x38, 0x39, 0x30,0};  
  //name 为未加密的用户名信息  
  
  unsigned char user[108]="FCE6F39EAEDFDCDDDADBD8AFACADAAABA8A9A6A7AEDFDCDDDADBD8AFACADAAABA8A9A6A7AEDFDCDDDADBD8AFACADAAABA8A9A6A7AE";  
  //user 为加密后的用户名  
  
  unsigned char set1[31]="111111111111111111111111111111";  
  unsigned char set2[31]="222222222222222222222222222222";  
  unsigned int set3[31];  
  unsigned char set3_2[31];  
  char set4[31];     //set4 为机器码  
  
  UpdateData(true);  
  strcpy(set4,m_Edit);  
  int i;  
  for(i=0;i<10;i++)  //temp[0]为[ebp-5]  
    temp[0]^=Order[i];  
  
  for(i=0;i<54;i++)  //temp[1]为[ebp-6]  
    temp[1]^=name[i];  
  
  for(i=0;i<30;i++)  
    switch(i%3)  
    {  
      case 0:  
        set3[i]=set4[i]^temp[1]^set2[i];  
        break;  
      case 1:  
        set3[i]=temp[0]^temp[1]^set4[i];  
        break;  
      case 2:  
        set3[i]=temp[0]^set4[i]^set1[i];  
    }  
  
    for(i=0;i<31;i++)  
    {  
      set3_2[i]=set3[i];  
    }  
  
    HKEY   hk;    
    RegOpenKey(HKEY_CURRENT_USER,"Software\\友锋图像处理系统",&hk);  
    RegCreateKey(HKEY_CURRENT_USER,   "Software\\友锋图像处理系统\\Application",   &hk);  
    RegSetValueEx(hk,"Order",   0,     REG_SZ,   Order,   9);    
    RegSetValueEx(hk,"Set1",   0,     REG_BINARY,   set1,   30);    
    RegSetValueEx(hk,"Set2",   0,     REG_BINARY,   set2,   30);    
    RegSetValueEx(hk,"Hint",   0,     REG_SZ,   user,   107);    
    RegSetValueEx(hk,"Set3",   0,     REG_BINARY,   set3_2,   30);    
    RegCloseKey(hk);    
    
    RegOpenKey(HKEY_CURRENT_USER,"Software\\友锋图像处理系统",&hk);    
    RegCreateKey(HKEY_CURRENT_USER,   "Software\\友锋图像处理系统\\User",   &hk);  
    RegSetValueEx(hk,"Value",   0,     REG_SZ,   user,   107);    
    RegCloseKey(hk);    
    
    MessageBox("破解成功！","恭喜",MB_OK);  
    UpdateData(false);   
  }  
```

--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 06 月 21 日 23:44:20

`4dd466d0e1e0cc123afbe504da2462ca`
