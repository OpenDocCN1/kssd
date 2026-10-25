
# 汇编代码片段

```
51                      push
ecx
:004553C2 8D8558FFFFFF            lea eax, dword ptr [ebp+FFFFFF58]
:004553C8 52
         push edx
:004553C9 8D8D68FFFFFF
       lea ecx, dword ptr [ebp+FFFFFF68]
:004553CF 50
                   push eax
:004553D0
8D9578FFFFFF            lea edx, dword ptr [ebp+FFFFFF78]
:004553D6
51                      push
ecx
:004553D7 8D4588
 lea eax, dword ptr [ebp-78]
:004553DA 52
             push edx
:004553DB 8D4D98
               lea ecx, dword ptr [ebp-68]
:004553DE
50                      push
eax
:004553DF 8D55A8
 lea edx, dword ptr [ebp-58]
:004553E2 51
             push ecx
:004553E3 8D45B8
               lea eax, dword ptr [ebp-48]
:004553E6
52                      push
edx
:004553E7 50
   push eax
:004553E8 6A0C
       push 0000000C
* Reference To: `MSVBVM60.__vbaFreeVarList`, Ord:0000h
 |
:004553EA FF1524104000            Call dword ptr [00401024]
:004553F0 83C434
     add esp, 00000034
:004553F3 C3
             ret
```

## 【算法总结】

其实分析完了也就知道这个程序并不复杂，虽然用了浮点运算，好象没多大作用？算法如下：

一、取用户号，将前3位换到最后。如`783009226` -> `009226783`

二、依次取变化后的用户号的数字，求其立方值

三、依次求循环次数的立方值

四、两者依次相加

五、相加之和依次与`A`求模！每次求模的结果就是注册码了！

## 【完美爆破】

`004718E5 1BF6`
`sbb esi, esi`

改为： `33F6`
`xor esi, esi`

## 【KeyMake之{54th}内存注册机】

中断地址：`4718CC`

中断次数：`1`

第一字节：`8B`

指令长度：`2`

内存方式：`EAX`

宽字符串

## 【注册信息保存】

```
REGEDIT4
[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\TspyxhtFigmnvfGlqnckt\Section1]
"Key01"="7\"$,6/6\""
"Key02"="2'\r.18"
[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\TspyxhtFigmnvfGlqnckt\Section2]
"Key01"="247"
"Key02"="钞己由化"
```

## 【整理】

用户号：`783009226`

注册码：`186232646`

```
Cracked By 巢水工作坊――fly【OCN】
2003-4-12   16:06
```
