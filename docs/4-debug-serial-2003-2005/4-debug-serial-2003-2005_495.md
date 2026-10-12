
# 搜易无需 SMTP 群发器 V3.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50441.htm`](https://www.pediy.com/kssd/pediy05/pediy50441.htm)

简单算法——搜易无需 SMTP 群发器 V3.0 (`EmailSender`)

下载页面：[`www.skycn.com/soft/5806.html`](http://www.skycn.com/soft/5806.html)

软件大小：613 KB

软件语言：简体中文

软件类别：国产软件 / 共享版 / 邮件群发

应用平台：`Win9x/NT/2000/XP`

加入时间：2002-12-04 14:24:51

下载次数：8106

推荐等级：\*\*\*

开发商：[`mailsender.yeah.net/`](http://mailsender.yeah.net/)

【软件简介】：国内一流首创专业服务器型无需 SMTP 多线程飞快邮件集群发送系统。软件功能如下：1.普通电脑自建 SMTP 发送服务器，无需再为电信局/ISP 封杀发信帐号烦恼；2.高品质多线程工作模式，验证发信速度最快可达每小时 50000 封电邮；3.手动/自动保存进程，可中断后继续发送，可晚上无人值守，适宜包月上网；4.单发/群发可选，轻松编撰文本/网页格式信体，支持不限数量附件发送；5.语音操作唤醒提示/自动挂断网络，可于系统托盘处后台工作，省时省钱；6.对每一邮址特快发信，自动分离发送成功/失败电邮，轻松筛选邮件列表库；7.国内多家知名电子商务公司选用，是推广网站和开展网上广告的绝佳工具。

【软件限制】：功能限制、次数限制

【作者声明】：小弟初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请各大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`W32Dasm8.93` 黄金版

—————————————————————————————

【过程】：

和 `EmailSearcher` V3.0 的算法是一样的，真是一家的。

序列号：147109425

试炼码：13572468

切入 `TRW`，`BPX 4D21B8`，`F5` 返回，点“确定”，拦下！

—————————————————————————————

```
:004D21B8 E8036AF3FF        call 00408BC0
:004D21BD 8B45D4            mov eax, dword ptr [ebp-2C]
:004D21C0 E81B6CF3FF        call 00408DE0
:004D21C5 8945F0            mov dword ptr [ebp-10], eax
:004D21C8 8955F4            mov dword ptr [ebp-0C], edx
:004D21CB 6A00              push 00000000
:004D21CD 6A18              push 00000018
                          ====注意：18
:004D21CF 8B45F0            mov eax, dword ptr [ebp-10]
                          ==》？EAX=CF1974
                          CF1974 是我们输入的数字 13572468 的 16 进制值！
:004D21D2 8B55F4            mov edx, dword ptr [ebp-0C]
:004D21D5 E84C46F3FF        call 00406826
                          ====关键 CALL！F8 进入！
```

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

`F8` 进入 `4D21D5` `call 00406826`

\* `Referenced by a CALL at Addresses:`

|:004068C6 , :004D21D5

|

```
:00406826 55                push ebp
:00406827 53                push ebx
:00406828 56                push esi
:00406829 57                push edi
:0040682A 33FF              xor edi, edi
:0040682C 8B5C2414          mov ebx, dword ptr [esp+14]
                          ====》18 入 EAX
:00406830 8B4C2418          mov ecx, dword ptr [esp+18]
:00406834 0BC9              or ecx, ecx
:00406836 7508              jne 00406840
:00406838 0BD2              or edx, edx
:0040683A 745C              je 00406898
:0040683C 0BDB              or ebx, ebx
:0040683E 7458              je 00406898
…………省略 …………
:00406898 F7F3              div ebx
                          ====》EAX=CF1974/18=8A10F
:0040689A 33D2              xor edx, edx
:0040689C EBF3              jmp 00406891
```

&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&

```
:004D21DA 8945F0            mov dword ptr [ebp-10], eax
:004D21DD 8955F4            mov dword ptr [ebp-0C], edx
:004D21E0 8B45F0            mov eax, dword ptr [ebp-10]
                          ====》EAX=8A10F
:004D21E3 8B55F4            mov edx, dword ptr [ebp-0C]
:004D21E6 2D84410100        sub eax, 00014184
                          ====》EAX=8A10F-14184=75F8B
:004D21EB 83DA00            sbb edx, 00000000
:004D21EE 8945F0            mov dword ptr [ebp-10], eax
                          ====》75F8B 入 [ebp-10]
:004D21F1 8955F4            mov dword ptr [ebp-0C], edx
:004D21F4 8D45D8            lea eax, dword ptr [ebp-28]
:004D21F7 E840ACFEFF        call 004BCE3C
                          ====》F8 进入！
```

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

`F8` 进入 `4D21F7` `call 004BCE3C`

```
:004BCE3C 53                push ebx
:004BCE3D 57                push edi
:004BCE3E 89C7              mov edi, eax
:004BCE40 B801000000        mov eax, 00000001
:004BCE45 0FA2              cpuid
                          ====》？EAX=544
:004BCE47 AB                stosd
:004BCE48 89D8              mov eax, ebx
:004BCE4A AB                stosd
:004BCE4B 89C8              mov eax, ecx
:004BCE4D AB                stosd
:004BCE4E 89D0              mov eax, edx
:004BCE50 AB                stosd
:004BCE51 5F                pop edi
:004BCE52 5B                pop ebx
:004BCE53 C3                ret
```

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

```
:004D21FC 8B45D8            mov eax, dword ptr [ebp-28]
                          ====》？EAX=544
:004D21FF 99                cdq
:004D2200 8945E8            mov dword ptr [ebp-18], eax
                          ====》544 入[ebp-18]
:004D2203 8955EC            mov dword ptr [ebp-14], edx
:004D2206 8B45F0            mov eax, dword ptr [ebp-10]
                          ====》？EAX=75F8B
:004D2209 8B55F4            mov edx, dword ptr [ebp-0C]
:004D220C 3B55EC            cmp edx, dword ptr [ebp-14]
:004D220F 754B              jne 004D225C
:004D2211 3B45E8            cmp eax, dword ptr [ebp-18]
                          ====》比较了！
                          ====》EAX=75F8B
                          ====》[ebp-18]=544
:004D2214 7546              jne 004D225C
                          ====》跳则 OVER！
:004D2216 6A00              push 00000000
:004D2218 668B0DE4224D00    mov cx, word ptr [004D22E4]
:004D221F B202              mov dl, 02
```

\* `Possible StringData Ref from Code Obj` ->"软件登记注册成功!"

|

```
:004D2221 B8F0224D00        mov eax, 004D22F0
:004D2226 E8E549F8FF        call 00456C10
:004D222B 8B8380050000      mov eax, dword ptr [ebx+00000580]
:004D2231 E84A97FDFF        call 004AB980
:004D2236 33D2              xor edx, edx
:004D2238 8B8300030000      mov eax, dword ptr [ebx+00000300]
:004D223E E8E1DAF6FF        call 0043FD24
:004D2243 A1D4A24E00        mov eax, dword ptr [004EA2D4]
:004D2248 8B00              mov eax, dword ptr [eax]
:004D224A 8B80E0020000      mov eax, dword ptr [eax+000002E0]
```

\* `Possible StringData Ref from Code Obj` ->"已注册版本"

|

```
:004D2250 BA0C234D00        mov edx, 004D230C
:004D2255 E876D3F5FF        call 0042F5D0
:004D225A EB15              jmp 004D2271
```

\* `Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

|:004D220F(C), :004D2214(C)

|

```
:004D225C 6A00              push 00000000
:004D225E 668B0DE4224D00    mov cx, word ptr [004D22E4]
:004D2265 B201              mov dl, 01
```

\* `Possible StringData Ref from Code Obj` ->"软件注册号错误!"

|

```
:004D2267 B820234D00        mov eax, 004D2320
```

—————————————————————————————

【总结】：

`真码=(544+14184)*18=1EA2C0`（H）=`2007744`

—————————————————————————————

【注册信息保存】：

`[HKEY_LOCAL_MACHINE\Software\CLASSES\CLSID\{1AE69D60-73D0-11D4-BD52-38A480C50000}]`

`"613738517"="16"`

`"238548705"="238548705"`

—————————————————————————————

`Cracked By` 巢水工作坊——`fly`【`OCN`】

2003-2-13 18:30  `4dd466d0e1e0cc123afbe504da2462ca`
