
```
MEMORY:FC07 sub_FC07        proc near               ; CODE XREF: sub_FC07+Cj
MEMORY:FC07                                         ; sub_FC16+1p ...
MEMORY:FC07                 jmp     short $+2
MEMORY:FC09 ; ---------------------------------------------------------------------------
MEMORY:FC09
MEMORY:FC09 loc_FC09:                               ; CODE XREF: sub_FC07j
MEMORY:FC09                 jmp     short $+2
MEMORY:FC0B ; ---------------------------------------------------------------------------
MEMORY:FC0B
MEMORY:FC0B loc_FC0B:                               ; CODE XREF: sub_FC07:loc_FC09j
MEMORY:FC0B                 in      al, 64h         ; 8042 keyboard controller status register
MEMORY:FC0B                                         ; 7:  PERR    1=parity error in data received from keyboard
MEMORY:FC0B                                         ;    +----------- AT Mode ----------+------------ PS/2 Mode ------------+
MEMORY:FC0B                                         ; 6: |RxTO    receive (Rx) timeout  | TO      general timeout (Rx or Tx)|
MEMORY:FC0B                                         ; 5: |TxTO    transmit (Tx) timeout | MOBF    mouse output buffer full  |
MEMORY:FC0B                                         ;    +------------------------------+-----------------------------------+
MEMORY:FC0B                                         ; 4:  INH     0=keyboard communications inhibited
MEMORY:FC0B                                         ; 3:  A2      0=60h was the port last written to, 1=64h was last
MEMORY:FC0B                                         ; 2:  SYS     distinguishes reset types: 0=cold reboot, 1=warm reboot
MEMORY:FC0B                                         ; 1:  IBF     1=input buffer full (keyboard can't accept data)
MEMORY:FC0B                                         ; 0:  OBF     1=output buffer full (data from keyboard is available)
MEMORY:FC0D                 cmp     al, 0FFh
MEMORY:FC0F                 jz      short locret_FC15
MEMORY:FC11                 test    al, 2
MEMORY:FC13                 jnz     short sub_FC07
MEMORY:FC15 locret_FC15:                            ; CODE XREF: sub_FC07+8j
MEMORY:FC15                 retn
```

具体的就不在这里赘述，大家可以自己去百度下关于A20地址线的相关资料。开启了A20之后，我们接着看程序又干了些啥

```
MEMORY:FA14                 mov     dword_6F00, 0FA41h
MEMORY:FA1D                 mov     ax, 1000h
MEMORY:FA20                 mov     es, ax
MEMORY:FA22                 assume es:nothing
MEMORY:FA22                 mov     eax, es:3Ch
MEMORY:FA27                 add     eax, 18h
MEMORY:FA2B                 mov     eax, es:[eax+10h]
MEMORY:FA31                 add     eax, 10000h
MEMORY:FA37                 mov     dword_FA9B, eax
MEMORY:FA3B                 xor     ax, ax
MEMORY:FA3D                 mov     es, ax
MEMORY:FA3F                 assume es:MEMORY
MEMORY:FA3F                 jmp     short loc_FA7C
```

这里则是从内存中读取一个数值，然后保存在`dword_FA9B`里，至于这个数值干啥用，我们接着看

```
MEMORY:FA7C loc_FA7C:                               ; CODE XREF: MEMORY:FA3Fj
MEMORY:FA7C                 cli
MEMORY:FA7D                 mov     word_FAAE, sp
MEMORY:FA81                 lgdt    fword ptr byte_FAD8
MEMORY:FA86                 mov     eax, cr0
MEMORY:FA89                 or      eax, 1
MEMORY:FA8D                 mov     cr0, eax
```

上面这段代码先是将`fword ptr byte_FAD8`加载到GDT表的寄存器中，然后开启保护模式。保护模式是否开启是由`cr0`的第0位控制的，为1表示开启保护模式，为0表示不开启。

开启了A20和保护模式之后，下面的代码就变成32位的了，至于后面都干了写啥，后期分析之后会分享出来，以上只是我个人的一些分析过程，有分析错误或者描述不对的地方，还希望大家能提出来，我们共同进步![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);)
  （29.83kb，10次下载）
- [2.jpg](javascript:void(0);)
  （33.84kb，14次下载）
- [3.jpg](javascript:void(0);)
  （61.30kb，29次下载）
- [MBR.jpg](javascript:void(0);)
  （166.17kb，27次下载）
- [DBR.jpg](javascript:void(0);)
  （161.32kb，23次下载）
- [13.jpg](javascript:void(0);)
  （65.34kb，15次下载）
- [7.jpg](javascript:void(0);)
  （86.72kb，7次下载）
- [short_dir.jpg](javascript:void(0);)
  （49.20kb，12次下载）
