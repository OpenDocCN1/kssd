
## 反汇编片段

```
:00401172 90                      nop <-跳的位置有点错位:)
:00401173 A130214000              mov eax, dword ptr [00402130] <-把相应内存中的值取出来
:00401178 85C0                    test eax, eax <-看是否为0
:0040117A 7590                    jne 0040110C <-如果已经有值就返回
:0040117C FF153F604000            call dword ptr [0040603F]     <-这是函数`GetTickCount`,随机数种子
:00401182 B923000000              mov ecx, 00000023 <-随机数的计算公式：
:00401187 F7E1                    mul ecx   Ran_Num=[(Ran_Seed*X)+Y] mod Z
:00401189 83C007                  add eax, 00000007   其中X，Y为素数，Z为范围
:0040118C B964000000              mov ecx, 00000064
:00401191 F7F1                    div ecx
:00401193 42                      inc edx <-上面是计算1-100的随机数的过程
:00401194 891530214000            mov dword ptr [00402130], edx <-把结果放好
:0040119A 6A00                    push 00000000
:0040119C 52                      push edx <-要显示的结果
* Possible Reference to Dialog: GAME, CONTROL_ID:03F0, ""
                                  |
:0040119D 68F0030000              push 000003F0 <-按钮对应的文本框的ID,用资源工具可查
:004011A2 FF7508                  push [ebp+08] <-对话框的句柄
:004011A5 FF151B604000            call dword ptr [0040601B]  <-函数`SetDlgItemInt`
:004011AB A138214000              mov eax, dword ptr [00402138] <-取另一个地方的内存
:004011B0 85C0                    test eax, eax
:004011B2 0F8454FFFFFF            je 0040110C                   <-如果为空说明对方还没有按,返回
:004011B8 8B1530214000            mov edx, dword ptr [00402130] <-取出己方得到的数
:004011BE 3BC2                    cmp eax, edx <-比较
:004011C0 7C18                    jl 004011DA
:004011C2 6A00                    push 00000000
* Possible StringData Ref from Data Obj ->"Number Game"
                                  |
:004011C4 680A304000              push 0040300A
* Possible StringData Ref from Data Obj ->"Player 2 win !"
                                  |
:004011C9 6856304000              push 00403056 <-这里我把原来的字串改了
:004011CE 6A00                    push 00000000
:004011D0 E879FFFFFF              call 0040114E <-函数`MessageBox`,显示"Player 2 win !"
:004011D5 E913000000              jmp 004011ED
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004011C0(C)
|
:004011DA 6A00                    push 00000000
* Possible StringData Ref from Data Obj ->"Number Game"
                                  |
:004011DC 680A304000              push 0040300A
* Possible StringData Ref from Data Obj ->"Player 1 win !"
                                  |
:004011E1 6816304000              push 00403016
:004011E6 6A00                    push 00000000
:004011E8 E861FFFFFF              call 0040114E <-显示"Player 1 win !"
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004011D5(U)
|
:004011ED 33C0                    xor eax, eax <-善后工作,EAX=0
:004011EF A330214000              mov dword ptr [00402130], eax
:004011F4 A338214000              mov dword ptr [00402138], eax <-把两处内存都清零,以便下次比较
:004011F9 E90EFFFFFF              jmp 0040110C <-走喽
:004011FE 0000                    add byte ptr [eax], al <-好险,只剩两个字节的空余了
```

按钮2的处理十分类似，只是因为地方不够了，在后面`403XXX`处找了一块空地（本来是`.data`块，比较危险，不推荐使用，最好新建一个块）在`HIEW`里这种跨块的长跳转好像有点问题，总是搞不对地址，（不知是`HIEW`的问题还是我的脑袋的问题，请高手讲解）后来只好用专门的伪代码工具得到机器码再输进去.（痛苦啊!）

改成以后试试，很像样子了。任务只完成了一半，那个得50分胜利和按“M”AND“Z”还没写，不过我实在想歇歇了，而且寒假作业还没写呢。BYE！
