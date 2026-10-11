
# 热血江湖辅助外挂

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-177.htm`](https://www.pediy.com/kssd/pediy08/pediy8-177.htm)

## 文章信息

*   **【文章标题】**: 热血江湖辅助外挂破解记录(菜鸟的第一篇破文,可能不太完整,请各位多多包涵)
*   **【文章作者】**: `hczcyy`
*   **【作者邮箱】**: `hczcyy@163.com`
*   **【作者 QQ 号】**: `55346577`
*   **【软件名称】**: 热血江湖辅助外挂
*   **【软件大小】**: `36.0k`
*   **【下载地址】**: 自己搜索下载
*   **【加壳方式】**: 无
*   **【保护方式】**: 机器码+注册码
*   **【编写语言】**: `Microsoft Visual C++ 6.0`
*   **【使用工具】**: `OD`, `PEID`
*   **【软件介绍】**: 热血江湖辅助外挂
*   **【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

注册好长时间了,一直不敢露面.原因无他,因这里的牛人太多.

闲话少叙,我们切入正题.

该软件下载后只有 `36k`,运行后发现是机器码+注册码的保护方式,随便输入点注册,弹出错误提示(简单的保护方式,暗喜..我等菜鸟最喜欢的就是这种保护方式不知道算法如何...)

用 `PEID` 查,无壳(继续暗喜中....).

拿出我们的利器 `od` 载入,查找字符 `"序列号错误,请输入序列号(嘿嘿,作者这话不太通顺)"`,双击来到这里:

```assembly
004010B3    55               push ebp
004010B4    56               push esi
004010B5    8BF1             mov esi,ecx
004010B7    6A 01            push 1
004010B9    897424 0C        mov dword ptr ss:[esp+C],esi
004010BD    E8 CA270000      call <jmp.&MFC42.#6334_CWnd::Upd>
004010C2    8B46 64          mov eax,dword ptr ds:[esi+64]
004010C5    8D6E 64          lea ebp,dword ptr ds:[esi+64]
004010C8    50               push eax
004010C9    E8 A2020000      call 江湖助手.00401370   //关键 call
004010CE    85C0             test eax,eax    
004010D0    0F84 8D010000    je 江湖助手.00401263       //关键跳转,爆破 84 ==>90  
004010D6    68 88664000      push 江湖助手.00406688
004010DB    68 D4604000      push 江湖助手.004060D4           ; ASCII "Software\\Microsoft\\Jhzs"
004010E0    68 02000080      push 80000002
004010E5    FF15 00404000    call dword ptr ds:[<&ADVAPI32.Re>; ADVAPI32.RegCreateKeyA
004010EB    8D4C24 10        lea ecx,dword ptr ss:[esp+10]
004010EF    8D5424 0C        lea edx,dword ptr ss:[esp+C]
004010F3    51               push ecx
004010F4    8B0D 88664000    mov ecx,dword ptr ds:[406688]
004010FA    8D4424 18        lea eax,dword ptr ss:[esp+18]
004010FE    52               push edx
004010FF    50               push eax
00401100    6A 00            push 0
00401102    68 CC604000      push 江湖助手.004060CC           ; ASCII "snzc"
00401107    51               push ecx
00401108    FF15 04404000    call dword ptr ds:[<&ADVAPI32.Re>; ADVAPI32.RegQueryValueExA
0040110E    8B45 00          mov eax,dword ptr ss:[ebp]
00401111    8B4C24 0C        mov ecx,dword ptr ss:[esp+C]
00401115    3BC8             cmp ecx,eax
00401117    0F84 1D010000    je 江湖助手.0040123A
0040111D    53               push ebx
0040111E    57               push edi
0040111F    6A 0A            push 0A
00401121    68 8C664000      push 江湖助手.0040668C
00401126    50               push eax
00401127    FF15 50424000    call dword ptr ds:[<&MSVCRT._ito>; msvcrt._itoa
0040112D    8B35 44404000    mov esi,dword ptr ds:[<&KERNEL32>; kernel32.WritePrivateProfileStringA
00401133    83C4 0C          add esp,0C
00401136    68 C0604000      push 江湖助手.004060C0           ; ASCII "winzs.dat"
0040113B    68 8C664000      push 江湖助手.0040668C
00401140    68 B0604000      push 江湖助手.004060B0           ; ASCII "RegisterInfo"
00401145    68 A4604000      push 江湖助手.004060A4           ; ASCII "Settings"
0040114A    FFD6             call esi
0040114C    68 C0604000      push 江湖助手.004060C0           ; ASCII "winzs.dat"
00401151    68 D4674000      push 江湖助手.004067D4           ; ASCII "20060513"
00401156    68 9C604000      push 江湖助手.0040609C           ; ASCII "Date"
0040115B    68 94604000      push 江湖助手.00406094           ; ASCII "RXJH"
00401160    FFD6             call esi
00401162    BF CC604000      mov edi,江湖助手.004060CC        ; ASCII "snzc"
00401167    83C9 FF          or ecx,FFFFFFFF
0040116A    33C0             xor eax,eax
0040116C    8B15 88664000    mov edx,dword ptr ds:[406688]
00401172    F2:AE            repne scas byte ptr es:[edi]
00401174    F7D1             not ecx
00401176    8B1D 08404000    mov ebx,dword ptr ds:[<&ADVAPI32>; ADVAPI32.RegSetValueA
0040117C    49               dec ecx
0040117D    51               push ecx
0040117E    68 CC604000      push 江湖助手.004060CC           ; ASCII "snzc"
00401183    6A 01            push 1
00401185    50               push eax
00401186    52               push edx
00401187    FFD3             call ebx
00401189    A1 88664000      mov eax,dword ptr ds:[406688]
0040118E    6A 04            push 4
00401190    55               push ebp
00401191    8B2D 0C404000    mov ebp,dword ptr ds:[<&ADVAPI32>; ADVAPI32.RegSetValueExA
00401197    6A 04            push 4
00401199    6A 00            push 0
0040119B    68 CC604000      push 江湖助手.004060CC           ; ASCII "snzc"
004011A0    50               push eax
004011A1    FFD5             call ebp
004011A3    68 C0604000      push 江湖助手.004060C0           ; winzs.dat
004011A8    6A 00            push 0
004011AA    68 8C604000      push 江湖助手.0040608C           ; time
004011AF    68 94604000      push 江湖助手.00406094           ; rxjh
004011B4    FF15 40404000    call dword ptr ds:[<&KERNEL32.Ge>; kernel32.GetPrivateProfileIntA
004011BA    3D D0070000      cmp eax,7D0
004011BF    A3 78684000      mov dword ptr ds:[406878],eax
004011C4    7F 5A            jg short 江湖助手.00401220
004011C6    68 C0604000      push 江湖助手.004060C0           ; winzs.dat
004011CB    68 84604000      push 江湖助手.00406084           ; 30000
004011D0    68 8C604000      push 江湖助手.0040608C           ; time
004011D5    68 94604000      push 江湖助手.00406094           ; rxjh
004011DA    C705 68684000 3>mov dword ptr ds:[406868],7530
004011E4    FFD6             call esi
004011E6    BF 7C604000      mov edi,江湖助手.0040607C        ; winzc
004011EB    83C9 FF          or ecx,FFFFFFFF
004011EE    33C0             xor eax,eax
004011F0    F2:AE            repne scas byte ptr es:[edi]
004011F2    F7D1             not ecx
004011F4    49               dec ecx
004011F5    51               push ecx
004011F6    8B0D 88664000    mov ecx,dword ptr ds:[406688]
004011FC    68 7C604000      push 江湖助手.0040607C           ; winzc
00401201    6A 01            push 1
00401203    50               push eax
00401204    51               push ecx
00401205    FFD3             call ebx
00401207    8B15 88664000    mov edx,dword ptr ds:[406688]
0040120D    6A 04            push 4
0040120F    68 68684000      push 江湖助手.00406868
00401214    6A 04            push 4
00401216    6A 00            push 0
00401218    68 7C604000      push 江湖助手.0040607C           ; winzc
0040121D    52               push edx
0040121E    FFD5             call ebp
00401220    8B4C24 10        mov ecx,dword ptr ss:[esp+10]
00401224    6A 00            push 0
00401226    6A 00            push 0
00401228    68 54604000      push 江湖助手.00406054           ; 序列号正确，欢迎您的使用,请重新运行程序
0040122D    E8 54260000      call <jmp.&MFC42.#4224_CWnd::Mes>
00401232    8B7424 10        mov esi,dword ptr ss:[esp+10]
00401236    5F               pop edi
00401237    5B               pop ebx
00401238    EB 10            jmp short 江湖助手.0040124A
0040123A    6A 00            push 0
0040123C    6A 00            push 0
0040123E    68 3C604000      push 江湖助手.0040603C           ; 请您重新购买使用时间！
00401243    8BCE             mov ecx,esi
00401245    E8 3C260000      call <jmp.&MFC42.#4224_CWnd::Mes>
0040124A    A1 88664000      mov eax,dword ptr ds:[406688]
0040124F    50               push eax
00401250    FF15 10404000    call dword ptr ds:[<&ADVAPI32.Re>; ADVAPI32.RegCloseKey
```
