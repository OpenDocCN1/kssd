

# IDA-IDC 建立 PE Header

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-388.htm`](https://www.pediy.com/kssd/pediy08/pediy8-388.htm)

**写在前面的话：**  
这篇关于建立 PE 文件头的 IDC 描述语言的代码和很早前发的那个 DELPHI.IDC 是同期作品，这个作品一只没有能够完成，有自己的原因，也有 IDC 所提供的函数的原因，在 IDA 中，你可以对一个结构的成员进行 OpOffEx 的操作，但在 IDC 中，对结构的建立却没有相应的参数，具体如下：  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> static ResourceStruct(ResBase)
> {
>   auto id;
>   id = AddStrucEx(-1,"IMAGE\_RESOURCE\_DIRECTORY\_ENTRY\_8\_8",0);
>   id = GetStrucIdByName("IMAGE\_RESOURCE\_DIRECTORY\_ENTRY\_8\_8");
>   AddStrucMember(id,"Name", 0X0,  0x20500400, ResBase, 4);
>   AddStrucMember(id,"DataIsDirectory",  0X4,  0x20500400, ResBase, 4);
> }
>   
> 
> * * *
> 
> 
> ```

  
  
下面是 IDA 在线帮助中对函数 AddStrucMember 的具体说明；  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\*\* add structure member  
        arguments:      id      - structure type ID  
                        name    - name of the new member  
                        offset  - offset of the new member  
                        flag    - type of the new member. Should be one of  
                                  FF\_BYTE..FF\_PACKREAL (see above)  
                                  combined with FF\_DATA  
                        typeid  - structure id if 'flag' == FF\_STRU  
                                  Denotes type of the member if the member  
                                  itself is a structure. Otherwise should be  
                                  -1.  
                                  if isOff0(flag) then typeid specifies  
                                  the offset base.  
                                  if isASCII(flag) then typeid specifies  
                                  the string type (ASCSTR\_...).  
                        nbytes  - number of bytes in the new member  
        returns:        0 - ok, otherwise error code:  
#define STRUC\_ERROR\_MEMBER\_NAME   1     // already have member with this name (bad name)  
#define STRUC\_ERROR\_MEMBER\_OFFSET 2     // already have member at this offset  
#define STRUC\_ERROR\_MEMBER\_SIZE   3     // bad number of bytes or bad sizeof(type)  
   
long AddStrucMember(long id,string name,long offset,long flag,  
                    long typeid,long nbytes);  
  
其中 flag 参数是我们主要研究的对象，flag 参数具体值如下：  
  
第 9-10 位  
  
#define MS\_CLS  0x00000600L             // Mask for typing  
#define FF\_CODE 0x00000600L             // Code ?  
#define FF\_DATA 0x00000400L             // Data ?  
#define FF\_TAIL 0x00000200L             // Tail ?  
#define FF\_UNK  0x00000000L             // Unknown ?  
  
第 20-23 位  
#define MS\_0TYPE 0x00F00000L            // Mask for 1st arg typing  
#define FF\_0VOID 0x00000000L            // Void (unknown)?  
#define FF\_0NUMH 0x00100000L            // Hexadecimal number?  
#define FF\_0NUMD 0x00200000L            // Decimal number?  
#define FF\_0CHAR 0x00300000L            // Char ('x')?  
#define FF\_0SEG  0x00400000L            // Segment?  
#define FF\_0OFF  0x00500000L            // Offset?  
#define FF\_0NUMB 0x00600000L            // Binary number?  
#define FF\_0NUMO 0x00700000L            // Octal number?  
#define FF\_0ENUM 0x00800000L            // Enumeration?  
#define FF\_0FOP  0x00900000L            // Forced operand?  
#define FF\_0STRO 0x00A00000L            // Struct offset?  
#define FF\_0STK  0x00B00000L            // Stack variable?  
  
第 28-31 位  
#define DT\_TYPE 0xF0000000L             // Mask for DATA typing  
#define FF\_BYTE 0x00000000L             // byte  
#define FF\_WORD 0x10000000L             // word  
#define FF\_DWRD 0x20000000L             // dword  
#define FF\_QWRD 0x30000000L             // qword  
#define FF\_TBYT 0x40000000L             // tbyte  
#define FF\_ASCI 0x50000000L             // ASCII ?  
#define FF\_STRU 0x60000000L             // Struct ?  
#define FF\_OWRD 0x70000000L             // octaword (16 bytes)  
#define FF\_FLOAT 0x80000000L            // float  
#define FF\_DOUBLE 0x90000000L           // double  
#define FF\_PACKREAL 0xA0000000L         // packed decimal real  
#define FF\_ALIGN    0xB0000000L         // alignment directive  
  
在我们这个例子中  
0x20500400= FF\_DATA | FF\_0OFF | FF\_DWRD  
  
这个 Offset 的设置相当于：  
OpOffEx(ea, 0, REF\_OFF32, -1, ResBase, 0x0);  
  
而我们需要的是类似下面的设置：  
OpOffEx(ea, 0, REF\_OFF32, -1, ResBase, 0x80000000);  
  
因为这个问题，资源这部分的内容无法完成，当时就放下了，准备以后有时间在想个折中的办法，或者什么曲线救国的方法，这一拖，已过了半年，而我对 IDC 的那点理解也完全溢出了（已经完全忘记了，哈哈）。  
  
现在将这个没有完工的东西拿出来，希望大家不要见笑，并希望有能力的兄弟继续完成这段东西。  
  
**说明：**  
这是用 IDA 的描述语言写的 IDC 文本，使用这段描述语言，对 IDA 在加载可执行文件时有一定的要求。在 IDA 加载文件时的 Load a New file 对话框有如下的选择：  
1、Load Resource 前打勾；  
2、Manual Load 前打勾；  
3、Make Imports Segment 不打勾；  
  
注：如果你没有修改 IDA.CFG 文件，IDA 缺省是不加载头文件的，也不加载资源文件，并对输入表建立一个虚拟的输入表段，这样，我们的这个描述语言将无法正确工作。  
  
**目的：**  
这篇文章是为了帮助那些希望理解 PE 文件格式的读者，这个 IDC 对于真正的破解并没有什么具体的帮助。一个可执行文件经过这个 IDC 整理之后，将建立非常直观的 PE 文件头和相应的输入表，输出表，延时输入表，捆绑输入表，调试输入表，（资源表没有建立）。用这种格式观察 PE 文件结构和相应的输入输出表，有助你对这些结构的深入理解，我们大多使用的 PE 工具给出的是一个非常抽象的和经过删节的 PE 结构。在 IDA 中你所见到的 PE 结构是在内存中的真实的影像，理解这些，对于你有志写自己的 PE 结构方面的程序或文章，无疑会提供许多帮助。  
  
**附件说明**  
附加中有两个 htm 文件显示了经过这个 IDC 整理后的结果，和 CreatePEHead.IDC 

单击此处打开附件:idc.rar

 4dd466d0e1e0cc123afbe504da2462ca