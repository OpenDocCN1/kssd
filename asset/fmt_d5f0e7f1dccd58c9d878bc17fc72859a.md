
# 游戏《东方绯想天》简单修改器以及一个辅助工具的实现

> 原文：[`www.pediy.com/kssd/pediy10/96332.html`](https://www.pediy.com/kssd/pediy10/96332.html)

《东方绯想天》是最近一个很流行的格斗游戏，还支持网络对战。一时感兴趣就拿这个游戏搞一下了。游戏中的界面如下：

由于本来就只是打算研究一下玩玩看的，所以调试的过程中也没做什么详细的笔记，大虾们飘过，新手耶随便看看好了。

## 一、 简单的修改

由于这个是格斗游戏，所以存放数据的地址很难找，现在可以知道的信息只有：显示面板上显示的连击伤害点数，还有卡片数目。我用 `CheatEngine` 将卡片数目的地址找出来后，再用 `od` 挂接，下内存访问断点，在一句这样的代码中停了下来 `mov cx,word ptr:[esi+0x56c]`。而往上回溯会发现很多操作都是以 `esi` 的指为基址加上偏移来修改的，所以可以判断这个是玩家数据的基址。

但 `esi` 的值是 `0x014XXXXX` 这样的，这不是在游戏中的常量区段中而是在进程的堆存储区中，并且每次进行游戏都会改变的，所以要往上回溯，寻找保存着临时数据基址的空间。一直往上可以总结出寻址的规律为：

`Mov eax,[0x6e6244]`

`Mov eax,[eax+0x0C]`

此时 `eax` 中的即为 `1p` 的数据基址。

尽管找到了数据基址，但像 `HP` 还有灵力等数据仍然未能确定。我比较笨，只好根据连击的伤害点数扣除的血量估计总血量的大概数值，和在上面那个内存断点中断下来的地方找一下相关的代码。于是找到了以下几个数值的偏移：

*   对方数据基址：`+0x170`
*   `HP`：`+0x174`
*   灵力：`+0x482`

顺带还找到了扣除 `HP` 的过程是先找到玩家的数据基址，在从 `0x170` 的地方取出敌对玩家的数据基址，再对对方的 `HP` 血量进行修改，其中我找到关键的一句代码就是在 `0x46baf1` 处的：

`sub word ptr [esi+174], bx`

将这句代码 `nop` 掉，双方就都不会掉血了。

而到了这里，写一个锁定 `HP` 的修改器就再简单不过了：写一个 `dll` 注入到游戏中，将这句代码改成一个跳转跳到自己的处理函数中，判断 `esi` 是不是 `1P` 的数据基址，是的话则跳过不进行处理，否则就执行这句代码。不过我比较懒，就用了另外一个比较笨的方法来实现：开启一条线程，隔一段时间往游戏进程中的内存地址中写入固定的数值，这样就等于是变相锁定 `HP` 了，具体代码如下：

```
DWORD WINAPI WorkThread(LPVOID)
{
  DWORD dataaddr;
  DWORD readbytes;
  WORD hp = 0x7fff;
  while(ischeat)
  {
    ReadProcessMemory(thhandle,(LPVOID)0x6e6244,&dataaddr,4,&readbytes);
    dataaddr+=0x0c;
    ReadProcessMemory(thhandle,(LPVOID)dataaddr,&dataaddr,4,&readbytes);
    dataaddr+=0x174;
    WriteProcessMemory(thhandle,(LPVOID)dataaddr,&hp,2,&readbytes);
    Sleep(1000);
  }
  return 0;
}
```

## 二、 输入 IP 的辅助工具

由于这个游戏是支持网站的，而我周围的同学有不少都是用教育网的，于是就写了一个小程序用来保存他们的 IP 地址然后要对战的时候直接在这个程序中选 IP 就行了。

寻找游戏中显示 IP 的方法也是用 `CheatEngine` 查找，找到地址后就用 `OD` 附加再在那个地址下内存访问断点，然后找到寻址规律如下：

`mov eax, dword ptr [860E08]`

`mov esi, dword ptr [eax+4]`

`mov ecx, dword ptr [esi+8]`

`mov esi, ecx`

`lea ebx, dword ptr [esi+420]`

内存中的 IP 地址结构为：

```
Struct ip{
  WORD ip[4];
  DWORD port;
};
```

有了这些信息，就可以用 `ReadProcessMemory` 跟 `WriteProcessMemory` 来修改游戏进程中对应的内存数值了。

具体代码为：

```
SetTouhouIP proc index
      local @readdata
      local @readbytes
      local @writedata[12]:BYTE
      local @ipdbase
      local @readbase
      local thhandle
      assume esi:ptr IPDATA
      invoke GetTHProcess
      .if eax == INVALID_HANDLE_VALUE
   invoke MessageBox,NULL,addr error3,addr errinfo,0
   ret
      .endif
      mov thhandle,eax
      invoke ReadProcessMemory,thhandle,database,addr @readdata,4,addr @readbytes
      mov eax, @readdata
      add eax,4
      mov @readbase,eax       
      invoke ReadProcessMemory,thhandle,@readbase,addr @readdata,4,addr @readbytes
      mov eax,@readdata
      add eax,8
      mov @readbase,eax
      invoke ReadProcessMemory,thhandle,@readbase,addr @readdata,4,addr @readbytes
      mov eax,@readdata
      add eax,420h
      mov @ipdbase,eax
      invoke RtlZeroMemory,addr @writedata,12
      mov ecx,index
      mov eax,offset ipdat
      imul ecx,ecx,sizeof IPDATA
      lea esi,[eax+ecx]
      add esi,8
      mov ecx,3
      xor ebx,ebx
      xor eax,eax
      lea edi,@writedata
      .while ecx < -1
            mov al,byte ptr [esi+ecx]
            mov byte ptr [edi+ebx],al
            dec ecx
            add ebx,2
      .endw
      mov eax,dword ptr [esi-8]
      mov dword ptr [edi+ebx],eax
      invoke WriteProcessMemory,thhandle,@ipdbase,addr @writedata,0ch,addr @readbytes
      invoke CloseHandle,thhandle
      ret
SetTouhouIP endp
```

而至于保存 IP 地址方法则是建立了一个文件，文件的头一个双字（32 位）是一个位图，用来表示空闲的块，然后程序会从中确认哪些块中保存有 IP 地址。具体的请查看附件的源代码。

程序效果如图

![`i447.photobucket.com/albums/qq192/3927aaa/thz.jpg`](img/fa67f27d4aef9abf3351cc095f617977.jpg)

上传的附件

| 截图 | 文件 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `src.rar` |
