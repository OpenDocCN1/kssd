
# PE 文件格式学习小结 之 校验和的计算

> 原文：[`www.pediy.com/kssd/pediy10/61945.html`](https://www.pediy.com/kssd/pediy10/61945.html)

大部分 `PE` 文件都不使用文件头中的 `CheckSum` 域的校验和值，不过有些 `PE` 文件，如关键的系统服务程序文件以及驱动程序文件则该值必须正确，否则系统加载器将拒绝加载。

`PE` 头部的 `CheckSum` 可以使用 `Imagehlp.dll` 的导出函数 `CheckSumMappedFile` 计算，在 `MSDN` 中记录的这个函数的定义是这样的：

```
PIMAGE_NT_HEADERS CheckSumMappedFile(
  IN LPVOID BaseAddress,   
  IN DWORD FileLength, 
  OUT LPDWORD HeaderSum, 
  OUT LPDWORD CheckSum 
);
```

它的使用方法我在网上没有找到具体的例子！我们可以逆向一下 `LordPE` 看一下：

```
0040FF50  /$  8B15 C8DF4100 mov     edx, dword ptr [41DFC8]
0040FF56  |.  83EC 14       sub     esp, 14
0040FF59  |.  8D4424 00     lea     eax, dword ptr [esp]
0040FF5D  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]
0040FF61  |.  50            push    eax
0040FF62  |.  A1 CCDF4100   mov     eax, dword ptr [41DFCC]
0040FF67  |.  51            push    ecx
0040FF68  |.  52            push    edx
0040FF69  |.  50            push    eax
0040FF6A  |.  FF15 74904100 call    dword ptr [<IMAGEHLP.CheckSumMappedFile>]   ;  IMAGEHLP.CheckSumMappedFile
0040FF70  |.  85C0          test    eax, eax
0040FF72  |.  75 04         jnz     short 0040FF78
0040FF74  |.  83C4 14       add     esp, 14
0040FF77  |.  C3            retn
0040FF78  |>  8B4C24 00     mov     ecx, dword ptr [esp]
0040FF7C  |.  8B15 68BC4100 mov     edx, dword ptr [41BC68]                   ;  LordPE_h.0041BCB0
0040FF82  |.  51            push    ecx
0040FF83  |.  8D4424 0C     lea     eax, dword ptr [esp+C]
0040FF87  |.  52            push    edx                                         ; |Format => "%08lX"
0040FF88  |.  50            push    eax                                         ; |s
0040FF89  |.  FF15 C0924100 call    dword ptr [<USER32.wsprintfA>]             ; \wsprintfA
0040FF8F  |.  8B5424 24     mov     edx, dword ptr [esp+24]
0040FF93  |.  83C4 0C       add     esp, 0C
0040FF96  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]
0040FF9A  |.  51            push    ecx                                         ; /Text
0040FF9B  |.  68 00040000   push    400                                         ; |ControlID = 400 (1024.)
0040FFA0  |.  52            push    edx                                         ; |hWnd
0040FFA1  |.  FF15 0C924100 call    dword ptr [<USER32.SetDlgItemTextA>]        ; \SetDlgItemTextA
0040FFA7  |.  B8 01000000   mov     eax, 1
0040FFAC  |.  83C4 14       add     esp, 14
0040FFAF  \\.  C3            retn
```

这里我给出我的使用例子！

```
在工程中导入 imagehlp.lib:  #pragma comment(lib, "imagehlp.lib")

unsigned long LoadPEFile(char *FileName, char **Buffer)
{
  FILE *fp = fopen(FileName, "rb");
  fseek(fp, 0, SEEK_END);
  unsigned long len = ftell(fp);
  fseek(fp, 0, SEEK_SET);
  *Buffer = new char[len + 4];
  memset(*Buffer, 0x0, len + 4);
  unsigned long i = 0;
  while(i < len)
  {
    fread(*Buffer + i, 4, 1, fp);
    i+=4;
  }
  fclose(fp);
  return len;
}


void OnCheckSum() 
{
  char *Buffer = NULL;
  char *PEFile = "c:\\ps.exe";
  DWORD HeaderSum,CheckSum;
  unsigned long len = LoadPEFile(PEFile, &Buffer);
  ::CheckSumMappedFile(Buffer, len, &HeaderSum, &CheckSum);
  m_CheckSum.Format(_T("%.8X"),CheckSum);

  UpdateData(FALSE);
}
```

当然，在有些时候，我们并不能使用 `IMAGEHLP.CheckSumMappedFile` 函数，必须让我们自己动手来计算！

也可以在将该域清 0 后按照如下简单的等价算法计算：

如果 `PE` 文件大小是奇数字节，则以 0 补足，使之按偶数字节。将 `PE` 文件头的 `CheckSum` 域清 0，然后以两个字节为单位进行 `adc` 运算，最后和将该累加和同文件实际大小进行 `adc` 运算即得到校验和的值。

具体的计算方法，`hume` 老大早就写过一篇了~，这里我就不自己写了，就直接引用他的

下面的 `cal_checksum` 过程假设 `esi` 已经指向 `PE` 文件头，文件头部 `CheckSum` 域已经被清 0，`CF` 标志位已经被复位：

```
;调用示例： 
;clc 
;push pe_fileseize 
;call cal_checksum
cal_checksum: 
adc bp,word [esi] ;初始 esi 指向文件头，ebx 中保存的是文件大小
inc esi 
inc esi 
loop cal_checksum 
mov ebx,[esp+4] 
add ebp,ebx ;ebp 中存放的就是 PE 的校验和
ret 4 
```

具体的代码，大家可以看 `hume` 老大发表的帖子，我就不写了~
