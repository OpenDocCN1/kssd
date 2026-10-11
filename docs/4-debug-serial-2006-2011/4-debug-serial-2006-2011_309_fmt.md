
# SiteSpinner 2.7.0 f 的注册分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-437.htm`](https://www.pediy.com/kssd/pediy09/pediy09-437.htm)

> `SiteSpinner` 2.7.0 f 的注册分析

> 平时会经常到 `www.download.com` 转一转，看看有什么新的软件可供把玩。某日在其首页上看到一个名为 `SiteSpinner` 的软件，号称是最方便使用的一站式网页设计软件，并且它的下载频度比较高，于是便对它发生了兴趣。

> `www.download.com` 上提供的软件版本号是“2.7.0 c”，而该软件的主页，`http://www.virtualmechanics.com` 上提供的最新版本号是“2.7.0 f”。由于“喜新厌旧”的脾性，就在该软件的主页上下载了一个最新版本。

> 下载回来的只是一个只有 100 多 K 的可执行程序，当时就怀疑是不是需要在线安装。运行之，果然提醒要到软件主站下载安装文件。这可不符合我们即装即用的意图。

> 取消安装后祭出 `Hex Workshop` 打开这个可执行程序，在靠近尾部的地方就会发现那个名为“`SiteSpinnerV270f.msi`”的安装文件的链接。我用它顺利完成了安装文件的下载并进行了安装。

> 这个软件的主程序是 `SiteSpinnerV2.exe`。用 `OllyICE` 载入后可以在反汇编代码的字里行间中看到软件是用 `MFC` 写的，我们根本不需要考虑解压或脱壳的问题。直接运行之，立即出现 `Splash` 窗口。`Splash` 窗口中提示输入 `Serial Number` 和 `Registration Code`。不过它已经给我们预设了一个以"TRIAL-"打头的 `Serial Number`。我们在 `Registration Code` 文本框中随便输入一些字符或数字，然后单击"OK"按钮，马上就会跳出标题为"Registration Not Accepted"的提示窗口，告诉您注册信息有误，要注意什么什么问题。

> 现在我们动用 `Ultra String Reference` 插件，搜索提示窗口内容中开头的那部分字符串，光标就会醒目指向 `48D160`。双击之，我们会看到如下代码片断：

```
0048D160  push     005CC864
0048D165  mov      ecx, 006A589C
0048D16A  call     <jmp.&MFC42.#537_CString::CString>
0048D16F  push     0048D180
0048D174  call     0059AA0C
0048D179  pop      ecx
0048D17A  retn

; 几个 NOP 指令

0048D180  mov      ecx, 006A589C
0048D185  jmp      <jmp.&MFC42.#800_CString::~CString>
```

> 这显然是 `CString` 对象的构造和析构函数，对象地址就是 `6A589C`。在数据窗口中查看 `6A589C` 处的内容，发现这里存放着一个指针，指向 `3B7D94`。再查看 `3B7D94` 处的内容，赫然就是我们要查找的字符串。

> 此时，我的第一感觉是对 `3B7D94` 处的内容设置内存访问断点，但事实证明这种做法是行不通的，因为程序会不停地被中断。这样，我们就只能看一看是哪些地方引用了 `6A589C`。于是，我们右击 `48D165` 处的那条指令，在出现的快捷菜单中选择“查找参考”，然后在其子菜单中选择“立即数”，`OllyICE` 就会在新的窗口中出现以下内容：

```
0048D165   mov      ecx, 006A589C
0048D180   mov      ecx, 006A589C
0048D2A6   mov      dword ptr [edi*4+6A589C], edx
0048E397   push     006A589C
0048E473   push     006A589C
0048EF34   mov      dword ptr [esi*4+6A589C], edx
```

> 前两条指令就在上边的那个代码段中，第三和最后一条指令处是为了清空 `CString` 对象的内容。于是，我们可以毫不犹豫地在 `48E397` 和 `48E473` 处下断，在单击提示窗口的"OK"按钮，使提示窗口关闭后，再次单击 `Splash` 窗口的"OK"按钮，程序就立即中断在 `48E473` 处。结合逐步设断的方法向上查看，您就会将目光集中在 `48D496` 处。这里的指令是 `CDialog::DoModal`，下面那条指令中的立即数 `2` 不就是 `IDCANCEL` 吗？好了，这里就是我们的切入点了。

```
... ...
0048D485  mov      ecx, dword ptr [esp+10]
0048D489  mov      dword ptr [6A5850], ecx
0048D48F  lea      ecx, dword ptr [esp+9C]
0048D496  call     <jmp.&MFC42.#2514_CDialog::DoModal>    ; 显示注册对话框
0048D49B  cmp      eax, 2
0048D49E  jnz      short 0048D4AC        ; 关闭对话框时若非单击了 CANCEL 按钮则转 48D4AC
0048D4A0  mov      byte ptr [6A5928], 0
0048D4A7  jmp      0048D544

; ... ...
; 这段代码转储注册信息，包括注册名、公司名、注册码和序列号
; ... ...

; 显然，下面这段代码是为了去掉 SerialNo 两端的空白字符
; (为了便于阅读，代码顺序会被调整)

0048D544  lea      ebx, dword ptr [ebp+17C]     ; SerialNo
0048D54A  lea      ecx, dword ptr [esp+18]
0048D54E  push     ebx
0048D54F  call     <jmp.&MFC42.#537_CString::CString>
0048D554  lea      ecx, dword ptr [esp+18]
0048D558  mov      byte ptr [esp+308], 4
0048D560  call     <jmp.&MFC42.#6283_CString::TrimRight>
0048D565  lea      ecx, dword ptr [esp+18]
0048D569  call     <jmp.&MFC42.#6282_CString::TrimLeft>
0048D56E  mov      edi, dword ptr [esp+18]
0048D572  or       ecx, FFFFFFFF
0048D575  xor      eax, eax
0048D577  repne    scas byte ptr es:[edi]
0048D579  not      ecx
0048D57B  sub      edi, ecx
0048D57D  mov      edx, ecx
0048D57F  mov      esi, edi
0048D581  mov      edi, ebx
0048D589  shr      ecx, 2
0048D58C  rep      movs dword ptr es:[edi], dword ptr [esi]
0048D58E  mov      ecx, edx
0048D591  and      ecx, 3
0048D594  rep      movs byte ptr es:[edi], byte ptr [esi]

; ... ...
; 同上，这段代码是为了去掉 RegCode 两端的空白字符
; ... ...

0048D5D7  mov      ecx, dword ptr [esp+10]
0048D5DD  mov      dword ptr [esp+30], ecx
0048D5CD  mov      al, byte ptr [6A5928]
0048D5DB  test     al, al
0048D5E1  je       0048E5FD
0048D5E7  lea      edi, dword ptr [ebp+1B0]     ; RegCode
0048D5ED  or       ecx, FFFFFFFF
0048D5F0  xor      eax, eax
0048D5F2  lea      ebx, dword ptr [ebp+196]     ; 取目标字节串地址
0048D5F8  repne    scas byte ptr es:[edi]
0048D5FA  not      ecx
0048D5FC  dec      ecx
0048D5FD  mov      byte ptr [esp+1C], al        ; 初始化目标字符串序号
0048D603  lea      esi, dword ptr [ecx-1]       ; esi 置为最末字符的序号
0048D601  test     ecx, ecx
0048D606  jle      short 0048D647
0048D608  /xor     edx, edx
0048D60A  |mov     dl, byte ptr [eax+5CB4F8]    ; 从 5CB4F8 始的表中取一字节
0048D610  |movsx   eax, byte ptr [eax+ebp+1B0]  ; 取 RegCode 的一个字符
0048D618  |sub     eax, edx           ; 字符值减去查表值
0048D61A  |sub     eax, 20            ; 再减去 0x20
0048D61D  |cmp     eax, 30            ; 与 0x30 比较
0048D620  |jge     short 0048D625
0048D622  |add     eax, 4A            ; 小于则加上 0x4A
0048D625  |mov     byte ptr [esi+ebx], al     ; 存入目标字节串
0048D628  |sub     esi, 2           ; 向前跳过一字符
0048D62B  |jns     short 0048D630
0048D62D  |lea     esi, dword ptr [ecx-2]     ; 若字符序号为负值则指向倒数第二个字符
0048D630  |mov     al, byte ptr [esp+1C]
0048D634  |inc     al
0048D636  |mov     byte ptr [esp+1C], al      ; 目标字节序号增 1
0048D63A  |mov     eax, dword ptr [esp+1C]
0048D63E  |and     eax, 0FF
0048D643  |cmp     eax, ecx           ; 处理完否？
0048D645  \jl      short 0048D608     ; 未处理完则继续循环
0048D647  test     ecx, ecx
0048D649  mov      byte ptr [ecx+ebx], 0       ; 目标字节串最后补 Null 字符
0048D64D  je       0048E2E7

; 调用 48E700，对字符串参数 1 逐字符进行如下变换，结果返回在参数 2 中
; 1、字符值不小于 0x62 则减去 0x20
; 2、字符值从 0x41 到 0x61，不变
; 3、字符值不大于 0x40 则加上 0x11

0048D653  lea      eax, dword ptr [esp+48]
0048D657  mov      ecx, ebp
0048D659  push     eax
0048D65A  lea      eax, dword ptr [ebp+17C]     ; SerialNo
0048D660  push     eax
0048D661  mov      dword ptr [esp+24], 0
0048D669  call     0048E700
0048D66E  mov      dword ptr [esp+2C], eax      ; 存放 SerialNo 的长度

; 48D672 始的代码比较变换后的 RegCode 的前 6 个字符是不是“g2r0`h”
; 根据前面的算法，“g2r0`h”对应的就是“EXTEND”
; RegCode="EXTEND"会是什么情况呢？
; 接着的代码中的提示信息会感谢您为扩展的试用期掏了腰包。这可不是我们所期望的。

; ... ...

; 48D8A0 始的代码比较变换后的 RegCode 的前 5 个字符是不是“1xg`u”
; 根据前面的算法，“1xg`u”对应的就是“RESET”
; 当 RegCode="RESET"时，程序会提醒您重置了试用版的序列号。这显然也不是我们所期望的。

; ... ...

0048D972  mov      esi, ebx        ; 变换后的 RegCode
0048D974  lea      eax, dword ptr [esp+48]   ; 变换后的 SerialNo
0048D978  /mov     dl, byte ptr [eax]
0048D97A  |mov     cl, dl
0048D97C  |cmp     dl, byte ptr [esi]
0048D97E  |jnz     short 0048D99C
0048D980  |test    cl, cl
0048D982  |je      short 0048D998
0048D984  |mov     dl, byte ptr [eax+1]
0048D987  |mov     cl, dl
0048D989  |cmp     dl, byte ptr [esi+1]
0048D98C  |jnz     short 0048D99C
0048D98E  |add     eax, 2
0048D991  |add     esi, 2
0048D994  |test    cl, cl
0048D996  \jnz     short 0048D978
0048D998  xor      eax, eax
0048D99A  jmp      short 0048D9A1
0048D99C  sbb      eax, eax
0048D99E  sbb      eax, -1
0048D9A1  test     eax, eax
0048D9A3  jnz      0048DD14

; 若变换后的 RegCode 等于变换后的 SerialNo

0048D9A9  lea      ecx, dword ptr [esp+10]   ; 就是程序给出的默认的 SerialNo 后面的那个数
0048D9AD  mov      byte ptr [esp+80], al
0048D9B4  mov      byte ptr [esp+64], al
0048D9B8  call     <jmp.&MFC42.#540_CString::CStrin>
0048D9BD  mov      eax, dword ptr [ebp+C4]
0048D9C3  mov      byte ptr [esp+308], 9
0048D9CB  push     eax
0048D9CC  lea      eax, dword ptr [esp+14]
0048D9D0  push     005B8678        ; %d
0048D9D5  push     eax
```



```
0048D9D6  call    <jmp.&MFC42.#2818\_CString::Forma>  ; 把上面那个数转成字符串  
0048D9DB  mov     edi, dword ptr \[esp+1C\]  
0048D9DF  or      ecx, FFFFFFFF  
0048D9E2  xor     eax, eax  
0048D9E4  add     esp, 0C  
0048D9E7  repne   scas byte ptr es:\[edi\]  
0048D9E9  not     ecx  
0048D9EB  sub     edi, ecx  
0048D9ED  lea     edx, dword ptr \[esp+80\]  
0048D9F4  mov     eax, ecx  
0048D9F6  mov     esi, edi  
0048D9F8  mov     edi, edx  
0048D9FA  lea     edx, dword ptr \[esp+80\]  
0048DA01  shr     ecx, 2  
0048DA04  rep     movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  
0048DA06  mov     ecx,eax  
0048DA08  and     ecx,3  
0048DA0B  rep     movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  ; 转移  
0048DA0D  lea     ecx, dword ptr \[esp+64\]  
0048DA11  push    ecx  
0048DA12  push    edx  
0048DA13  mov     ecx, ebp  
0048DA15  call    0048E700        ; 调用 48E700，对上面那个数值串进行变换  
0048DA1A  mov     dword ptr \[esp+2C\], eax  
0048DA1E  mov     edi, ebx        ; 变换后的 RegCode  
0048DA20  or      ecx, FFFFFFFF  
0048DA23  xor     eax, eax  
0048DA25  repne   scas byte ptr es:\[edi\]  
0048DA27  not     ecx  
0048DA29  dec     ecx  
0048DA2A  cmp     ecx, 8  
0048DA2D  jbe     0048DD03        ; 变换后的 RegCode 的长度若不大于 8 则转 48DD03  
0048DA33  lea     esi, dword ptr \[ebp+19C\]    ; 指向变换后的 RegCode 的第七字节  
0048DA39  lea     eax, dword ptr \[esp+64\]    ; 指向变换后的数值串  
0048DA3D  /mov     dl, byte ptr \[eax\]  
0048DA3F  |mov     cl, dl  
0048DA41  |cmp     dl, byte ptr \[esi\]  
0048DA43  |jnz     short 0048DA61  
0048DA45  |test    cl, cl  
0048DA47  |je      short 0048DA5D  
0048DA49  |mov     dl, byte ptr \[eax+1\]  
0048DA4C  |mov     cl, dl  
0048DA4E  |cmp     dl, byte ptr \[esi+1\]  
0048DA51  |jnz     short 0048DA61  
0048DA53  |add     eax, 2  
0048DA56  |add     esi, 2  
0048DA59  |test    cl, cl  
0048DA5B  \\jnz     short 0048DA3D  
0048DA5D  xor     eax, eax  
0048DA5F  jmp     short 0048DA66  
0048DA61  sbb     eax, eax  
0048DA63  sbb     eax, -1  
0048DA66  test    eax, eax  
0048DA68  jnz     0048DD03  

; 变换后的 RegCode 从第七字节始的部分若等于变换后的数值串则  
0048DA6E  mov     eax, dword ptr \[ebp+178\]  
0048DA74  mov     ecx, eax  
0048DA76  shr     ecx, 1  
0048DA78  and     ecx, 7  
0048DA7B  cmp     ecx, 5  
0048DA7E  jge     0048DC5D        ; 48DC5D 处会提醒您扩展天数超出了最大值  

; 48DA84 到 48DC5C 会提醒您将试用期扩展了若干天。这还不是我们所期望的。  

; ... ...  

0048DD03  lea     ecx, dword ptr \[esp+10\]  
0048DD07  mov     byte ptr \[esp+308\], 4  
0048DD0F  call    <jmp.&MFC42.#800\_CString::~CStri>  

; 48DD14 始的代码好像是在 5CB64C 始的黑名单中查找变换后的 SerialNo  
; 找到后就施行一些“破坏”活动  
0048DD14  mov     dword ptr \[esp+10\], 005CB64C      ; 黑名单表  
0048DD1C  /lea     edi, dword ptr \[esp+48\]  
0048DD20  |or      ecx, FFFFFFFF  
0048DD23  |xor     eax, eax  
0048DD25  |lea     esi, dword ptr \[esp+48\]  
0048DD29  |repne   scas byte ptr es:\[edi\]  
0048DD2B  |mov     edi, dword ptr \[esp+10\]  
0048DD2F  |not     ecx  
0048DD31  |dec     ecx  
0048DD32  |xor     eax, eax  
0048DD34  |repe    cmps byte ptr es:\[edi\], byte ptr \[esi\]  
0048DD36  |jnz     0048DDEB  
0048DD3C  |lea     edi, dword ptr \[esp+48\]  
0048DD40  |or      ecx, FFFFFFFF  
0048DD43  |repne   scas byte ptr es:\[edi\]  
0048DD45  |not     ecx  
0048DD47  |dec     ecx  
0048DD48  |mov     edi, 005CBFA0        ; 若找到就看一看是不是"vscnewscast"  
0048DD4D  |lea     esi, dword ptr \[esp+48\]  
0048DD51  |xor     edx, edx  
0048DD53  |repe    cmps byte ptr es:\[edi\], byte ptr \[esi\]  
0048DD55  |jnz     short 0048DD6C  

; 若找到的是"vscnewscast"就将 GetLocalTm 返回的小时值加进变换后的 RegCode 的第一个字节  
0048DD57  |push    edx  
0048DD58  |lea     ecx, dword ptr \[ebp+C4\]  
0048DD5E  |call    <jmp.&MFC42.#3337\_CTime::GetLocalTm>  
0048DD63  |mov     eax, dword ptr \[eax+8\]  
0048DD66  |mov     cl, byte ptr \[ebx\]  
0048DD68  |add     cl, al  
0048DD6A  |mov     byte ptr \[ebx\], cl  
0048DD6C  |lea     edi, dword ptr \[esp+48\]  
0048DD70  |or      ecx, FFFFFFFF  
0048DD73  |xor     eax, eax  
0048DD75  |lea     esi, dword ptr \[esp+48\]  
0048DD79  |repne   scas byte ptr es:\[edi\]  
0048DD7B  |not     ecx  
0048DD7D  |dec     ecx  
0048DD7E  |mov     edi, 005CB63C        ; vscnewscast  
0048DD83  |xor     eax, eax  
0048DD85  |repe    cmps byte ptr es:\[edi\], byte ptr \[esi\]  
0048DD87  |push    eax  
0048DD88  |jnz     short 0048DDA0  
0048DD8A  |lea     ecx, dword ptr \[ebp+C4\]  
0048DD90  |call    <jmp.&MFC42.#3337\_CTime::GetLocalTm>  
0048DD95  |mov     eax, dword ptr \[eax+8\]  
0048DD98  |mov     cl, byte ptr \[ebx\]  
0048DD9A  |add     cl, al  
0048DD9C  |mov     byte ptr \[ebx\], cl        ; 再加一次  
0048DD9E  |jmp     short 0048DDEB  

; 若找到的不是"vscnewscast"就将 GetLocalTm 返回的月份值加进变换后的 RegCode 的第一个字节  
0048DDA0  |lea     esi, dword ptr \[ebp+C4\]  
0048DDA6  |mov     ecx, esi  
0048DDA8  |call    <jmp.&MFC42.#3337\_CTime::GetLocalTm>  
0048DDAD  |mov     eax, dword ptr \[eax+10\]  
0048DDB0  |mov     cl, byte ptr \[ebx\]  
0048DDB2  |inc     eax  
0048DDB3  |push    0  
0048DDB5  |add     cl, al  
0048DDB7  |mov     byte ptr \[ebx\], cl  

; 将 GetLocalTm 返回的日期值加进变换后的 RegCode 的第二个字节  
0048DDB9  |mov     ecx, esi  
0048DDBB  |call    <jmp.&MFC42.#3337\_CTime::GetLocalTm>  
0048DDC0  |mov     eax, dword ptr \[eax+C\]  
0048DDC3  |mov     cl, byte ptr \[ebp+197\]  
0048DDC9  |add     cl, al  
0048DDCB  |push    0  
0048DDCD  |mov     byte ptr \[ebp+197\], cl  

; 将 GetLocalTm 返回的小时值加进变换后的 RegCode 的第三个字节  
0048DDD3  |mov     ecx, esi  
0048DDD5  |call    <jmp.&MFC42.#3337\_CTime::GetLocalTm>  
0048DDDA  |mov     eax, dword ptr \[eax+8\]  
0048DDDD  |mov     cl, byte ptr \[ebp+198\]  
0048DDE3  |add     cl, al  
0048DDE5  |mov     byte ptr \[ebp+198\], cl  
0048DDEB  |mov     eax, dword ptr \[esp+10\]  
0048DDEF  |add     eax, 0F  
0048DDF2  |cmp     eax, 005CBF61  
0048DDF7  |mov     dword ptr \[esp+10\], eax  
0048DDFB  \\jl      0048DD1C  

0048DE01  cmp     dword ptr \[esp+2C\], 5        ; SerialNo 长度小于 5 就报错  
0048DE06  jl      0048E471  

0048DE0C  lea     esi, dword ptr \[esp+48\]  
0048DE10  mov     eax, ebx  
0048DE12  /mov     dl, byte ptr \[eax\]  
0048DE14  |mov     cl, dl  
0048DE16  |cmp     dl, byte ptr \[esi\]  
0048DE18  |jnz     short 0048DE36  
0048DE1A  |test    cl, cl  
0048DE1C  |je      short 0048DE32  
0048DE1E  |mov     dl, byte ptr \[eax+1\]  
0048DE21  |mov     cl, dl  
0048DE23  |cmp     dl, byte ptr \[esi+1\]  
0048DE26  |jnz     short 0048DE36  
0048DE28  |add     eax, 2  
0048DE2B  |add     esi, 2  
0048DE2E  |test    cl, cl  
0048DE30  \\jnz     short 0048DE12  
0048DE32  xor     eax, eax  
0048DE34  jmp     short 0048DE3B  
0048DE36  sbb     eax, eax  
0048DE38  sbb     eax, -1  
0048DE3B  test    eax, eax  
0048DE3D  jnz     0048E471          ; 若变换后的 RegCode 不等于变换后的 SerialNo 就报错  

; 以下这段代码查看 SerialNo 的前几个字符  
; 若是 VS2、SLS2、RSS2、EDS2、RRS2 就转 48DFC7  
0048DE43  mov     edi, 005CC7A0          ; VS2  
0048DE48  or      ecx, FFFFFFFF  
0048DE4B  repne   scas byte ptr es:\[edi\]  
0048DE4D  not     ecx  
0048DE4F  lea     edx, dword ptr \[ebp+17C\]  
0048DE55  dec     ecx  
0048DE56  mov     edi, 005CC7A0          ; VS2  
0048DE5B  mov     esi, edx  
0048DE5D  xor     eax, eax  
0048DE5F  repe    cmps byte ptr es:\[edi\], byte ptr \[esi\]  
0048DE61  je      0048DFC7  
0048DE67  mov     edi, 005CCCD0          ; SLS2  
0048DE6C  or      ecx, FFFFFFFF  
0048DE6F  repne   scas byte ptr es:\[edi\]  
0048DE71  not     ecx  
```



```
> 0048DE73  dec     ecx  
> 0048DE74  mov     edi, 005CCCD0            ; SLS2  
> 0048DE79  mov     esi, edx  
> 0048DE7B  xor     eax, eax  
> 0048DE7D  repe    cmps byte ptr es:[edi], byte ptr [esi]  
> 0048DE7F  je      0048DFC7  
> 0048DE85  mov     edi, 005CCCC8             ; RSS2  
> 0048DE8A  or      ecx, FFFFFFFF  
> 0048DE8D  repne   scas byte ptr es:[edi]  
> 0048DE8F  not     ecx  
> 0048DE91  dec     ecx  
> 0048DE92  mov     edi, 005CCCC8             ; RSS2  
> 0048DE97  mov     esi, edx  
> 0048DE99  xor     eax, eax  
> 0048DE9B  repe    cmps byte ptr es:[edi], byte ptr [esi]  
> 0048DE9D  je      0048DFC7  
> 0048DEA3  mov     edi, 005CCCC0             ; EDS2  
> 0048DEA8  or      ecx, FFFFFFFF  
> 0048DEAB  repne   scas byte ptr es:[edi]  
> 0048DEAD  not     ecx  
> 0048DEAF  dec     ecx  
> 0048DEB0  mov     edi, 005CCCC0             ; EDS2  
> 0048DEB5  mov     esi, edx  
> 0048DEB7  xor     eax, eax  
> 0048DEB9  repe    cmps byte ptr es:[edi], byte ptr [esi]  
> 0048DEBB  je      0048DFC7  
> 0048DEC1  mov     edi, 005CCCB8             ; RRS2  
> 0048DEC6  or      ecx, FFFFFFFF  
> 0048DEC9  repne   scas byte ptr es:[edi]  
> 0048DECB  not     ecx  
> 0048DECD  dec     ecx  
> 0048DECE  mov     edi, 005CCCB8             ; RRS2  
> 0048DED3  mov     esi, edx  
> 0048DED5  xor     eax, eax  
> 0048DED7  repe    cmps byte ptr es:[edi], byte ptr [esi]  
> 0048DED9  je      0048DFC7  
> 0048DEDF  mov     edi, 005CCCB4             ; IWS  
> 0048DEE4  or      ecx, FFFFFFFF  
> 0048DEE7  repne   scas byte ptr es:[edi]  
> 0048DEE9  not     ecx  
> 0048DEEB  dec     ecx  
> 0048DEEC  mov     edi, 005CCCB4             ; IWS  
> 0048DEF1  mov     esi, edx  
> 0048DEF3  xor     eax, eax  
> 0048DEF5  repe    cmps byte ptr es:[edi], byte ptr [esi]  
> 0048DEF7  jnz     short 0048DF25            ; 不是"IWS"则转 48DF25  
> 0048DEF9  mov     edi, 005CCCB4             ; IWS  
> 0048DEFE  or      ecx, FFFFFFFF  
> 0048DF01  repne   scas byte ptr es:[edi]  
> 0048DF03  not     ecx  
> 0048DF05  dec     ecx  
> 0048DF06  lea     ecx, dword ptr [ecx+ebp+17C]  
> 0048DF0D  push    ecx  
> 0048DF0E  call    dword ptr [<&MSVCRT.atol>]     ; 是"IWS"则将后面的部分转成数值  
> 0048DF14  add     esp, 4  
> 0048DF17  cmp     eax, 493E0  
> 0048DF1C  jl      short 0048DF75  
> 0048DF1E  cmp     eax, 4953E  
> 0048DF23  jmp     short 0048DF6F            ; 转成的数值是从 0x493E0 到 0x4953E？  
> 0048DF25  mov     edi, 005CCCAC             ; BYWSP  
> 0048DF2A  or      ecx, FFFFFFFF  
> 0048DF2D  xor     eax, eax  
> 0048DF2F  mov     esi, edx  
> 0048DF31  repne   scas byte ptr es:[edi]  
> 0048DF33  not     ecx  
> 0048DF35  dec     ecx  
> 0048DF36  mov     edi, 005CCCAC             ; BYWSP  
> 0048DF3B  xor     edx, edx  
> 0048DF3D  repe    cmps byte ptr es:[edi], byte ptr [esi]  
> 0048DF3F  jnz     0048E0DA                  ; 这几个都不是则转 48E0DA  
> 0048DF45  mov     edi, 005CCCAC             ; BYWSP  
> 0048DF4A  or      ecx, FFFFFFFF  
> 0048DF4D  repne   scas byte ptr es:[edi]  
> 0048DF4F  not     ecx  
> 0048DF51  dec     ecx  
> 0048DF52  lea     eax, dword ptr [ecx+ebp+17C]  
> 0048DF59  push    eax  
> 0048DF5A  call    dword ptr [<&MSVCRT.atol>]     ; 是"BYWSP"则将后面的部分转成数值  
> 0048DF60  add     esp, 4  
> 0048DF63  cmp     eax, 831  
> 0048DF68  jl      short 0048DF75  
> 0048DF6A  cmp     eax, 84D                  ; 转成的数值是从 0x831 到 0x84D？  
> 0048DF6F  jle     0048E0D2                  ; 转成的数值在范围内则转 48E0D2  
> ; 若转成的数值不在范围则 48DF75 到 48DFC6 像上面那样进行“破坏”，然后转 48E0DA  
> ; ... ...  
> 0048DFC7  lea     esi, dword ptr [ebp+180]  
> 0048DFCD  push    esi  
> 0048DFCE  call    dword ptr [<&MSVCRT.atol>]     ; 将 SerialNo 第 5 位始的部分转成数值  
> 0048DFD4  add     esp, 4  
> 0048DFD7  test    eax, eax  
> 0048DFD9  jl      0048E085  
> 0048DFDF  cmp     eax, 004C4B40  
> 0048DFE4  jg      0048E085                  ; 若不是从 0 到 0x4C4B40 则转 48E085  
> 0048DFEA  mov     eax, dword ptr [esp+28]  
> 0048DFEE  test    eax, eax  
> 0048DFF0  jg      0048E0D2                  ; 若[esp+28]中的数值大于 0 则转 48E0D2  
> 0048DFF6  lea     edi, dword ptr [ebp+17C]  
> 0048DFFC  or      ecx, FFFFFFFF  
> 0048DFFF  xor     eax, eax  
> 0048E001  mov     dword ptr [esp+1C], 0  
> 0048E009  repne   scas byte ptr es:[edi]  
> 0048E00B  not     ecx  
> 0048E00D  dec     ecx  
> 0048E00E  cmp     ecx, 4  
> 0048E011  jle     0048E0D2                  ; 若 SerialNo 的长度不大于 4 则转 48E0D2  
> 0048E017  mov     edx, esi                  ; 现在开始计算 SerialNo 中的字母个数  
> 0048E019  mov     esi, -17C  
> 0048E01E  sub     esi, ebp  
> 0048E020  /mov    al, byte ptr [edx]  
> 0048E022  |cmp     al, 30  
> 0048E024  |jl      short 0048E02A  
> 0048E026  |cmp     al, 39  
> 0048E028  |jle     short 0048E02E  
> 0048E02A  |inc     dword ptr [esp+1C]  
> 0048E02E  |lea     edi, dword ptr [ebp+17C]  
> 0048E034  |or      ecx, FFFFFFFF  
> 0048E037  |xor     eax, eax  
> 0048E039  |inc     edx  
> 0048E03A  |repne   scas byte ptr es:[edi]  
> 0048E03C  |not     ecx  
> 0048E03E  |dec     ecx  
> 0048E03F  |lea     eax, dword ptr [esi+edx]  
> 0048E042  |cmp     eax, ecx  
> 0048E044  \jl     short 0048E020  
> 0048E046  mov     eax, dword ptr [esp+1C]  
> 0048E04A  test    eax, eax  
> 0048E04C  je      0048E0D2                  ; 若 SerialNo 中的字母个数为 0 则转 48E0D2  
> 0048E052  lea     esi, dword ptr [esp+48]  
> 0048E056  mov     eax, ebx  
> 0048E058  /mov    dl, byte ptr [eax]  
> 0048E05A  |mov    cl, dl  
> 0048E05C  |cmp    dl, byte ptr [esi]  
> 0048E05E  |jnz    short 0048E07C  
> 0048E060  |test   cl, cl  
> 0048E062  |je     short 0048E078  
> 0048E064  |mov    dl, byte ptr [eax+1]  
> 0048E067  |mov    cl, dl  
> 0048E069  |cmp    dl, byte ptr [esi+1]  
> 0048E06C  |jnz    short 0048E07C  
> 0048E06E  |add    eax, 2  
> 0048E071  |add    esi, 2  
> 0048E074  |test   cl, cl  
> 0048E076  \jnz    short 0048E058  
> 0048E078  xor     eax, eax  
> 0048E07A  jmp     short 0048E081  
> 0048E07C  sbb     eax, eax  
> 0048E07E  sbb     eax, -1  
> 0048E081  test    eax, eax  
> 0048E083  jnz     short 0048E0D2            ; 若此时变换后的 RegCode 不等于变换后的 SerialNo 则转 48E0D2  
> ; 48E085 到 48E0D1 搞“破坏”  
> ; ... ...  
> 0048E0D2  mov     dword ptr [esp+1C], 1  
> 0048E0DA  lea     esi, dword ptr [esp+48]  
> 0048E0DE  mov     eax, ebx  
> 0048E0E0  /mov    dl, byte ptr [eax]  
> 0048E0E2  |mov    bl, byte ptr [esi]  
> 0048E0E4  |mov    cl, dl  
> 0048E0E6  |cmp    dl, bl  
> 0048E0E8  |jnz    short 0048E108  
> 0048E0EA  |test   cl, cl  
> 0048E0EC  |je     short 0048E104  
> 0048E0EE  |mov    dl, byte ptr [eax+1]  
> 0048E0F1  |mov    bl, byte ptr [esi+1]  
> 0048E0F4  |mov    cl, dl  
> 0048E0F6  |cmp    dl, bl  
> 0048E0F8  |jnz    short 0048E108  
> 0048E0FA  |add    eax, 2  
> 0048E0FD  |add    esi, 2  
> 0048E100  |test   cl, cl  
> 0048E102  \jnz    short 0048E0E0  
> 0048E104  xor     eax, eax  
> 0048E106  jmp     short 0048E10D  
> 0048E108  sbb     eax, eax  
> 0048E10A  sbb     eax, -1  
> 0048E10D  test    eax, eax  
> 0048E10F  jnz     0048E395                  ; 若此时变换后的 RegCode 不等于变换后的 SerialNo 则报错  
> 0048E115  mov     eax, dword ptr [esp+1C]  
> 0048E119  test    eax, eax  
> 0048E11B  je      0048E395                  ; 若此时 SerialNo 中的字母个数为 0 则报错  
> 0048E121  mov     eax, dword ptr [esp+14]  
> 0048E125  cmp     byte ptr [eax], 0         ; 注册名为空？  
> 0048E128  jnz     0048E1B4  
> ; 48E12E 到 48E1B3 提醒您输入注册名后退出  
> ; ... ...  
> 0048E1B4  lea     edx, dword ptr [ebp+178]  
> 0048E1BA  push    0  
> 0048E1BC  push    edx  
> 0048E1BD  mov     ecx, ebp  
> 0048E1BF  mov     byte ptr [ebp+39C], 1  
> 0048E1C6  call    005608A0  
> 0048E1CB  lea     eax, dword ptr [ebp+1B0]  
```



```
0048E1D1  lea     ecx, dword ptr [esp+3C]  
0048E1D5  push    eax  
0048E1D6  call    <jmp.&MFC42.#537_CString::CString>  
0048E1DB  mov     esi, eax  
0048E1DD  lea     eax, dword ptr [ebp+17C]  
0048E1E3  lea     ecx, dword ptr [esp+38]  
0048E1E7  push    eax  
0048E1E8  mov     byte ptr [esp+30C], 13  
0048E1F0  call    <jmp.&MFC42.#537_CString::CString>  
0048E1F5  mov     edi, eax  
0048E1F7  push    005CCC9C          ; Serial Number:  
0048E1FC  lea     eax, dword ptr [esp+14]  
0048E200  push    006A5894          ; 提醒您保存注册信息的 CString 对象  
0048E205  push    eax  
0048E206  mov     byte ptr [esp+314], 14  
0048E20E  call    <jmp.&MFC42.#924_operator+>  
0048E213  push    edi  
0048E214  lea     ecx, dword ptr [esp+20]  
0048E218  push    eax  
0048E219  push    ecx  
0048E21A  mov     byte ptr [esp+314], 15  
0048E222  call    <jmp.&MFC42.#922_operator+>  
0048E227  push    005CCC88          ;  ASCII 0A,"Registration:  "  
0048E22C  lea     edx, dword ptr [esp+30]  
0048E230  push    eax  
0048E231  push    edx  
0048E232  mov     byte ptr [esp+314], 16  
0048E23A  call    <jmp.&MFC42.#924_operator+>  
0048E23F  push    esi  
0048E240  push    eax  
0048E241  lea     eax, dword ptr [esp+30]  
0048E245  mov     byte ptr [esp+310], 17  
0048E24D  push    eax  
0048E24E  call    <jmp.&MFC42.#922_operator+>  
0048E253  lea     ecx, dword ptr [esp+2C]  
0048E257  mov     byte ptr [esp+308], 1D  
0048E25F  call    <jmp.&MFC42.#800_CString::~CString>  
0048E264  lea     ecx, dword ptr [esp+1C]  
0048E268  mov     byte ptr [esp+308], 1C  
0048E270  call    <jmp.&MFC42.#800_CString::~CString>  
0048E275  lea     ecx, dword ptr [esp+10]  
0048E279  mov     byte ptr [esp+308], 1B  
0048E281  call    <jmp.&MFC42.#800_CString::~CString>  
0048E286  lea     ecx, dword ptr [esp+38]  
0048E28A  mov     byte ptr [esp+308], 1A  
0048E292  call    <jmp.&MFC42.#800_CString::~CString>  
0048E297  lea     ecx, dword ptr [esp+3C]  
0048E29B  mov     byte ptr [esp+308], 19  
0048E2A3  call    <jmp.&MFC42.#800_CString::~CString>  
0048E2A8  mov     ecx, dword ptr [esp+28]  
0048E2AC  push    0  
0048E2AE  push    0  
0048E2B0  push    ecx  
0048E2B1  call    <jmp.&MFC42.#1200_AfxMessageBox>  
0048E2B6  mov     edx, dword ptr [esp+310]  
0048E2BD  push    0  
0048E2BF  push    edx  
0048E2C0  mov     ecx, ebp  
0048E2C2  call    0048E8B0  
0048E2C7  mov     esi, 1  
0048E2CC  mov     byte ptr [esp+308], 4  
0048E2D4  mov     dword ptr [esp+10], esi  
0048E2D8  lea     ecx, dword ptr [esp+28]  
0048E2DC  call    <jmp.&MFC42.#800_CString::~CString>  
0048E2E1  mov     dword ptr [6A5850], esi  
0048E2E7  mov     al, byte ptr [6A5928]  
0048E2EC  test    al, al  
0048E2EE  je      0048E5FD          ; [6A5928]=0 则正常退出  
0048E2F4  mov     eax, dword ptr [esp+14]  
0048E2F8  cmp     byte ptr [eax], 0  
0048E2FB  jnz     0048E54D  
0048E301  mov     al, byte ptr [ebp+39C]  
0048E307  test    al, al  
0048E309  je      0048E54D  
```

```
; 48E30F 到 48E394 提醒输入注册名后退出
; ... ...
; 48E395 到 48E470、48E471 到 48E54C 提醒您注册信息无效后退出
; ... ...
```

```
0048E54D  cmp     byte ptr ss:[ebp+39C],1
0048E554  je      SITESPIN.0048E5FD
0048E55A  mov     eax,dword ptr ss:[esp+30]
0048E55E  test    eax,eax
0048E560  jnz     SITESPIN.0048E5FD
```

```
; 48E566 到 48E5FC 提醒您试用期已过后退出
; ... ...
```

```
0048E5FD  mov     eax, dword ptr [esp+10]
0048E601  xor     edx, edx
0048E603  test    eax, eax
0048E605  setge   dl
0048E608  mov     esi, edx
0048E60A  lea     ecx, dword ptr [esp+18]
0048E60E  mov     byte ptr [esp+308], 3
0048E616  call    <jmp.&MFC42.#800_CString::~CString>
0048E61B  lea     ecx, dword ptr [esp+9C]
0048E622  mov     byte ptr [esp+308], 2
0048E62A  call    00493430
0048E62F  lea     ecx, dword ptr [esp+20]
0048E633  mov     byte ptr [esp+308], 0
0048E63B  call    <jmp.&MFC42.#800_CString::~CString>
0048E640  lea     ecx, dword ptr [esp+24]
0048E644  mov     dword ptr [esp+308], -1
0048E64F  call    <jmp.&MFC42.#800_CString::~CString>
0048E654  mov     eax, esi
0048E656  mov     ecx, dword ptr [esp+300]
0048E65D  pop     edi
0048E65E  pop     esi
0048E65F  pop     ebp
0048E660  mov     dword ptr fs:[0], ecx
0048E667  pop     ebx
0048E668  add     esp, 2FC
0048E66E  retn    4
```

```
005CB4F8  07 08 02 0F 06 07 09 0C 04 0A 03 03 09 04 05 06 07 02 00 08 06 11 0D 02
```

通过前面的分析我们可以得出以下结论：

1. 有效的注册码不能是`EXTEND`、`RESET`，甚至其第七位开始的部分不能等同于默认序列号的数字部分；
2. 按照代码分析中的变换算法，对`Serial Number`和`Registration Code`分别进行变换的结果应该相同；
3. 在满足条件 2 的前提下，若`Serial Number`的前几个字符是`VS2`、`SLS2`、`RSS2`、`EDS2`、`RRS2`，则其第 5 位始的数值应从 0 到 5000000；若`Serial Number`的前几个字符是`IWS`，则其后面的数值应从 300000 到 300350；若`Serial Number`的前几个字符是`BYWSP`，则其后面的数值应从 2097 到 2125。

通过跟踪发现，程序会在注册表的`HKEY_LOCAL_MACHINE\Software\CLASSES`下创建一个名为`Ssiomv2`的主键来记录注册信息。

设计注册机的时候，关键是`Registration Code`的变换算法的理解。我设计的注册机用起来很好。

`4dd466d0e1e0cc123afbe504da2462ca`
