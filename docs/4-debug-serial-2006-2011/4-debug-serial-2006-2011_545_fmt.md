
# SmartUndelete 的注册码寻找过程

> 原文：[`www.pediy.com/kssd/pediy11/105079.html`](https://www.pediy.com/kssd/pediy11/105079.html)

## 文章信息

*   【文章标题】: `SmartDelete` 破解
*   【文章作者】: 一朵雪花
*   【软件名称】: `SmartUndelete`
*   【软件大小】: 500 多 k
*   【下载地址】: 自己搜索下载
*   【加壳方式】: 无
*   【保护方式】: 注册码
*   【编写语言】: `C++`
*   【使用工具】: `OD`, `UltraEdit`
*   【软件介绍】: 删除文件恢复工具
*   【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

### 1. 首先用 `OD` 载入，然后搜索字符串信息

在找到的字符串列表中找到这一条：

`Ultra String Reference, 条目 6 Address=004046D9 Disassembly=push 004683E0 Text String=wrong registration key`

双击进入：

```
004046B0   .  51          push    ecx
004046B1   .  56          push    esi
004046B2   .  8BF1        mov     esi, ecx
004046B4   .  6A 01       push    1
004046B6   .  E8 9FC80200 call    00430F5A
004046BB   .  8B0D 2C834600 mov   ecx, dword ptr [46832C]         ;  SmartUnd.004680F4
```

走到这一步，数据窗口显示如下信息，感觉是注册码：

`ds:[0046832C]=004680F4 (SmartUnd.004680F4), ASCII "4K5TY-UT3C2-KWP54-71MN2"`

```
004046C1   .  8B46 5C     mov     eax, dword ptr [esi+5C]         ;  这里保存这我们输入的序列号
004046C4   .  51          push    ecx
004046C5   .  50          push    eax
004046C6   .  E8 05F50100 call    00423BD0
004046CB   .  83C4 08     add     esp, 8
004046CE   .  85C0        test    eax, eax
004046D0   .  6A 00       push    0
004046D2   .  74 14       je      short 004046E8
004046D4   .  68 F8834600 push    004683F8                          ;  message
004046D9   .  68 E0834600 push    004683E0                          ;  wrong registration key
004046DE   .  8BCE        mov     ecx, esi
004046E0   .  E8 F9BB0200 call    004302DE
004046E5   .  5E          pop     esi
004046E6   .  59          pop     ecx
004046E7   .  C3          retn
004046E8   >  8D5424 08   lea     edx, dword ptr [esp+8]           ;  |
004046EC   .  52          push    edx                               ;  |pHandle
004046ED   .  6A 00       push    0                                 ;  |pSecurity = NULL
004046EF   .  68 3F000F00 push    0F003F                            ;  |Access = KEY_ALL_ACCESS
004046F4   .  6A 00       push    0                                 ;  |Options = REG_OPTION_NON_VOLATILE
004046F6   .  6A 00       push    0                                 ;  |Class = NULL
004046F8   .  6A 00       push    0                                 ;  |Reserved = 0
004046FA   .  68 BC834600 push    004683BC                          ;  |software\smart soft\smartundelete
004046FF   .  68 01000080 push    80000001                          ;  |hKey = HKEY_CURRENT_USER
00404704   .  FF15 08404500 call    dword ptr [<&ADVAPI32.RegCreateK>; \RegCreateKeyExA
0040470A   .  85C0        test    eax, eax
0040470C   .  75 31       jnz     short 0040473F
0040470E   .  8B15 2C834600 mov   edx, dword ptr [46832C]         ;  SmartUnd.004680F4
00404714   .  57          push    edi
00404715   .  8BFA        mov     edi, edx
00404717   .  83C9 FF     or      ecx, FFFFFFFF
0040471A   .  F2:AE       repne   scas byte ptr es:[edi]
0040471C   .  F7D1        not     ecx
0040471E   .  51          push    ecx                                 ;  /BufSize
0040471F   .  52          push    edx                                 ;  |Buffer => SmartUnd.004680F4
00404720   .  6A 01       push    1                                   ;  |ValueType = REG_SZ
00404722   .  50          push    eax                                 ;  |Reserved
00404723   .  8B4424 18   mov     eax, dword ptr [esp+18]           ;  |
00404727   .  68 A8834600 push    004683A8                          ;  |registration key
0040472C   .  50          push    eax                                 ;  |hKey
0040472D   .  FF15 04404500 call    dword ptr [<&ADVAPI32.RegSetValu>; \RegSetValueExA
00404733   .  8B4C24 08   mov     ecx, dword ptr [esp+8]
00404737   .  51          push    ecx                                 ;  /hKey
00404738   .  FF15 34404500 call    dword ptr [<&ADVAPI32.RegCloseKe>; \RegCloseKey
0040473E   .  5F          pop     edi
0040473F   >  6A 00       push    0
00404741   .  68 98834600 push    00468398                          ;  congratulations
00404746   .  68 34834600 push    00468334                          ;  you have successfully registered smartundelete. now the recover functionality is fully available.
0040474B   .  8BCE        mov     ecx, esi
0040474D   .  C705 94E24600> mov   dword ptr [46E294], 0
00404757   .  E8 82BB0200 call    004302DE
0040475C   .  8BCE        mov     ecx, esi
0040475E   .  E8 5FED0200 call    004334C2
00404763   .  5E          pop     esi
00404764   .  59          pop     ecx
00404765   .  C3          retn
```

在地址 `004046B0` 处下断点，单步几下，就看到注册码出现在内存中，而且是保存在全局位置中。

感觉注册码可能是明文保存在程序中的。觉得不可思议。

### 2. 第二步

*   重新运行程序，让程序停在入口点;
*   在 `cmd` 插件中 `dd 46832C`
*   数据窗口中的值为：

    ```
    0046832C  004680F4  ASCII "4K5TY-UT3C2-KWP54-71MN2"
    00468330  5A5A5A5A
    00468334  20756F59
    00468338  65766148
    0046833C  63755320
    00468340  73736563
    00468344  6C6C7566
    ```

*   注册码确实是明文存放在程序中。
*   用 `UltraEdit` 打开程序，切换到文本模式，搜索 `4K5TY-UT3C2-KWP54-71MN2` 字符串。
*   找到以下信息：“`ZZZZ4K5TY-UT3C2-KWP54-71MN2` `http://www.smartundelete.com/order.htm`”。

### 3. 第三步

顺便提一下，虽然注册码明文存在内存中，但是用 `winhex` 来寻找，仍然是不可能的。

### 4. 最后

*   21 世纪了，还有作者这么善良的人......让人感慨。`![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")`
*   程序的保护机制很弱，但是程序的功能还是很强的，以前见到的文件恢复程序要几十 M 大小，恢复的效果感觉都还不如它。
*   建议大家收藏，以备不时之需。
*   程序很小，有时间可以用来学习驱动编写。
*   最后，求版主给个邀请码吧！

--------------------------------------------------------------------------------

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`2010 年 01 月 13 日 上午 12:04:01`
