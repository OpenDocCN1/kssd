
# FolderView V1.81

> 原文：[`www.pediy.com/kssd/pediy06/pediy6061.htm`](https://www.pediy.com/kssd/pediy06/pediy6061.htm)

**FolderView V1.81 注册分析**

(附：VB 注册码源代码：(初学))

(说明：此版本与 1.7 版本注册计算大至一样，只是个别参数改变!)

软件名称：`FolderView V1.81`

软件介绍：把文件夹里的内容按条件(时间、大小、名称等)进行列表，导出并打印!

软件下载：`http://js\-http.skycn.net/down/FVSetup.exe`

破解时间：`2003-05-18`

破解工具：略

开始：......略略，来到这里(注册计算分析的地方)：

\* Referenced by a CALL at Address:

```
|:00407D70   

|

:00407E20 81EC00010000          sub esp, 00000100
:00407E26 A0F4784100            mov al, byte ptr [004178F4]
:00407E2B 53                   push ebx
:00407E2C 55                   push ebp
:00407E2D 56                   push esi
:00407E2E 57                   push edi
:00407E2F 88442410             mov byte ptr [esp+10], al
:00407E33 B93F000000           mov ecx, 0000003F
:00407E38 33C0                 xor eax, eax
:00407E3A 8D7C2411             lea edi, dword ptr [esp+11]
:00407E3E F3                   repz
:00407E3F AB                   stosd
:00407E40 66AB                 stosw
:00407E42 AA                   stosb
:00407E43 8BBC2414010000       mov edi, dword ptr [esp+00000114]
:00407E4A 57                   push edi
```

\* Reference To: `KERNEL32.lstrlenA`, `Ord:03BFh`

```
                          |
:00407E4B FF156C214100     Call dword ptr [0041216C]  ///  检查用户名是否为空？
:00407E51 8BF0             mov esi, eax   
:00407E53 33C9             xor ecx, ecx  //清零
:00407E55 33C0             xor eax, eax  //清零
:00407E57 85F6             test esi, esi
:00407E59 7613             jbe 00407E6E   //如果为空则出错
:00407E5B 8B156C4A4100     mov edx, dword ptr [00414A6C]   //固定值 EDX=50     第一组注册码
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407E6C(C)

|

:00407E61 0FBE1C38          movsx ebx, byte ptr [eax+edi]  //依次取用户名的 ASC 值进行计算!=ebx
:00407E65 03DA             add ebx, edx      //ebx=ebx+edx  即：ebx=ebx+50
:00407E67 03CB             add ecx, ebx      //ecx=ecx+ebx
:00407E69 40                inc eax           //累加 1
:00407E6A 3BC6             cmp eax, esi
:00407E6C 72F3             jb 00407E61       //未完则继续循环，直至用户名循环结束后,
///其值为： 第一组注册码 CODE1
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407E59(C)

|

:00407E6E 8B9C2418010000    mov ebx, dword ptr [esp+00000118]
:00407E75 51                push ecx
```

\* Possible StringData Ref from Data Obj ->`"%u-"`       //注册码形式为："-"格式

```
                          |
:00407E76 68C44A4100        push 00414AC4
:00407E7B 53                push ebx
```

\* Reference To: `USER32.wsprintfA`, `Ord:02D5h`

```
                          |
:00407E7C FF1510224100     Call dword ptr [00412210]
:00407E82 83C40C           add esp, 0000000C
:00407E85 33C9             xor ecx, ecx
:00407E87 33C0             xor eax, eax
:00407E89 85F6             test esi, esi
:00407E8B 7614             jbe 00407EA1
:00407E8D 8B15704A4100     mov edx, dword ptr [00414A70]    //固定值 EDX=40    第二组注册码   
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407E9F(C)

|

:00407E93 0FBE2C38          movsx ebp, byte ptr [eax+edi]   //依次取用户名的 ASC 值进行计算!=ebp
:00407E97 0FAFEA           imul ebp, edx                   //ebp=ebp*edx
:00407E9A 03CD             add ecx, ebp                    //ecx=ecx+ebp
:00407E9C 40                inc eax
:00407E9D 3BC6             cmp eax, esi
:00407E9F 72F2             jb 00407E93                     //未完则继续循环，直至用户名循环结束后,
///其值为： 第二组注册码 CODE2
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407E8B(C)

|

:00407EA1 51                push ecx
:00407EA2 8D4C2414          lea ecx, dword ptr [esp+14]
```

\* Possible StringData Ref from Data Obj ->`"%u-"`

```
                          |
:00407EA6 68C44A4100        push 00414AC4
:00407EAB 51                push ecx
```

\* Reference To: `USER32.wsprintfA`, `Ord:02D5h`

```
                          |
:00407EAC FF1510224100     Call dword ptr [00412210]
:00407EB2 83C40C           add esp, 0000000C
:00407EB5 8D542410         lea edx, dword ptr [esp+10]
:00407EB9 52                push edx
:00407EBA 53                push ebx
```

\* Reference To: `KERNEL32.lstrcatA`, `Ord:03B0h`

```
                          |
:00407EBB FF1598214100     Call dword ptr [00412198]
:00407EC1 33C9             xor ecx, ecx
:00407EC3 33C0             xor eax, eax
:00407EC5 85F6             test esi, esi
:00407EC7 7613             jbe 00407EDC
:00407EC9 8B15744A4100     mov edx, dword ptr [00414A74]    //固定值 EDX=30    第三组注册码   
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407EDA(C)

|

:00407ECF 0FBE2C38          movsx ebp, byte ptr [eax+edi]    //依次取用户名的 ASC 值进行计算!=ebp
:00407ED3 03EA             add ebp, edx                    //ebp=ebp+edx
:00407ED5 03CD             add ecx, ebp                    //ecx=ecx+ebp
:00407ED7 40                inc eax
:00407ED8 3BC6             cmp eax, esi
:00407EDA 72F3             jb 00407ECF         //未完则继续循环，直至用户名循环结束后,
///其值为： 第三组注册码 CODE2
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407EC7(C)

|

:00407EDC 51                push ecx
:00407EDD 8D442414          lea eax, dword ptr [esp+14]
```

\* Possible StringData Ref from Data Obj ->`"%u-"`

```
                          |
:00407EE1 68C44A4100        push 00414AC4
:00407EE6 50                push eax
```

\* Reference To: `USER32.wsprintfA`, `Ord:02D5h`

```
                          |
:00407EE7 FF1510224100     Call dword ptr [00412210]
:00407EED 83C40C           add esp, 0000000C
:00407EF0 8D4C2410         lea ecx, dword ptr [esp+10]
:00407EF4 51                push ecx
:00407EF5 53                push ebx
```

\* Reference To: `KERNEL32.lstrcatA`, `Ord:03B0h`

```
                          |
:00407EF6 FF1598214100     Call dword ptr [00412198]
:00407EFC 33C9             xor ecx, ecx
:00407EFE 33C0             xor eax, eax
:00407F00 85F6             test esi, esi
:00407F02 7614             jbe 00407F18
:00407F04 8B15784A4100     mov edx, dword ptr [00414A78]  //固定值 EDX=11    第四组注册码   
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407F16(C)

|

:00407F0A 0FBE2C38          movsx ebp, byte ptr [eax+edi]    //依次取用户名的 ASC 值进行计算!=ebx
:00407F0E 0FAFEA           imul ebp, edx                   //  ebp*= edx
:00407F11 03CD             add ecx, ebp                    //ecx=ecx+ebp
:00407F13 40                inc eax
:00407F14 3BC6             cmp eax, esi
:00407F16 72F2             jb 00407F0A                     //未完则继续循环，直至用户名循环结束后,
///其值为： 第四组注册码 CODE4
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00407F02(C)

|

:00407F18 51                push ecx
:00407F19 8D542414          lea edx, dword ptr [esp+14]
```

\* Possible StringData Ref from Data Obj ->`"%u"`  把四组注册码串成一起!OK

## 总结：

注册码共分为四组：

1. **第一组：**`code1`
   `ebp = ebp(依次取用户名的 ASC 值)+50`
   `ecx = ecx+ebp`

2. **第二组：**`code2`
   `ebp = ebp(依次取用户名的 ASC 值)*40`
   `ecx = ecx+ebp`

3. **第三组：**`code2`
   `ebp = ebp(依次取用户名的 ASC 值)+30`
   `ecx = ecx+ebp`

4. **第四组：**`code2`
   `ebp = ebp(依次取用户名的 ASC 值)* 11`
   `ecx = ecx+ebp`



注册码为：`code1-cdoe2-code3-code4` 

注册成功后保存在注册表：

`[HKEY_CURRENT_USER\Software\FolderView\Registration]`

`"Name"="qhj"`

`"Code"="473-12920-413-3553"`

===================================================

附：VB 注册码源代码：(初学)

```vb
Private Sub 计算()  

If Text1.Text = "" Then  

MsgBox "注意：用户名不能为空!", 48, "绝密档案"  

Text1.SetFocus  

Exit Sub  

End If 

Dim I, name， name_h, name_h2, code1, code2, code3, code4 

'''先把名称串成一串  

name = Trim(Text1.Text)  '清除左右空格  

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))    ''' 求出相应的 ASC 码值  

ebp = name_h2 + 50  

ecx = ecx + ebp  

code1 = ecx  

Next I 

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))  

ebp = name_h2 * 40  

ecx = ecx + ebp  

code2 = ecx  

Next I 

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))  

ebp = name_h2 + 30  

ecx = ecx + ebp  

code3 = ecx  

Next I 

ecx = 0  

For I = 1 To Len(name)  

name_h2 = Asc(Mid(name, I, 1))  

ebp = name_h2 * 11  

ecx = ecx + ebp  

code4 = ecx  

Next I 

Text2.Text = code1 & "-" & code2 & "-" & code3 & "-" & code4 

Private Sub Text1_KeyPress(KeyAscii As Integer)  

If KeyAscii > 255 Or KeyAscii < 0 Then  

KeyAscii = 0  

MsgBox "暂不支持中文", 48, "绝密档案"  

End If  

End Sub 

''暂不支持中文，请高手指点! 

绝密档案 2003-05-19  

[hongjian.126.com](http://hongjian.126.com)
```

算法分析本身就不完整，当然支持不了中文……贴个 TC 2.0 注册机，写得不好，请多多见谅，支持中文注册！

```c
main()  

{  

int i;  

char name[80];  

long sn1=0,sn2=0,sn3=0,sn4=0;  

clrscr();  

printf("\n\nFolderView V1.8 KeyGen *Make by PaulYoung*\n\nPlease input your name:");  

gets(name);  

for (i=0;name[i];i++)  

{  

sn1+=name[i]+0x32;  

sn2+=name[i]*0x28;  

sn3+=name[i]+0x1E;  

sn4+=name[i]*0xB;  

}  

printf(" Your RegisterCode is:%lu-%lu-%lu-%lu\n\n",sn1,sn2,sn3,sn4);  

getch();  

}   

4dd466d0e1e0cc123afbe504da2462ca
```
