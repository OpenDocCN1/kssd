
# shellcode 汇编之采用 API 函数

> 原文：[`www.pediy.com/kssd/pediy11/125105.html`](https://www.pediy.com/kssd/pediy11/125105.html)

**标 题:** 【原创】shellcode 汇编之采用 API 函数
**作 者:** `jgaoabc`
**时 间:** 2010-11-19

内容比较肤浅，大牛莫笑。

按照 `shellcode` 代码的方式用汇编语言编写程序的优点在于它的可移植性，因为它没有重定位，没有输入表。如果把它写成一个子程序，可以在任意用得着的程序中粘贴调用它。

重定位可以采用自定位的方式来解决，同样输入表函数也可以采用搜索的方式来取得。

要取得输入表函数的内存指针，首先须取得 `kernel32.dll` 基址，这个不难，看雪论坛是我们的老师，看这个文章《Win 7 下定位 kernel32.dll 基址及 shellcode 编写》`http://bbs.pediy.com/showthread.php?t=122260`。其次，采用搜索取得各 DLL 的 API 函数的地址，网络上也有很多办法，我从 `Poison Ivy 2.3.2` 中找到了一个好办法，附带说下 `Poison Ivy 2.3.2` 的服务端，它就是按照 `shellcode` 格式编写的，它的代码是可移植的，但是它在 `WINDOWS 7` 下却不能运行，因为它取得 `kernel32.dll` 基址的方法不适用于 `WINDOWS 7`，如果把它换成看雪论坛的方法（也就是下面的代码），就可以在 `WINDOWS 7` 运行了。

取得 `kernel32.dll` 基址的子程序汇编代码：（二进制代码请看附件）

```
00401000  KrnlB>   51                      push ecx
00401001       56                      push esi
00401002       57                      push edi
00401003       33C9                    xor ecx,ecx
00401005       64:8B71 30              mov esi,dword ptr fs:[ecx+30]
00401009       8B76 0C                 mov esi,dword ptr ds:[esi+C]
0040100C       8B76 1C                 mov esi,dword ptr ds:[esi+1C]
0040100F       8B46 08                 mov eax,dword ptr ds:[esi+8]
00401012       8B7E 20                 mov edi,dword ptr ds:[esi+20]
00401015       8B36                    mov esi,dword ptr ds:[esi]
00401017       66:394F 18              cmp word ptr ds:[edi+18],cx
0040101B     ^ 75 F2                   jnz short KrnlBase.0040100F
0040101D       5F                      pop edi
0040101E       5E                      pop esi
0040101F       59                      pop ecx
00401020       C3                      retn
```

搜索 API 函数的子程序汇编代码：

```
00401021       55                      push ebp
00401022       8BEC                    mov ebp,esp
00401024       83C4 EC                 add esp,-14
00401027       56                      push esi
00401028       53                      push ebx
00401029       57                      push edi
0040102A       52                      push edx
0040102B       51                      push ecx
0040102C       8B45 10                 mov eax,dword ptr ss:[ebp+10]
0040102F       33D2                    xor edx,edx
00401031       92                      xchg eax,edx
00401032       BE 3C000000             mov esi,3C
00401037       0375 0C                 add esi,dword ptr ss:[ebp+C]
0040103A       8B06                    mov eax,dword ptr ds:[esi]
0040103C       0345 0C                 add eax,dword ptr ss:[ebp+C]
0040103F       8B70 78                 mov esi,dword ptr ds:[eax+78]
00401042       83C6 18                 add esi,18
00401045       0375 0C                 add esi,dword ptr ss:[ebp+C]
00401048       8B06                    mov eax,dword ptr ds:[esi]
0040104A       8945 EC                 mov dword ptr ss:[ebp-14],eax
0040104D       83C6 04                 add esi,4
00401050       8D7D F8                 lea edi,dword ptr ss:[ebp-8]
00401053       AD                      lods dword ptr ds:[esi]
00401054       0345 0C                 add eax,dword ptr ss:[ebp+C]
00401057       AB                      stos dword ptr es:[edi]
00401058       8945 F8                 mov dword ptr ss:[ebp-8],eax
0040105B       AD                      lods dword ptr ds:[esi]
0040105C       0345 0C                 add eax,dword ptr ss:[ebp+C]
0040105F       50                      push eax
00401060       AB                      stos dword ptr es:[edi]
00401061       8945 F4                 mov dword ptr ss:[ebp-C],eax
00401064       8B06                    mov eax,dword ptr ds:[esi]
00401066       0345 0C                 add eax,dword ptr ss:[ebp+C]
00401069       8945 F0                 mov dword ptr ss:[ebp-10],eax
0040106C       5E                      pop esi
0040106D       C745 FC 00000000        mov dword ptr ss:[ebp-4],0
00401074       56                      push esi
00401075       8B06                    mov eax,dword ptr ds:[esi]
00401077       0345 0C                 add eax,dword ptr ss:[ebp+C]
0040107A       97                      xchg eax,edi    ; 这里下断，数据窗口查看 eax 的值，然后在数据窗口粘贴要取得的 API 函数字符的二进制
0040107B       8BDF                    mov ebx,edi
0040107D       57                      push edi
0040107E       32C0                    xor al,al
00401080       AE                      scas byte ptr es:[edi]
00401081     ^ 75 FD                   jnz short KrnlBase.00401080
00401083       5E                      pop esi
00401084       2BFB                    sub edi,ebx
00401086       52                      push edx
00401087       FC                      cld
00401088       33C9                    xor ecx,ecx
0040108A       49                      dec ecx
0040108B       8BD1                    mov edx,ecx
0040108D       33C0                    xor eax,eax
0040108F       33DB                    xor ebx,ebx
00401091       AC                      lods byte ptr ds:[esi]
00401092       32C1                    xor al,cl
00401094       8ACD                    mov cl,ch
00401096       8AEA                    mov ch,dl
00401098       8AD6                    mov dl,dh
0040109A       B6 08                   mov dh,8
0040109C       66:D1EB                 shr bx,1
0040109F       66:D1D8                 rcr ax,1
004010A2       73 09                   jnb short KrnlBase.004010AD
004010A4       66:35 2083              xor ax,8320
004010A8       66:81F3 B8ED            xor bx,0EDB8
004010AD       FECE                    dec dh
004010AF     ^ 75 EB                   jnz short KrnlBase.0040109C
004010B1       33C8                    xor ecx,eax
004010B3       33D3                    xor edx,ebx
004010B5       4F                      dec edi
004010B6     ^ 75 D5                   jnz short KrnlBase.0040108D
004010B8       F7D2                    not edx
004010BA       F7D1                    not ecx
004010BC       8BC2                    mov eax,edx
004010BE       C1C0 10                 rol eax,10
004010C1       66:8BC1                 mov ax,cx
004010C4       5A                      pop edx
004010C5       3BD0                    cmp edx,eax     ; eax 中的值是 DLL 函数的输出表各个函数的特征值，edx 是指定的特征值。
004010C7       74 0A                   je short KrnlBase.004010D3
004010C9       5E                      pop esi
004010CA       83C6 04                 add esi,4
004010CD       8345 FC 01              add dword ptr ss:[ebp-4],1
004010D1     ^ EB A1                   jmp short KrnlBase.00401074
004010D3       5E                      pop esi
004010D4       8B45 FC                 mov eax,dword ptr ss:[ebp-4]
004010D7       D1E0                    shl eax,1
004010D9       0345 F0                 add eax,dword ptr ss:[ebp-10]
004010DC       33F6                    xor esi,esi
004010DE       96                      xchg eax,esi
004010DF       66:8B06                 mov ax,word ptr ds:[esi]
004010E2       66:C1E0 02              shl ax,2
004010E6       0345 F8                 add eax,dword ptr ss:[ebp-8]
004010E9       96                      xchg eax,esi
004010EA       8B06                    mov eax,dword ptr ds:[esi]
```



## 反汇编代码片段

```
004010EC     0345 0C              add eax,dword ptr ss:[ebp+C]  
004010EF     59                  pop ecx  
004010F0     5A                  pop edx  
004010F1     5F                  pop edi  
004010F2     5B                  pop ebx  
004010F3     5E                  pop esi  
004010F4     C9                  leave  
004010F5     C2 0C00             retn 0C  
```

运用上面的子程序取得 API 函数的格式如下：

```
00401020     68 0736F319         push 19F33607             API 函数特征值  
00401025     FF75 FC             push dword ptr ss:[ebp-4]   API 函数所属的 DLL 的基址  
00401028     6A 00               push 0  
0040102A     E8 B2120000         call 00401021               搜索 API 函数的子程序  
```

运行后 `eax` 就是 API 函数的内存指针

## 反调试实例说明

下面就我编写的一个反调试实例 `EnableWindow`（在看雪论坛查反调试技术实例）做下说明：

这个程序用到了二个函数，`user32.dll` 有 `GetForegroundWindow` 和 `EnableWindow`，当然要取得 `user32.dll` 的基址，须调用 `kernel32.dll` 的 `LoadLibraryA`。

```
004010F8     55                  push ebp  
004010F9     8BEC                mov ebp,esp  
004010FB     83EC 20             sub esp,20  
004010FE     53                  push ebx  
004010FF     56                  push esi  
00401100     57                  push edi  
00401101     E8 FAFEFFFF         call KrnlBase.00401000          ; 取得 kernel32.dll 基址  
00401106     8945 FC             mov dword ptr ss:[ebp-4],eax  
00401109     68 ADD13441         push 4134D1AD                    ; LoadLibraryA 的特征值  
0040110E     FF75 FC             push dword ptr ss:[ebp-4]         ; kernel32.dll 基址  
00401111     6A 00               push 0  
00401113     E8 09FFFFFF         call KrnlBase.00401021           ; 搜索 API 函数的子程序  
00401118     8945 F8             mov dword ptr ss:[ebp-8],eax  
0040111B     E8 07000000         call KrnlBase.00401127           ; 自定位 user32 的地址  
00401120     75 73               jnz short KrnlBase.00401195       ; user32  
00401122     65:72 33            jb short KrnlBase.00401158  
00401125     3200                xor al,byte ptr ds:[eax]  
00401127     FF55 F8             call dword ptr ss:[ebp-8]         ; 调用 LoadLibraryA  
0040112A     8BF0                mov esi,eax  
0040112C     68 3F235577         push 7755233F                     ; GetForegroundWindow 的特征值  
00401131     56                  push esi  
00401132     6A 00               push 0  
00401134     E8 E8FEFFFF         call KrnlBase.00401021  
00401139     8945 F4             mov dword ptr ss:[ebp-C],eax  
0040113C     68 09D99B70         push 709BD909                     ; EnableWindow 的特征值  
00401141     56                  push esi  
00401142     6A 00               push 0  
00401144     E8 D8FEFFFF         call KrnlBase.00401021  
00401149     8945 F0             mov dword ptr ss:[ebp-10],eax  
0040114C     FF55 F4             call dword ptr ss:[ebp-C]         ; user32.GetForegroundWindow  
0040114F     8BF8                mov edi,eax  
00401151     6A 00               push 0                              ; 禁止窗口  
00401153     57                  push edi  
00401154     FF55 F0             call dword ptr ss:[ebp-10]        ; user32.EnableWindow  
00401157     6A 01               push 1                              ; 允许窗口  
00401159     57                  push edi  
0040115A     FF55 F0             call dword ptr ss:[ebp-10]        ; user32.EnableWindow  
0040115D     5F                  pop edi  
0040115E     5E                  pop esi  
0040115F     5B                  pop ebx  
00401160     83C4 20             add esp,20  
00401163     5D                  pop ebp  
00401164     C3                  retn  
```

注意：运行禁止窗口的 `00401154` 之后，整个 `OD` 将被禁止，你只能在任务管理器关闭 `OD`。把禁止窗口和允许窗口一起运行比如在 `40115D` 设断就可以避免。

本程序在 `2003`、`XP` 和 `WINDOWS 7` 运行通过。

### 函数特征值取得的方法

先把 API 函数字符换成二进制代码，如 `EnableWindow`，先把它换成二进制 `456E61626C6557696E646F7700`，注意后面有 `00` 结束，这项工作可以在 `C32ASM` 中进行，然后在 `0040107A` 下断，在数据窗口跟踪 `eax` 的值，并且在数据窗口把 API 函数的二进制字符粘贴进去，然后再在 `004010C5` 下断，`eax` 里的值就是 API 函数的特征值。

## 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `KrnlBase.rar`  
|
