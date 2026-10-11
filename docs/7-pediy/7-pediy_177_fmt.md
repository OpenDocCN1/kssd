
# 改造 PE 中的函数为导出函数

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-16162.htm`](https://www.pediy.com/kssd/pediy07/pediy7-162.htm)

## 1、前言：为什么要这么做

很多时候，我们发现一个 PE(EXE 或 DLL)中非常有用且功能独立的函数(`call xxxxxxxx`)，并且已经知道了这个函数的各个入口参数的类型和具体含义，我们想在其他的软件中使用这个函数。于是，我想到了将这个 PE 中的函数改成一个导出函数，这样，我们就可以在任何软件中通过`LoadLibrary("ThisPE.EXE")`等 API 来使用这个函数了。

## 2、技术基础：已经了解 PE 结构

## 3、分类

### （1）有导出表的 DLL 和 EXE 文件--非常简单

对于这类 PE 文件，要简单的多，因为只要修改一下 PE 中的 Export，增加上一个导出函数，并把这个导出函数的入口指向我们要导出的 call 的调用地址(RVA 格式)就可以了。甚至可以更简单的处理：找个不用的导出函数，改一下入口就可以了。

### （2）没有导出表的 EXE 文件--稍微复杂

因为大部分 EXE 文件没有导出表，所以我们必须要给它增加一个导出表。其实也很简单，给 EXE 增加一个节，并在这个节中按照导出表的格式构造一个导出函数，并将这个函数入口指向我们要导出的 call 的调用地址的 RVA，并在 PE 头中指出其位置和大小。

## 4、举例说明

`test.exe`：`ImageBase=0x00400000`

发现 `test.exe` 的 `0x00408050` 处为 `call 0x0040A012` 的 `0x0040A012` 是我们要导出的函数，我们将它导出为 `MyFunction` 函数。

（1）给 `test.exe` 增加一个节：`RVA=0x28000`, `size=0x1000`

（2）构造导出表：

```
My_Export_Table   dd 0 ;Characteristics

My_TimeDateStamp dd 0   ;TimeDateStamp

                 dw 0   ;MajorVersion

                 dw 0  ;MinorVersion

My_nName         dd My_DLL_nName-ImageBase ;nName

                 dd 1  ;nBase

                 dd 1  ;NumberOfFunctions

                 dd 1    ;NumberOfNames

My_AddressOfFunctions      dd 0x0040A012-ImageBase ;AddressOfFunctions

My_AddressOfNames          dd My_Fun_Name-ImageBase ;AddressOfNames

My_AddressOfNameOrdinals   dd 0  ;AddressOfNameOrdinals

---------------------------------------------------------------------

My_Fun_Name              db 'MyFunction',0

My_DLL_nName             db 'test.exe',0
```

（3）`LoadPe` 修改 PE 头，将导出表地址填入 `0x28000`，大小填入 `0x1000`，保存。

Spirng.W/2005.3.14

### 给出个另类方法

```
mod=LoadLibrary("xxx.dll"); //载入文件

func=(char*)mod+offset;     //找到函数入口地址

__asm push param2           //参数 2

__asm push param1           //参数 1

__asm call func;            //直接硬调用

__asm mov ret,eax           //保存返回值
```

> 引用:
>
> * * *
>
> *最初由 dwing 发布*
>
> **给出个另类方法:**
>
> ```
> mod=LoadLibrary("xxx.dll"); //载入文件
>
> func=(char*)mod+offset;     //找到函数入口地址
>
> __asm push param2           //参数 2
>
> __asm push param1           //参数 1
>
> ........
> ```

在 C 中呼叫地址，我通常用 function pointer

```
HINSTANCE mod;

int offset;

int (*func)( int, int );

...

mod = LoadLibrary( "xxx.dll" );

func = (void*)( (char*)mod + offset );

...

myvalue = func( 50, 100 );  //一般呼叫
```

如果呼叫 API 地址，便使用 WINAPI 方式的 calling convention

```
int (WINAPI *MessageBoxPointer)( HWND, LPCTSTR, LPCTSTR, UNIT );
```

VB 中用 `CallWindowProc` API 调用即可。

重定位是个问题。要不然把父程序的基址改掉。

> 引用:
>
> * * *
>
> *最初由 peaceclub 发布*
>
> **VB 中用 CallWindowProc API 调用即可。**
>
> **重定位是个问题。要不然把父程序的基址改掉。**

改父程序的基址不行.我把父程序的基址改成了 `800000`,加载目标 exe(基址 `400000`)后,Handle 不是 `400000`,晕.

怎么解决呢?

自程序之中再套嵌自程序的时候，一般调用都发生错误，因为基地址变了，而 EXE 中的基地址还是原来的。

只有子程序中没有其他套嵌子程序，或者被套嵌的子程序寻址方式是动态（比如取此时的 IP 数进行变换），才可以正常运行，还有个前提是 EXE 自程序中不再存在对 EXE 全局变量的存取.........

我觉得用 `CREATE_SUSPENDED` 形式的 `CreateProcess` 来建立进程，再调用进程中的子程序，或许效果好一点。

4dd466d0e1e0cc123afbe504da2462ca
