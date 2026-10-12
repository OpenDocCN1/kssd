
# NetAnts v1.23

> 原文：[`www.pediy.com/kssd/pediy03/forum878.htm`](https://www.pediy.com/kssd/pediy03/forum878.htm)

`NetAnts`（网络蚂蚁）`v1.23` 广告条破解法

**作者**：`PaulYoung`

**破解时间**：`2001.4.24` 凌晨 `02:20`

**破解工具**：`TRW 2000` `V1.03`, `W32DASM` `V8.93`, `HIEW` `V6.55`

**软件下载**：`http://download.yesky.com/servlet/mydown.yeskydown?tag=7&objID=18831&se=1`

刚刚看完 `C-pen` 大侠关于破解软件广告条的教学，很受启发，刚好手头上有一个 `NetAnts` `v1.23`，拿来试试手。

1. `Ctrl+N` 激活 `TRW 2000`，下 `bpx movewindow`（函数功能：该函数改变指定窗口的位置和尺寸。对于顶层窗口，位置和尺寸是相对于屏幕的左上角的：对于子窗口，位置和尺寸是相对于父窗口客户区的左上角坐标的。）；
2. `F5` 退出，运行 `NetAnts`，立即被拦截；
3. 每当 `NetAnts` 被 `TRW 2000` 拦截，按 `F5`，共按 `23` 或 `24`、`25` 次，下 `PMODULE`，来到下面（这可是我试过×次的失败，测试出来的，因为那时 `NetAnts` 的广告窗口刚好若隐若现。如果未到，可以继续按 `F5`，再下 `PMODULE`，直到看到下面代码为止）：

```
:0041BE44 FF35A8AB4400          push dword ptr [0044ABA8]           //0044ABA8 就是广告条的 Flag
:0041BE4A FFD7                  call edi                             //[USER32!MoveWindow]
:0041BE4C 8345F443              add dword ptr [ebp-0C], 00000043      //光标停在这里
:0041BE50 391D78AB4400          cmp dword ptr [0044AB78], ebx
:0041BE56 0F84F5000000          je 0041BF51
```

4. 根据 `C-pen` 的教学分析，可以知道，`0044ABA8` 就是广告条的 `Flag`，这段代码是用来设定广告窗口大小的。
5. 用 `W32DASM` 反汇编 `NetAnts`（好耶，没加壳），查找 `0044ABA8`，第 `1` 次找到上面这段代码，第 `2` 次找到下面这段代码……

```
:0041C030 57                    push edi
:0041C031 E86265FEFF          call 00402598                        //调用函数的 Call
:0041C036 3BC3                  cmp eax, ebx
:0041C038 59                    pop ecx
:0041C039 A3A8AB4400            mov dword ptr [0044ABA8], eax       //程序会将窗口的 ID 放入 0044ABA8
                                                                 //供程序应用
:0041C03E 0F843C1F0000          je 0041DF80
:0041C044 57                    push edi
```

6. 好了，开始修改，具体如下（我是用 `HIEW` 修改的）：

①

```
:0041BE4C 8345F443              add dword ptr [ebp-0C], 00000043    //00000043 是用来设定广告窗口
                                                                 //大小的，因此我们把 00000043
                                                                 //改为 00000000，即在 HIEW 中把
                                                                 //8345F443 改为 8345F400 即可
```

②

```
:0041C030 57                    push edi                 //改为 XOR EAX,EAX 即 33C0
:0041C031 E86265FEFF          call 00402598             //NOP
:0041C036 3BC3                  cmp eax, ebx             //NOP
:0041C038 59                    pop ecx                 //NOP
:0041C039 A3A8AB4400            mov dword ptr [0044ABA8], eax   //小心，这个要保留！
:0041C03E 0F843C1F0000          je 0041DF80              //NOP
```

最后，运行一下你的作品，哇噻耶！！那只大蚂蚁不再戴着大墨镜向你推销广告了！！！

YEAH……

THE EDN

`netants` `1.0-1.23` 一直不变的修改方法:

1. 去广告

`* Possible Reference to String Resource ID=00001: "Invalid retry delay"`

```
:0041BE36 6A01                  push 00000001
:0041BE38 2B45F0                sub eax, dword ptr [ebp-10]
:0041BE3B 6A42                  push 00000042                  //push 0
:0041BE3D 50                    push eax
:0041BE3E FF75F4                push [ebp-0C]
:0041BE41 FF75F0                push [ebp-10]
:0041BE44 FF35A8AB4400          push dword ptr [0044ABA8]
:0041BE4A FFD7                  call edi
:0041BE4C 8345F443              add dword ptr [ebp-0C], 00000043    //nop
:0041BE50 391D78AB4400          cmp dword ptr [0044AB78], ebx
:0041BE56 0F84F5000000          je 0041BF51
```

2. 清后门(先用 `optout0.999999` 把垃圾清掉 `http://grc.com/optout.htm` 作者不再更新它了)

```
40223e call 40bc19  ->nop
4024ee jne  4024fd  ->jmp 4024fd
40261c jne  40262b  ->jmp 40262b
40261a cmp  eax,esi ->mov al,1
```

改这些的主要目的是把缺少文件的提示去掉

`by zombieys` `4dd466d0e1e0cc123afbe504da2462ca`
