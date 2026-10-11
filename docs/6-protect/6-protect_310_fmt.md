
# API 调用的隐藏技术

> 原文：[`www.pediy.com/kssd/pediy10/73398.html`](https://www.pediy.com/kssd/pediy10/73398.html)

`WINDOWS` 为它的程序员提供了大量的 API 接口，这极大地提高了软件开发的效率，
但是逆向分析工作人员往往就是通过这些 API 在极短时间内获取了大量信息，从而使他们成功定位目标程序的关键代码段。所以隐藏对 API 的调用可以有效地提高程序的抗分析能力。下面逐一介绍隐藏 API 调用的基本方法。

首先，看一下最简单的 情况：

```
MessageBoxA(NULL,"UFO","ZHY",MB_OK);
```

编译器会生成如下代码：

```
.text:0040115A                 push    0               ; uType
.text:0040115C                 push    offset Caption  ; "ZHY"
.text:00401161                 push    offset Text     ; "UFO"
.text:00401166                 push    0               ; hWnd
.text:00401168             call    ds:\__imp__MessageBoxA@16 ; MessageBoxA(x,x,x,x)
```

这是我们最常用的形式，IDA 等反汇编工具会轻易地对他们进行标识。

好了，我们来看一下稍微复杂点的情况：

```
TCHAR DllName[MAX_PATH]="user32.dll";
TCHAR ImportedFunctionName[MAX_PATH]="MessageBoxA";
HANDLE hDllModule=NULL;
DWORD dwFunctionAddress=0;
hDllModule=LoadLibrary(DllName);
(FARPROC)dwFunctionAddress=GetProcAddress(hDllModule,ImportedFunctionName);
((FARPROC)dwFunctionAddress)(NULL,OutputInfor,"Info",MB_OK | MB_ICONINFORMATION);
```

对应的汇编代码如下：

```
.text:004010E7                 lea     edx, [ebp+LibFileName]
.text:004010ED                 push    edx             ; lpLibFileName
.text:004010EE                 call    ds:\__imp__LoadLibraryA@4 ; LoadLibraryA(x)
.text:004010F4                 cmp     esi, esp
.text:004010F6                 call    __chkesp
.text:004010FB                 mov     [ebp+hModule], eax
.text:00401101                 mov     esi, esp
.text:00401103                 lea     eax, [ebp+ProcName]
.text:00401109                 push    eax             ; lpProcName
.text:0040110A                 mov     ecx, [ebp+hModule]
.text:00401110                 push    ecx             ; hModule
.text:00401111                 call    ds:\__imp__GetProcAddress@8 ; GetProcAddress(x,x)
.text:00401117                 cmp     esi, esp
.text:00401119                 call    __chkesp
.text:0040111E                 mov     [ebp+var_314], eax
.text:00401124                 mov     esi, esp
.text:00401126                 push    40h
.text:00401128                 push    offset aInfo    ; "Info"
.text:0040112D                 lea     edx, [ebp+var_30C]
.text:00401133                 push    edx
.text:00401134                 push    0
.text:00401136                 call    [ebp+var_314]     ；；对 MessageBox() 的调用
```

可以看到，IDA 没有识别我们 的 `MessageBox()`,这是值得庆贺的，但它识别除了 `LoadlLibrary()`,`GetProcAddress()`.这在 某种程度上也揭示我们将要调用的 API ，所以我们需要寻找更好的方法。

请看以下代码段：

```
_asm
{
      push MB_OK
      push StringAddress
      mov eax,0
      push eax
      push eax
      mov eax,offset Label001
      push eax
      jmp [dwFunctionAddress]    //call MessageBox()
Label002:
Label001:
    nop
    nop
}
```

以上代码段中的

```
      mov eax,offset Label001
      push eax
      jmp [dwFunctionAddress]
```

实现了 `CALL` 指令的功能，且功能有所增强，如果使用 `CALL` 指令，API 函数的返回地址必定是紧接 `CALL` 指令的下一条指令的地址。

而此代码段可返回到任意指定地址，形成更具迷惑性的代码。

```
_asm
{
    push MB_OK
    push StringAddress
    mov eax,0
    push eax
    push eax
    mov eax,offset Label001    ；；返回到 Label001
    push eax
    jmp [dwFunctionAddress]    ;;call MessageBox()
Label002:
    //Insert invalid instructions here to confuse the code analysiser
    //Instructions between the Label001 and Label002 will never be executed.
    nop
    nop
    call dwFunctionAddress
Label001:
    nop
    nop
}
```

由 IDA 得到的汇编代码如下：

```
.text:00401175                 push    0
.text:00401177                 push    [ebp+var_318]
.text:0040117D                 mov     eax, 0
.text:00401182                 push    eax
.text:00401183                 push    eax
.text:00401184                 mov     eax, offset loc_401198
.text:00401189                 push    eax
.text:0040118A                 jmp     [ebp+var_314]
.text:0040118A main            endp
.text:0040118A
.text:0040118A ; ---------------------------------------------------------------------------
.text:00401190                 db 2 dup(90h)     ；；我们的 NOP，
                            ；；可以插入任意数据
.text:00401192 ; ---------------------------------------------------------------------------
.text:00401192                 call    dword ptr [ebp-314h]
.text:00401198
.text:00401198 loc_401198:                       ; DATA XREF: main+174 o
```

在这，IDA 已经不能正确分析，说明此方法非常有效。`<<Hacker Disassembling Uncovered>>` 的作者对这种思想有以下描述：

`It can be very difficult to identify such functions (especially if they have no prolog); a context search gives no result because the body of any program contains plenty of JMP instructions used for near jumps. How, then, can we analyze all of them? If we don't identify the functions, two of them will drop out of sight  the called function and the function to which control is passed just upon returning. Unfortunately, there is no quick and easy solution to this problem; the only hook here is that the calling JMP practically always goes beyond the boundaries of the function in whose body it's located. We can determine the boundaries of a function by using an epilog.`

翻译为中文大意如下：

识别这样的函数是非常困难的（特别是当这些函数没有 `prolog` 的时候）；因为任何程序体都包含大量 的用来实现近跳转的 `JMP` 指令，所以联系上下文进行搜索不会有结果。那我们怎样识别这些函数呢？如果我们不识别出这些函数，被调用的函数以及函数的返回地址将会逃出我们的视线。不幸的是，这个问题没有快速且简单的解决方法。这里唯一的线索是：

调用跳转总是会跳转到调用跳转指令所在的函数体的外部。我们可以通过使用 `epilog` 来检测函数的边界。

好了，下面我们回到 `LoadLibrary()`,`GetProcAddress()`.

如果我们使用这种方法来获取 API 函数地址，在程序中必定会出现字符串，
如 `user32.dll`,`kernel32.dll`,`MessageBoxA`…等。这些信息可以通过分析工具轻易地得到，
这对于增强程序的抗分析能力 是无益的。

所以，加密字符串并在代码使用他们是进行动态解密是十分必要的。

我第一次看到这样的示例是在一个 `Downloader` 中：

```
PS\_\_\_\_\_\_:0040361C ; LPCSTR lpString2
PS\_\_\_\_\_\_:0040361C lpString2       dd offset dword_403620  ; DATA XREF: CreateBOLE\_INI:loc\_403B5B r
PS\_\_\_\_\_\_:0040361C                             ; CreateBOLE\_INI+83 r ...
PS\_\_\_\_\_\_:00403620 dword\_403620    dd 1D151517h, 16081708h, 5050505h, 5551514Dh, 4C0A0A1Fh
PS\_\_\_\_\_\_:00403620                             ; DATA XREF: PS\_\_\_\_\_\_:lpString2 o
PS\_\_\_\_\_\_:00403620                 dd 460B4C4Ch, 554C564Dh, 51404B0Bh, 564C490Ah, 5D510B51h
PS\_\_\_\_\_\_:00403620                 dd 5050551h, 8 dup(5050505h), 37h dup(20202020h), 0
PS\_\_\_\_\_\_:00403B51                 xor     ecx, ecx        ; 0
PS\_\_\_\_\_\_:00403B53                 cmp     dword\_403618, ebx ; compare 4CH with 0
PS\_\_\_\_\_\_:00403B59                 jbe     short loc\_403B7D
PS\_\_\_\_\_\_:00403B5B
PS\_\_\_\_\_\_:00403B5B loc\_403B5B:                       ; CODE XREF: CreateBOLE\_INI+98 j
PS\_\_\_\_\_\_:00403B5B                 mov     eax, lpString2
PS\_\_\_\_\_\_:00403B60                 xor     byte ptr [eax+ecx], 25h
PS\_\_\_\_\_\_:00403B64                 add     eax, ecx        ; useless
PS\_\_\_\_\_\_:00403B66                 mov     eax, lpString2
PS\_\_\_\_\_\_:00403B6B                 add     eax, ecx
PS\_\_\_\_\_\_:00403B6D                 cmp     byte ptr [eax], 20h
PS\_\_\_\_\_\_:00403B70                 jnz     short loc\_403B74
PS\_\_\_\_\_\_:00403B72                 mov     [eax], bl     ；；BL==0
PS\_\_\_\_\_\_:00403B74
PS\_\_\_\_\_\_:00403B74 loc\_403B74:                       ; CODE XREF: CreateBOLE\_INI+8D j
PS\_\_\_\_\_\_:00403B74                 inc     ecx
PS\_\_\_\_\_\_:00403B75                 cmp     ecx, dword\_403618
PS\_\_\_\_\_\_:00403B7B                 jb      short loc\_403B5B
PS\_\_\_\_\_\_:00403B7D
```

此代码段的功能非常简单，循环将一长度为 `4ch` 的加密字符串解密，使用 `XOR` ，如果与 `25H` 异或后得到 `20H`，则用 `0` 替换，

```
0010 0101                       ；；（0010 0101）=25h
0000 0101    XOR               ；；（0000 0101）=5h
0010 0000                       ；；（0010 0000）=20h
```



也就是说，原字符串中数值为`05H`的字节均被`0`替换。

为了生动地将这一过程展献给读者，有了以下程序：

```
#include<windows.h>
#include<stdio.h>

void main()
{
  int i=0;
  DWORD String\[13\]={
                  0x1D151517, 0x16081708, 0x5050505, 0x5551514D, 0x4C0A0A1F,
                  0x460B4C4C, 0x554C564D, 0x51404B0B, 0x564C490A, 0x5D510B51,
                  0x5050551,0x5050505,0x5050505};
    char \*pString=NULL;
  char DestinationString\[0x4c\];
//////////////////////////////////////////////////////////////////////////////////////////
  CopyMemory(DestinationString,String,0x4c);
  for(i=0;i<0x4c;i++)
  {
    \*(DestinationString+i)=\*(DestinationString+i)⁰x25;
    if(\*(DestinationString+i)==0x20)
    {
      \*(DestinationString+i)=0;
    }
    if(\*(DestinationString+i)=='\\0')
    {
      printf("\\n");
    }
    else
    {
      printf("%c",\*(DestinationString+i));
    }
  }
  printf("\\n\\n");
  //////////////////////////////////////////////////////////////////////////////////
  CopyMemory(DestinationString,String,0x4c);
  for(i=0;i<0x4c;i++)
  {
    \*(DestinationString+i)=\*(DestinationString+i)⁰x25;
    if(\*(DestinationString+i)==0x20)
    {
      \*(DestinationString+i)=0;
    }
    if(\*(DestinationString+i)=='\\0')
    {
      printf("\\n");
      printf("%d\\n",i);
    }
    else
    {
      printf("%c",\*(DestinationString+i));
    }
  }
}
```

此程序有如下输出：

```
2008-2-3

8

9

10

11

http://iii.chsip.net/list.txt

41

42

43

44

45

46

47

48

49

50

51

i%%%遐 7%L2e%$%%%????
```

其中，`http://iii.chsip.net/list.txt`是`Downloader`要用到的列表文件，他会将解密生成的字符串写入`BOLE.INI`文件中。

好了，不要扯的太远，我们要调用的`API`名称字符串也可以用相同的方法来操作。

事实上，在`Win32`病毒技术中，`kernel32.dll`在进程地址空间中的基址，`GetProcAddress()`函数地址都是通过搜索`kernel32.dll`的内存空间得到的。

所以病毒体中的`API`调用相对比较隐蔽，但也完全可以应用`calling jmp`及字符串动态解密技术增强他们的抗分析能力。

由于病毒的破坏性，传播性较强，故其`MASM`源码不列出，请读者见谅。
