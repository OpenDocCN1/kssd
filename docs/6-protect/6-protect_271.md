

# 模拟 Xuetr 枚举 GDT

> 原文：[`www.pediy.com/kssd/pediy12/130581.html`](https://www.pediy.com/kssd/pediy12/130581.html)

标 题: 【原创】模拟 Xuetr 枚举 GDT  
作 者:  房有亮  
时 间: 2011-03-09,15:08  
  
说是原创其实也是 A 黑客防线的代码，其中也参考了 Yas,不过请看标题，不知道能不能申个精。![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")  
下面不多说了，直接代码，第一次弄有错误请大家批评。  
//结构  
typedef struct \_SELECTOR{  
  unsigned short dpl: 2;  
  unsigned short ti: 1;  
  unsigned short index: 13;  
} SELECTOR, \*PSELECTOR;  
  
typedef struct \_CODE\_DATA\_SEGMMENT\_DESC{  
  unsigned char limit\_0\_7;  
  unsigned char limit\_8\_15;  
  unsigned char base\_0\_7;  
  unsigned char base\_8\_15;  
  unsigned char base\_16\_23;  
  unsigned char type: 4;  
  unsigned char system: 1;  
  unsigned char dpl: 2;  
  unsigned char present: 1;  
  unsigned char limit\_16\_19: 4;  
  unsigned char avl: 1;  
  unsigned char l: 1;  
  unsigned char d\_b: 1;  
  unsigned char granularity: 1;  
  unsigned char base\_24\_31;  
} CODE\_DATA\_SEGMMENT\_DESC, \*PCODE\_DATA\_SEGMMENT\_DESC;  
  
typedef struct \_INTER\_TRAP\_GATE\_DESC{  
  unsigned short offset\_0\_15;  
  unsigned short selector;  
  unsigned char reserved;  
  unsigned char type: 4;  
  unsigned char system: 1;  
  unsigned char dpl: 2;  
  unsigned char present: 1;  
  unsigned short offset\_16\_31;  
} INTER\_TRAP\_GATE\_DESC, \*PINTER\_TRAP\_GATE\_DESC;  
  
typedef struct \_CALL\_GATE\_DESC{  
  unsigned short offset\_0\_15;  
  unsigned short selector;  
  unsigned char param\_count: 5;  
  unsigned char some\_bits: 3;  
  unsigned char type: 4;  
  unsigned char system: 1;  
  unsigned char dpl: 2;  
  unsigned char present: 1;  
  unsigned short offset\_16\_31;  
} CALL\_GATE\_DESC, \*PCALL\_GATE\_DESC;  
  
typedef struct \_TASK\_GATE\_DESC{  
  unsigned short reserved1;  
  unsigned short selector;  
  unsigned char reserved2;  
  unsigned char type: 4;  
  unsigned char system: 1;  
  unsigned char dpl: 2;  
  unsigned char present: 1;  
  unsigned short reserved3;  
} TASK\_GATE\_DESC, \*PTASK\_GATE\_DESC;  
  
typedef struct \_GENERAL\_DESC{  
  unsigned int unknown1;  
  unsigned char unknown2;  
  unsigned char type: 4;  
  unsigned char system: 1;  
  unsigned char dpl: 2;  
  unsigned char present: 1;  
  unsigned short unknown3;  
} GENERAL\_DESC, \*PGENERAL\_DESC;  
  
typedef struct \_GDTR{  
  unsigned short GdtLimit;  
  unsigned short LowGdtbase;  
  unsigned short HighGdtbase;  
} GDTR, \*PGDTR;  
  
//变量  
#define         SIZEOF\_GDTR 6;  
#define         SIZEOF\_IDTR 6;  
#define         SIZEOF\_DESCRIPTOR 8;  
#define         SIZEOF\_DESC 8  
#define         SIZEOF\_SELECTOR 2;  
#define         MAKELONG(a, b) ((unsigned long) (((unsigned short) (a)) | ((unsigned long) ((unsigned short) (b))) << 16)) //IDT    
#define FOUR\_BYTE\_TO\_DWORD( byte\_0\_7, byte\_8\_15, byte\_16\_23, byte\_24\_31 ) \\  
  \\  
  (unsigned int)byte\_0\_7      |                \\  
  (unsigned int)byte\_8\_15 << 8  |                \\  
  (unsigned int)byte\_16\_23 << 16  |                \\  
  (unsigned int)byte\_24\_31 << 24  
  
#define TWENTY\_BIT\_TO\_DWORD( bit\_0\_7, bit\_8\_15, bit\_16\_19 ) \\  
  \\  
  (unsigned int)bit\_0\_7      |        \\  
  (unsigned int)bit\_8\_15 << 8    |        \\  
  (unsigned int)bit\_16\_19 << 16  
  
#define TWO\_SHORT\_TO\_DWORD( short\_0\_15, short\_16\_31 ) \\  
  \\  
  (unsigned int)short\_0\_15    |      \\  
  (unsigned int)short\_16\_31 << 16  
  
char\* NonSystemType\[16\] =  
{  
  "数据段：只读",          // 00  
  "数据段：只读,访问",      // 01  
  "数据段：读写",          // 02    
  "数据段：读写,访问",      // 03  
  "数据段：只读,向下扩展",    // 04  
  "数据段：只读,向下扩展,访问",  // 05  
  "数据段：读写,向下扩展",    // 06  
  "数据段：读写,向下扩展,访问",  // 07  
  "代码段：只执行",        // 08  
  "代码段：只执行,访问",      // 09  
  "代码段：执行/读",      // 10  
  "代码段：执行/读,访问",    // 11    
  "代码段：只执行,符合",      // 12  
  "代码段：只执行,符合,访问",  // 13    
  "代码段：执行/只读, 符合",    // 14  
  "代码段：执行/只读, 符合,访问"  // 15  
};  
  
char\* SystemType\[16\] =  
{  
  "保留",            // 00  
  "16 位：任务状态段(可用)",  // 01  
  "LDT",            // 02  
  "16 位：任务状态段(忙)",    // 03  
  "16 位：调用门",        // 04  
  "任务门",          // 05  
  "16 位：中断门",            // 06  
  "16 位：陷阱门",        // 07  
  "保留",            // 08  
  "32 位：任务状态段(可用)",  // 09  
  "保留",            // 10  
  "32 位：任务状态段(忙)",    // 11  
  "32 位：调用门",        // 12  
  "保留",            // 13  
  "32 位：中断门",            // 14  
  "32 位：陷阱门"        // 15  
};  
//函数  
//@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
//模块名字：VOID FindGDT(ULONG Index)  
//模块功能：获取 GDT 地址  
//返回数值：  
//@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
//参数说明：参数名      |   输入/输出  |        参数说明  
//@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
VOID FindGDT(ULONG Index)  
{      
  PGENERAL\_DESC  GDT;   
  PCODE\_DATA\_SEGMMENT\_DESC Code\_Data\_Desc;  
  PCALL\_GATE\_DESC Call\_Gate;  
  GDTR  Gdt\_Base;    
  ULONG GdtCount;  
  ULONG i;   
  ULONG BaseAddr;  
  ULONG limit;  
  ULONG offset;  
  PUCHAR TypeName;  
  ULONG  DPL;  
  PUCHAR SegmentSize;  
  
KeSetSystemAffinityThread(Index);//绑定 CPU  
  \_\_asm  sgdt  Gdt\_Base;  
  
  GDT = (PGENERAL\_DESC) MAKELONG(Gdt\_Base.LowGdtbase,Gdt\_Base.HighGdtbase);  
  GdtCount = (ULONG)((Gdt\_Base.GdtLimit+1) >> 3);  
  
  for(i = 0; i < GdtCount; i++)  
  {  
    Code\_Data\_Desc = (PCODE\_DATA\_SEGMMENT\_DESC) &GDT\[i\];  
    BaseAddr = FOUR\_BYTE\_TO\_DWORD( Code\_Data\_Desc->base\_0\_7,Code\_Data\_Desc->base\_8\_15,Code\_Data\_Desc->base\_16\_23,Code\_Data\_Desc->base\_24\_31);  
    limit = TWENTY\_BIT\_TO\_DWORD( Code\_Data\_Desc->limit\_0\_7,  Code\_Data\_Desc->limit\_8\_15,  Code\_Data\_Desc->limit\_16\_19  );  
    if ( Code\_Data\_Desc->granularity == 1 )  
    {  
      SegmentSize = "Page";//4kb  
    }   
    else   
    {  
      SegmentSize = "Byte";//1byte  
    }  
    if ( GDT\[i\].system == 1 )  //代码和数据段描述符  
    {  
      TypeName = NonSystemType\[Code\_Data\_Desc->type\];  
    }  
    else  
    {  
      TypeName = SystemType\[GDT\[i\].type\];  
    }  
    if (GDT\[i\].type == 12) //32 位调用门  
    {  
      Call\_Gate = (PCALL\_GATE\_DESC) &GDT\[i\];  
      DPL = Call\_Gate->dpl;  
    }   
    else  
    {  
      DPL = Code\_Data\_Desc->dpl;  
    }  
    if (strncmp(TypeName,"保留",strlen("保留")) != 0)   
    {  
      DbgPrint("GDT-ID:%04x 基址： 0x%08X 界限：0x%08X 段粒度：%s 段特权级：%d 类型：%s", i,BaseAddr,limit,SegmentSize,DPL,TypeName);  
    }  
  }   
KeRevertToUserAffinityThread();//卸载绑定  
}  
![](img/dc72760be29362914ad766fcec12be31.jpg)  
可编译源码

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || EnumGDT.rar  
 |