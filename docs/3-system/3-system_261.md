

# 关于"中断门"的数据结构

> 原文：[`www.pediy.com/kssd/pediy10/63078.html`](https://www.pediy.com/kssd/pediy10/63078.html)

在我印象当中,中断描述符是占八个字节的,然儿,今天看<<rootkit:windows 内核安全防护>>时,它却给出这样的结构:  
  
typedef struct \_idtentry   
{   
  //中断执行代码偏移量的底 16 位；   
  unsigned short  OffsetLow;   
  //选择器，也就是寄存器；   
  unsigned short  Selector;   
  //保留位，始终为零；   
  unsigned char    Reserved;   
  //IDT 中的门的类型：包括中断门，陷阱门和任务门；   
  unsigned char    Type:4;   
  //段标识位；   
  unsigned char    SegmentFlag:1;   
  //中断门的权限等级，0 表示内核级，3 表示用户级；   
  unsigned char    DPL:2;   
  //呈现标志位；   
  unsigned char    Present:1;   
  //中断执行代码偏移量的高 16 位；   
  unsigned short  OffsetHigh;   
}IDTENTRY,\*PIDTENTRY;   
  
我数了数,这应该是 11 个字节吧  
然后我在 google 了下,网络上关于门数据结构是:  
  
 GATE     STRUC      ;门结构类型定义   
    OFFSETL  DW      0  ;32 位偏移的低 16 位   
    SELECTOR DW      0  ;选择子   
    DCOUNT   DB      0  ;双字计数字段   
    GTYPE    DB      0  ;类型   
    OFFSETH  DW      0  ;32 位偏移的高 16 位   
    GATE     ENDS   
  
这个结构是占八个字节.  
  
  
请大家帮帮忙,为什么?![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")

typedef struct \_idtentry   
{   
  //中断执行代码偏移量的底 16 位；   
  unsigned short  OffsetLow;  //2 bytes  
  //选择器，也就是寄存器；   
  unsigned short  Selector;  //2 bytes  
  //保留位，始终为零；   
  unsigned char    Reserved;  //1 byte  
  //IDT 中的门的类型：包括中断门，陷阱门和任务门；   
  unsigned char    Type:4;  // 4 bit  
  //段标识位；   
  unsigned char    SegmentFlag:1; //1 bit  
  //中断门的权限等级，0 表示内核级，3 表示用户级；   
  unsigned char    DPL:2;  //2 bit  
  //呈现标志位；   
  unsigned char    Present:1; //1 bit  
  //中断执行代码偏移量的高 16 位；   
  unsigned short  OffsetHigh;  //2 bytes  
}IDTENTRY,\*PIDTENTRY;   
  
  
total:8 bytes