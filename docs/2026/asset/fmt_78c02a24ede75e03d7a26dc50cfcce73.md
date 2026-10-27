
![bp2jumpvmexit](https://bbs.kanxue.com/upload/attach/202102/93_8D73FYUQ5ZVM8HS.png)

`od` `view` `查看` 菜单里面有 `run trace` 我们点击进去然后右键记录到文件方便我们后面好静态分析

*如果你的 `run trace` 记录的文件没有汇编 参数等请百度设置一下 `trace` 选项*

- 2. 然后我们按 `F9` 到达我们第一个断点在按快捷键 `Ctrl+F11` 跟踪步入然后到我们的第二个断点，这个时候我们查看我们 `run trace` 查看追踪出的信息然后右键关闭记录文件下图是记录的跟踪信息

![runtraceasm](https://bbs.kanxue.com/upload/attach/202102/93_FNDY7SZQGT9EPT7.png)

然后看看保存的跟踪文件大概是这个样子我有处理过

![runtracefile](https://bbs.kanxue.com/upload/attach/202102/93_4GSTPFM9BQAU4GB.png)

那么到现在我们就把 `vm` 后的代码给扒下来了之后我们只要好好分析这个文件就 `OK` 了

下面我们先搜索一下我们之前 `vm` 的汇编中的常量我们先搜索第一个 `1111` 为什么是 `00001111` 就不多解释了

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `0041DB98` `Main     BSWAP ECX                                    ; ECX``=``00001111`  `0041DB9A` `Main     RCR AH,CL`  `0041DB9C` `Main     OR EAX,EBP                                   ; EAX``=``0041DBFF`  `0041DB9E` `Main     NEG AX                                       ; EAX``=``00412401`  `0041DBA1` `Main     XOR EBX,ECX                                  ; EBX``=``0046B588`  `0041DBA3` `Main     BSR EAX,ESP                                  ; EAX``=``00000014`  `0041DBA6` `Main     SUB EDI,``4`                                    `; EDI``=``0012FF88`  `0041DBAC` `Main     LAHF                                         ; EAX``=``00000614`  `0041DBAD` `Main     MOV DWORD PTR DS:[EDI],ECX` |

我们去一下混淆

|  |  |

| --- | --- |

| 1  2  3 | `0041DB98` `Main     BSWAP ECX                                    ; ECX``=``00001111`  `0041DBA6` `Main     SUB EDI,``4`                                    `; EDI``=``0012FF88`  `0041DBAD` `Main     MOV DWORD PTR DS:[EDI],ECX` |

就这 `3` 句有用的得到 `00001111` 怎么来的我们先不深究 `EDI-=4` 然后在向 `[EDI]=ECX` 这个像不像是压栈的操作，栈的增长方向是向下的是吧栈顶减小就是压栈 `思考一下 那有没有栈顶减小的是出栈呢`。回到上文 `一、认识与环境搭建` 中我们说了 `ebp` 是代表栈，`EDI` 是跳转基址 `JumpBase`，怎么现在又说这个 `EDI` 是代表栈了。莫急我们在往下看看

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `0041DBBF` `Main     MOV EAX,DWORD PTR DS:[ESI]                ; EAX``=``1AB34C77`  `0041DBC1` `Main` `CMP` `EDX,EDX`  `0041DBC3` `Main     XOR EAX,EBX                               ; EAX``=``1AF5F9FF`  `0041DBC5` `Main     JMP vmptest_.``0046A2FF`  `0046A2FF` `Main     BSWAP EAX                                 ; EAX``=``FFF9F51A`  `0046A301` `Main     JMP vmptest_.``00476E95`  `00476E95` `Main     DEC EAX                                   ; EAX``=``FFF9F519`  `00476E96` `Main     NOT EAX                                   ; EAX``=``00060AE6`  `00476E98` `Main     JMP vmptest_.``0044E862`  `0044E862` `Main     DEC EAX                                   ; EAX``=``00060AE5`  `0044E863` `Main     XOR EBX,EAX                               ; EBX``=``0040BF6D`  `0044E865` `Main     JMP vmptest_.``0041F316`  `0041F316` `Main     ADD EBP,EAX                               ; EBP``=``0047E65C`  `0041F318` `Main     JMP vmptest_.``00472E41`  `00472E41` `Main     LEA EDX,DWORD PTR SS:[ESP``+``60``]             ; EDX``=``0012FF00`  `00472E45` `Main     TEST DH,AL`  `00472E47` `Main     CLC`  `00472E48` `Main` `CMP` `EDI,EDX`  `00472E4A` `Main     JMP vmptest_.``0046EE86`  `0046EE86` `Main     JA vmptest_.``00480A05`  `00480A05` `Main     JMP EBP` |

很乱我们去混淆一下

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `0041DBBF` `Main     MOV EAX,DWORD PTR DS:[ESI]                ; EAX``=``1AB34C77`  `0041DBC3` `Main     XOR EAX,EBX`  `0046A2FF` `Main     BSWAP EAX                                 ; EAX``=``FFF9F51A`  `00476E95` `Main     DEC EAX                                   ; EAX``=``FFF9F519`  `00476E96` `Main     NOT EAX                                   ; EAX``=``00060AE6`  `0044E862` `Main     DEC EAX                                   ; EAX``=``00060AE5`  `0041F316` `Main     ADD EBP,EAX                               ; EBP``=``0047E65C`  `00472E41` `Main     LEA EDX,DWORD PTR SS:[ESP``+``60``]             ; EDX``=``0012FF00`  `00472E48` `Main` `CMP` `EDI,EDX`  `0046EE86` `Main     JA vmptest_.``00480A05`  `00480A05` `Main     JMP EBP` |

我们看到 `esi` 还是 `bytescode` 他这里是得到了跳转间隔密文，我们往的话发现之前的 `00001111` 值也是从 `esi` 中得到密文解密出来的，那么说明 `esi` 里面不仅存在数据常量也有跳转间隔，而且他们都是密文然后回到这里我们又发现 `ebp` 变为了跳转 `base` `JumpBase`。看到 `00472E41` 这条指令给 `edx` 赋值 `esp + 0x60` 你还记得最开始虚拟机入口时那两句汇编吗

|  |  |

| --- | --- |

| 1  2 | `.vmp30:``00430A1F` `8B` `EC         mov     ebp, esp              ; ebp 为压环境后esp的值`  `.vmp30:``00430A21` `81` `EC C0` `00``+`  `sub     esp,` `0C0h`             `; esp下移` `0xC0` |

然后我们之前说了 `edi` 是栈，因为上面那个语句是 `edi-=4` 然后把 `00001111` 赋值给这个 `EDI` 指向的地址 `其实这里我们截取了VM_PushImm32指令分析的` 是吧，然后 `edi` 就是栈然后 `edi` 这里又与这个 `esp + 0x60` 比教，这里的结果是大于的结果 `因为是追踪出来的所以只有一个分支` 我们结合这 `3` 个块分析出一个图

![VMStackAndVmReg](https://bbs.kanxue.com/upload/attach/202102/93_ZEJH3PSNKDJKPA2.png)

那么这个图到底正确不正确或者完善了没呢？到目前我们分析的我觉得应该是没问题的，目前只是我们的初略分析。

## 三、详细分析

我们先分析一下头和尾，那我们怎么知道哪里才是一个 `VM` 指令的开始和结束呢？记得 `jmp reg` 和 `push + ret` 没，我们就由这两个去确定一个 `VM` 指令的范围

- A、分析一下 `VM` 进入的操作

|  |  |

| --- | --- |
