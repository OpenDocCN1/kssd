
# 发个输出表移位的脚本

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-586.htm`](https://www.pediy.com/kssd/pediy08/pediy8-586.htm)

主要应用于那些，修复输入表时复盖了输出表的情况！

在 `fly` 的贴子里面提到过，看雪老大写了一个输出表移位的工具，可我 一直就没找着![](img/e30f1ad374468d5494d012eeb872a0c5.jpg) ，所以写了这么个脚本![](img/d96007cf245c7669301e5df8beb41bf5.jpg)

```
/*
//////////////////////////////////////////////////
  Move The Address of export table!
  Author: ftts
  Email : ftts1@163.com
  Date  : 2006-07-22
  Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/

var ExportAddress
var Export
var tem
var value
var AddAdd
var Count
var DllBase
var SizeOfExportTable

GMEMI eip, MEMORYOWNER //get dll base

mov DllBase,$RESULT

mov tem,DllBase

add tem,3C

mov tem,[tem]//Get the position of Coff Header

add tem,DllBase

add tem,78

mov ExportAddress,[tem]//Get the position of Export Table

cmp ExportAddress,0

je exit1

add tem,4

mov SizeOfExportTable,[tem]

cmp SizeOfExportTable,0

je exit1

ask "Please input the Export Address you want save"

cmp $RESULT,0

je exit2

mov AddAdd,$RESULT

sub AddAdd,ExportAddress

mov tem,SizeOfExportTable

add tem,20

alloc tem

mov Export,$RESULT

exec

pushad

ende

mov eax,ExportAddress

add eax,DllBase

mov ecx,SizeOfExportTable

mov ebx,Export

exec

push ecx

push eax

push ebx

call memcpy

popad

ende

mov tem,Export

add tem,C //here is the address of Export dll's name

mov value,[tem]

add value,AddAdd

mov [tem],value

//-----------

mov tem,Export

add tem,1C // change function Address , function name ,ordinal

mov Count,3

c1:

mov value,[tem]

add value,AddAdd

mov [tem],value

add tem,4

dec Count

cmp Count,0

jne c1

//--------

mov tem,Export

add tem,18 //Number of function name

mov Count,[tem]

mov tem,Export

add tem,20 //function name address

mov value,[tem]

sub value,AddAdd

sub value,ExportAddress

add value,Export

mov tem,value

c2:

mov value,[tem]

add value,AddAdd

mov [tem],value

add tem,4

dec Count

cmp Count,0

jne c2

dma Export, SizeOfExportTable , "export.bin"

msg "Export table have been saved ,as file name is export.bin"

ret

exit1:

msg "This application have no any Export table yet!"

ret

exit2:

msg "you have not input the address of export"

ret

//--------------------------------------
```

这里还特别提醒那些修复重定位表的朋友。

有些壳在加壳的时候，把代码区块跟数据区块都放在一起了

如果直接用程序加密两个不同名的 `dll` 的话，再用 `ReloX` 去比较的话

就很容易出问题，因为这时 `dll` 初使化会改修改数据。

如果值不同的话，那就会重定会出错了。。

所以只有让两个 `dll` 在加载时运行到入口就要停下来。。

再 `dump` 就不会有问题了。。。

---

## 下面是一个最简单的加载器源码

```c
#include <windows.h>

#pragma comment (linker,"/base:0x800000")

#pragma comment (linker,"/entry:main_")

#pragma comment (linker,"/subsystem:console")

#pragma comment (lib,"kernel32.lib")

void main_()

{

    LoadLibrary("Test1.dll");

    LoadLibrary("Test2.dll");

    while(true)

    {

        Sleep(1000);

    }

}
```

