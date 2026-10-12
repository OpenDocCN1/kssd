
# 麻将拼图 V1.04

> 原文：[`www.pediy.com/kssd/pediy06/pediy6577.htm`](https://www.pediy.com/kssd/pediy06/pediy6577.htm)

教菜鸟写注册机——实战篇

呵呵，再灌一篇，等再过两天开学就没得灌喽![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

前面我们折腾了半天都是在跟 `CRACKME` 过不去，可能有人觉得没意思了，这次咱们来个有实际意义的，呵呵，一个发布不久的软件——麻将拼图 V1.04，这里来下：

[`skycn.softreg.com.cn/product...BE-9524013EFADA`](http://skycn.softreg.com.cn/product.asp?id=/E5DCF286-05EE-4AA8-8ABE-9524013EFADA)

用 `W32DASM` 反下，找串式参考“注册失败！”到下面：（分析见后）

> ```
> 代码:
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402E16(C)
> |
> :00402E3F 8B3D88974000             mov edi, dword ptr [00409788]
> :00402E45 B940000000               mov ecx, 00000040
> :00402E4A 33C0                     xor eax, eax
> :00402E4C 8B6C2414                 mov ebp, dword ptr [esp+14]
> :00402E50 F3                       repz
> :00402E51 AB                       stosd
> :00402E52 8B3D7C974000             mov edi, dword ptr [0040977C]
> :00402E58 B940000000               mov ecx, 00000040
> :00402E5D F3                       repz
> :00402E5E AB                       stosd
> :00402E5F 8B0D88974000             mov ecx, dword ptr [00409788]
> 
> \* Reference To: USER32.SendDlgItemMessageA, Ord:020Fh
>                                 |
> :00402E65 8B1D38714000             mov ebx, dword ptr [00407138]  ;注意
> :00402E6B 51                       push ecx     ;用户名存放地址[409788]
> :00402E6C 6A10                     push 00000010
> :00402E6E 6A0D                     push 0000000D     ;WM_GETTEXT
> 
> \* Possible Reference to Dialog: DialogID_0070, CONTROL_ID:03E8, ""
>                                 |
> :00402E70 68E8030000               push 000003E8     ;控件 ID
> :00402E75 55                       push ebp
> :00402E76 FFD3                     call ebx     ;得到用户名
> :00402E78 8B157C974000             mov edx, dword ptr [0040977C]
> :00402E7E 52                       push edx     ;注册码地址[40977C]
> :00402E7F 6A10                     push 00000010     
> :00402E81 6A0D                     push 0000000D     ;WM_GETTEXT
> 
> \* Possible Reference to Dialog: DialogID_0070, CONTROL_ID:03E9, ""
>                                 |
> :00402E83 68E9030000               push 000003E9     ;控件 ID
> :00402E88 55                       push ebp
> :00402E89 FFD3                     call ebx     ;得到注册码
> :00402E8B A188974000               mov eax, dword ptr [00409788]
> :00402E90 803800                   cmp byte ptr [eax], 00
> :00402E93 0F8438010000             je 00402FD1
> :00402E99 8B0D7C974000             mov ecx, dword ptr [0040977C]
> :00402E9F 803900                   cmp byte ptr [ecx], 00
> :00402EA2 0F8429010000             je 00402FD1
> :00402EA8 50                       push eax     ;压入用户名
> :00402EA9 E822FeFFFF               call 00402CD0     ;关键 CALL
> :00402EAE 8B3D7C974000             mov edi, dword ptr [0040977C]  ;假码
> :00402EB4 A188974000               mov eax, dword ptr [00409788]  ;真码
> :00402EB9 83C404                   add esp, 00000004
> :00402EBC 8BF7                     mov esi, edi
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402EDC(C)
> |
> :00402EBE 8A10                     mov dl, byte ptr [eax]
> :00402EC0 8ACA                     mov cl, dl
> :00402EC2 3A16                     cmp dl, byte ptr [esi]
> :00402EC4 751C                     jne 00402EE2
> :00402EC6 84C9                     test cl, cl
> :00402EC8 7414                     je 00402EDE
> :00402ECA 8A5001                   mov dl, byte ptr [eax+01]
> :00402ECD 8ACA                     mov cl, dl
> :00402ECF 3A5601                   cmp dl, byte ptr [esi+01]
> :00402ED2 750E                     jne 00402EE2
> :00402ED4 83C002                   add eax, 00000002
> :00402ED7 83C602                   add esi, 00000002
> :00402EDA 84C9                     test cl, cl
> :00402EDC 75E0                     jne 00402EBE
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402EC8(C)
> |
> :00402EDE 33C0                     xor eax, eax
> :00402EE0 EB05                     jmp 00402EE7
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:00402EC4(C), :00402ED2(C)
> |
> :00402EE2 1BC0                     sbb eax, eax
> :00402EE4 83D8FF                   sbb eax, FFFFFFFF
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402EE0(U)
> |
> :00402EE7 85C0                     test eax, eax
> :00402EE9 0F848D000000             je 00402F7C
> 
> \* Possible StringData Ref from Data Obj ->"52341546"    ;骗人的
>                                 |
> :00402EEF BEA4904000               mov esi, 004090A4
> :00402EF4 8BC7                     mov eax, edi
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402F14(C)
> |
> :00402EF6 8A10                     mov dl, byte ptr [eax]
> :00402EF8 8ACA                     mov cl, dl
> :00402EFA 3A16                     cmp dl, byte ptr [esi]
> :00402EFC 751C                     jne 00402F1A
> :00402EFE 84C9                     test cl, cl
> :00402F00 7414                     je 00402F16
> :00402F02 8A5001                   mov dl, byte ptr [eax+01]
> :00402F05 8ACA                     mov cl, dl
> :00402F07 3A5601                   cmp dl, byte ptr [esi+01]
> :00402F0A 750E                     jne 00402F1A
> :00402F0C 83C002                   add eax, 00000002
> :00402F0F 83C602                   add esi, 00000002
> :00402F12 84C9                     test cl, cl
> :00402F14 75E0                     jne 00402EF6
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402F00(C)
> |
> :00402F16 33C0                     xor eax, eax
> :00402F18 EB05                     jmp 00402F1F
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:00402EFC(C), :00402F0A(C)
> |
> :00402F1A 1BC0                     sbb eax, eax
> :00402F1C 83D8FF                   sbb eax, FFFFFFFF
> 
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00402F18(U)
> |
> :00402F1F 85C0                     test eax, eax
> :00402F21 7459                     je 00402F7C
> :00402F23 A180974000               mov eax, dword ptr [00409780]
> :00402F28 6A00                     push 00000000
> :00402F2A 83F803                   cmp eax, 00000003
> 
> \* Possible StringData Ref from Data Obj ->"用户注册"
>                                 |
> :00402F2D 6898904000               push 00409098
> :00402F32 7D23                     jge 00402F57
> 
> \* Possible StringData Ref from Data Obj ->"注册码错误！请重新输入！"
>                                 |
> :00402F34 687C904000               push 0040907C
> :00402F39 55                       push ebp
> 
> \* Reference To: USER32.MessageBoxA, Ord:01BEh
>                                 |
> :00402F3A FF1534714000             Call dword ptr [00407134]
> :00402F40 A180974000               mov eax, dword ptr [00409780]
> :00402F45 5F                       pop edi
> :00402F46 40                       inc eax
> :00402F47 5E                       pop esi
> :00402F48 A380974000               mov dword ptr [00409780], eax
> :00402F4D 5D                       pop ebp
> :00402F4E B801000000               mov eax, 00000001
> :00402F53 5B                       pop ebx
> :00402F54 C21000                   ret 0010
> ```

根据错误错息找串式参考，很容易找到上面这些。然后从这个函数的开头开始分析。

首先值得一提的是程序取得文本框文本的方法。在这个程序中如果你下 `GetWindowText`, `GetDlgItemText` 这些断点都不能断到，那么它是靠的什么方法呢？我们可以在 `402E65` 这一句看到把一个 API 函数 `SendDlgItemMessage` 的地址给了 `EBX`，然后下面两次调用 `EBX`。可以想到这就是取用户名和注册码了，具体说明见下：

```c
LONG SendDlgItemMessage(

    HWND hDlg,  // 对话框句柄

    int nIDDlgItem,  // 控件的 ID

    UINT Msg,  // 要发送的消息

    WPARAM wParam,  // 消息的第一个附加值

    LPARAM lParam   // 消息的第二个附加值

   );
```

对照上面的程序，可以发现它向 ID 为 `3E8h` 即 `1000` 的控件（用资源编辑工具可以发现这就是用户名的输入框）发送一个编号为 `0D` 的消息，这个消息表示什么呢？在 `winuser.h` 里有定义：

```c
#define WM_SETTEXT    0x000C

#define WM_GETTEXT    0x000D

#define WM_GETTEXTLENGTH  0x000E   
```

呵呵，不出所料是 `WM_GETTEXT`。再来看 `WM_GETTEXT` 的附加值有什么意义



# `WM_GETTEXT`  

`wParam = (WPARAM) cchTextMax;   // number of characters to copy` 得到的字串长度  

`lParam = (LPARAM) lpszText;     // address of buffer for text` 字串存放的地址  

再看上面的程序就很明显了，第一个 `CALL` 的 `wParam` 为 `10`，字串长度，`lParam` 为 `[409788]`，这是用户名的位置，记好。第二个 `CALL` 得到假注册码放在 `[40977C]`。  

很不错的方式，躲开了常见的 `API`，让我想起了 `DELPHI`。  

接下去是判断是否为空。然后把用户名 `PUSH` 进了，接着 `CALL`。呵呵~~~~还不快跟进：  

```
代码:
* Referenced by a CALL at Addresses:
|:00402EA9   , :004030ED    
|
:00402CD0 53                        push ebx
:00402CD1 56                        push esi
:00402CD2 57                        push edi
:00402CD3 8B7C2410                  mov edi, dword ptr [esp+10]  ;EDI 指向用户名
:00402CD7 32DB                      xor bl, bl
:00402CD9 8BCF                      mov ecx, edi
:00402CDB 8A07                      mov al, byte ptr [edi]
:00402CDD 84C0                      test al, al
:00402CDF 740A                      je 00402CEB

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402CE9(C)
|
:00402CE1 02D8                      add bl, al       ;各字符累加起来
:00402CE3 8A4101                    mov al, byte ptr [ecx+01]  ;循环取字符
:00402CE6 41                        inc ecx       
:00402CE7 84C0                      test al, al      ;是否取完
:00402CE9 75F6                      jne 00402CE1      ;这是一个循环
          
;注意参与计算的是 AL,BL,即只取累加结果的低 8 位放在 BL

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402CDF(C)
|
:00402CEB A178974000                mov eax, dword ptr [00409778]   
:00402CF0 33F6                      xor esi, esi
:00402CF2 A384974000                mov dword ptr [00409784], eax
:00402CF7 A174974000                mov eax, dword ptr [00409774]
:00402CFC 85C0                      test eax, eax
:00402CFE 7E2D                      jle 00402D2D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402D2B(C)
|
:00402D00 8A0C3E                    mov cl, byte ptr [esi+edi]  ;EDI 指向用户名,ESI 循环变量,取字符
:00402D03 32CB                      xor cl, bl       ;每次取的字符与上面累加结果 BL 异或
:00402D05 51                        push ecx       ;把异或所得结果压入
:00402D06 E895FFFFFF                call 00402CA0       ;关键 CALL，跟进
{
  :00402CA0 0FBE442404              movsx eax, byte ptr [esp+04]  ;EAX=上面压入的结果
  :00402CA5 030584974000            add eax, dword ptr [00409784]  ;[409784]最初是 989681h
  :00402CAB 69C0697DAE42            imul eax, 42AE7D69
  :00402CB1 0531D40000              add eax, 0000D431    ; D431h=54321 
  :00402CB6 A384974000              mov dword ptr [00409784], eax  ;把结果放回,下次计算要用
  :00402CBB C1F810                  sar eax, 10       ;右移 10 位
  :00402CBE 83E00F                  and eax, 0000000F    ;只取最低 4 位（见后）
  :00402CC1 C3                      ret
}
:00402D0B 83C404                    add esp, 00000004
:00402D0E 88043E                    mov byte ptr [esi+edi], al  ;AL 是上面 CALL 返回的结果
:00402D11 3C0A                      cmp al, 0A       
:00402D13 0FBEC0                    movsx eax, al
:00402D16 7D05                      jge 00402D1D      ;如果 AL>=0A 就跳
:00402D18 83C030                    add eax, 00000030    ;如果 AL<0A 就加 30h
:00402D1B EB03                      jmp 00402D20

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402D16(C)
|
:00402D1D 83C041                    add eax, 00000041    ;如果 AL>=0A 就加 41h

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402D1B(U)
|
:00402D20 88043E                    mov byte ptr [esi+edi], al  ;把结果放好,这可是最后注册码哟
:00402D23 A174974000                mov eax, dword ptr [00409774]  ;[409774]是常量 8
:00402D28 46                        inc esi      ;ESI 是循环变量
:00402D29 3BF0                      cmp esi, eax       
:00402D2B 7CD3                      jl 00402D00      ;这里是一个循环

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402CFE(C)
|
:00402D2D C6043800                  mov byte ptr [eax+edi], 00
:00402D31 5F                        pop edi
:00402D32 5E                        pop esi
:00402D33 5B                        pop ebx
:00402D34 C3                        ret
```

先说说关于位操作的事。上面有一个 `SAR`，这个和 `SHR` 有一些区别，就在最高位的处理上。我也说不太明白，举个例子 `11011100 shr 4 = 00001101` 而 `11011100 sar 4 = 11111101`。不过这里右移 10 位，我们只取最后 4 位，所以高 10 位的变化是影响不到的，可以简单的用右移(`>>`)来取代。还有要说的就是 `AND`，有一个 `AND 0000000F`，我说是只取最低 4 位，可能有菜鸟不明白，其实 `AND` 只要有一个操作数为 0，结果肯定为 0，如果一个操作数是 1，结果肯定等于另一个操作数。比方说我们如果想“屏蔽”一个八位数的第三四位，只要让它 `AND 11110011` 就行了。同理，因为 `0F=1111`，所以 `AND 0000000F` 等于只取出了最后面的四位。  

可以发现经过这个 `CALL` 后形成的一个新的字串，仍然放在原用户名的位置。接下来的代码把它们取出，逐位比较，不等就跳。。。自己看看吧，练习一下，很简单的。  

最后要说的是它还设了个陷阱，后面有一个固定字串 `"52341546"`，输入这个会提示成功注册，但其实下次启动又成未注册了。  

整理下思路，先把用户名累加，把结果再和用户名的每位进行一些计算，就得到正确的注册码了。很简单的是不是？  

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void main()
{
  char name[20]={0};
  char password[20]={0};
  unsigned int s=0,k=0,i;

  printf("Please input your name:");
  scanf("%s",name);
  int len=strlen(name);
  for (i=0;i<len;i++)
  {
    s+=name[i];
    s%=0x100;    //取低 8 位
  }
  int LOCAL=0x989681;
  for (i=0;i<8;i++)
  {
    k=s^name[i];
    k=(k+LOCAL)*0x42AE7D69+0xD431;
    LOCAL=k;
    k>>=0x10;    //SAR
    k%=0x10;    //取最低 4 位
    if (k>=0xA) k+=0x41;
    else k+=0x30;
    password[i]=k;
  }
  printf("Your password is : %s\n",password);
  printf("Keygen by RoBa\nEnjoy Cracking!\n");
}
```

一个可用的用户名：`RoBa`  

注册码：`0P6N0089` `4dd466d0e1e0cc123afbe504da2462ca`
