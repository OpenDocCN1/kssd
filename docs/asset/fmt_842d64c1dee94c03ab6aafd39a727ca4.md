
# 超级小精灵 Ver 1.00

> 原文：`www.pediy.com/kssd/pediy03/forum618.htm` ([Link](https://www.pediy.com/kssd/pediy03/forum618.htm))

《超级小精灵》Ver 1.00 共享版的注册分析

## 软件简介
超级小精灵主要是一个时间处理器，它主要具有如下功能：

1. 整点报时，您可以选择声音提醒，也可以选用文字提醒。也可两种方式都选用。
2. 定时提醒，可以按照每天、每周、任何日期三种方式设置。提醒界面比较漂亮。
3. 定时执行任务，如指定时间播放音乐，运行程序，弹出文本信息等，可以指定执行任务时的状态，如最大化、最小化。
4. 自带一个很酷的数字表，有了它，您赶快把 WIN98 带的表扔掉吧，这个表可以随心所欲地选择您喜爱的颜色。使用方法：在主控制面板上点击‘显示时钟’选择框，即可弹出时钟，用右键点击表界面，还可弹出表的控制菜单，执行其中的选项即可。
5. 保护 WINDOWS 系统。计划设置：对任何您指定的子目录进行加密，保护您的私人信息，没有密码是无法打开的。

## 作者信息
- **作者地址**：辽宁.鞍山钢铁学院计 97.1 班
- **作者姓名**：钟四化
- **邮政编码**：114002
- **电子信箱**：zhongsihua@yeah.net
- **注册费用**：人民币 10 元。（感觉此程序不错的，请向作者注册）
- **软件下载**：`http://newhua.infosail.com/down/superspirit.exe`

## 注册分析信息
- **分析人员**：李海涛
- **分析工具**：
  - `W32Dasm Ver:8.93`
  - `TRW2000 Ver:1.23`
- **信箱地址**：lihaitao@xaonline.com

## 分析过程

一、先用 `WD32ASM8.93` 超级中文版进行反汇编，然后查找，会看到：

```assembly
:00406150 6689442438                mov word ptr [esp+38], ax
:00406155 894C2414                  mov dword ptr [esp+14], ecx
:00406159 0F84DF010000              je 0040633E                      ; 检验你输入的注册名和注册码是否符合要求
:0040615F 8B6B64                    mov ebp, dword ptr [ebx+64]
:00406162 394DF8                    cmp dword ptr [ebp-08], ecx
:00406165 0F84D3010000              je 0040633E                      ; 程序从这跳（或是从上面跳）！！！下去看看吧！！！
:0040616B 83FA06                    cmp edx, 00000006
:0040616E 7F1A                      jg 0040618A                      ; 程序到这不跳！！！笨！！！
:00406170 51                        push ecx

* Possible StringData Ref from Data Obj ->" 错误提示 "
                                |
:00406171 68C4474100                push 004147C4

* Possible StringData Ref from Data Obj ->" 用户名不合法  "
                                |
:00406176 68B0474100                push 004147B0
:0040617B 8BCB                      mov ecx, ebx

* Reference To: MFC42.Ordinal:1080, Ord:1080h
                                |
:0040617D E8F4690000                Call 0040CB76
:00406182 5F                        pop edi
:00406183 5E                        pop esi
:00406184 5D                        pop ebp
:00406185 5B                        pop ebx
:00406186 83C42C                    add esp, 0000002C
:00406189 C3                        ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040616E(C)
|
:0040618A 33C0                      xor eax, eax
:0040618C 8D7AFE                    lea edi, dword ptr [edx-02]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004061A6(C)
|
:0040618F 8A140E                    mov dl, byte ptr [esi+ecx]
:00406192 2254042C                  and dl, byte ptr [esp+eax+2C]
:00406196 3BCF                      cmp ecx, edi
:00406198 8854041C                  mov byte ptr [esp+eax+1C], dl
:0040619C 7E03                      jle 004061A1
:0040619E 49                        dec ecx
:0040619F EB01                      jmp 004061A2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040619C(C)
|
:004061A1 41                        inc ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040619F(U)
|
:004061A2 40                        inc eax
:004061A3 83F80F                    cmp eax, 0000000F
:004061A6 7CE7                      jl 0040618F
:004061A8 C644242A00                mov [esp+2A], 00
:004061AD 33D2                      xor edx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004061F7(C)
|
:004061AF 8A4C141C                  mov cl, byte ptr [esp+edx+1C]
:004061B3 0FBEC1                    movsx eax, cl
:004061B6 83E830                    sub eax, 00000030
:004061B9 790B                      jns 004061C6
:004061BB 8A4C141D                  mov cl, byte ptr [esp+edx+1D]
:004061BF 2AC8                      sub cl, al
:004061C1 80C102                    add cl, 02
:004061C4 EB29                      jmp 004061EF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004061B9(C)
|
:004061C6 83F84A                    cmp eax, 0000004A
:004061C9 7E08                      jle 004061D3
:004061CB 8A4C141D                  mov cl, byte ptr [esp+edx+1D]
:004061CF 2AC8                      sub cl, al
:004061D1 EB1C                      jmp 004061EF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004061C9(C)
|
:004061D3 83F80A                    cmp eax, 0000000A
:004061D6 7C0A                      jl 004061E2
:004061D8 83F810                    cmp eax, 00000010
:004061DB 7F05                      jg 004061E2
:004061DD 80C10A                    add cl, 0A
:004061E0 EB0D                      jmp 004061EF

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004061D6(C), :004061DB(C)
|
:004061E2 83F82B                    cmp eax, 0000002B
:004061E5 7C0C                      jl 004061F3
:004061E7 83F830                    cmp eax, 00000030
:004061EA 7F07                      jg 004061F3
:004061EC 80C114                    add cl, 14

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004061C4(U), :004061D1(U), :004061E0(U)
|
:004061EF 884C141C                  mov byte ptr [esp+edx+1C], cl

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004061E5(C), :004061EA(C)
|
:004061F3 42                        inc edx
:004061F4 83FA0E                    cmp edx, 0000000E
:004061F7 7CB6                      jl 004061AF
:004061F9 33C0                      xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406210(C)
|
:004061FB 0FBE1428                  movsx edx, byte ptr [eax+ebp]   ; D EAX+EBP=你输入的注册码
:004061FF 0FBE4C041C                movsx ecx, byte ptr [esp+eax+1C] ; D ESP+EAX+1C=正确的注册码
:00406204 2BD1                      sub edx, ecx                   ; 依次对比注册码
:00406206 0F850E010000              jne 0040631A                   ; 此处一跳，死！！！
      改：909090909090             （暴力）
:0040620C 40                        inc eax
:0040620D 83F80E                    cmp eax, 0000000E              ; 对比 14 位注册码
:00406210 7CE9                      jl 004061FB

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040631D(C)
|

* Reference To: ADVAPI32.RegOpenKeyExA, Ord:0172h
                                |
:00406212 8B350CF04000              mov esi, dword ptr [0040F00C]
:00406218 8D542410                  lea edx, dword ptr [esp+10]
:0040621C 52                        push edx
:0040621D 6806000200                push 00020006
:00406222 6A00                      push 00000000

* Possible StringData Ref from Data Obj ->"Software\\Microsoft\\Windows"
                                |
:00406224 6894474100                push 00414794
:00406229 6801000080                push 80000001
:0040622E FFD6                      call esi
:00406230 85C0                      test eax, eax
:00406232 7410                      je 00406244
:00406234 6A00                      push 00000000
:00406236 6A00                      push 00000000

* Possible StringData Ref from Data Obj ->" 保存数据失败!  "
                                |
:00406238 68EC404100                push 004140EC
:0040623D 8BCB                      mov ecx, ebx

* Reference To: MFC42.Ordinal:1080, Ord:1080h
                                |
:0040623F E832690000                Call 0040CB76

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406232(C)
|
:00406244 8D442418                  lea eax, dword ptr [esp+18]
:00406248 50                        push eax
:00406249 6806000200                push 00020006
:0040624E 6A00                      push 00000000
```
