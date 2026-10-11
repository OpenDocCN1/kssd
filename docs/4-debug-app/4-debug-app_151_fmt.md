
# 逆向记事本看 UTF8 编码判断错误

> 原文：[`www.pediy.com/kssd/pediy10/101120.html`](https://www.pediy.com/kssd/pediy10/101120.html)

本人菜鸟，有不对的地方请各位大侠轻点拍砖，谢谢。打开记事本，输入“去”字，用 `ANSI` 方式保存。再重新打开，如果不出意外的话，看到的竟然是乱码。为了搞清楚“去”到底怎么变的身，开始了我非常不熟练的 `IDA`＋`WinDbg` 逆向，权当是练手。

首先用 `UE` 以十六进制的方式打开 `txt` 文件，其十六进制内容竟然是 `0xFEFF` `0x0225`。`0xFEFF` 是 `UNICODE` 编码的签名字节，而“去”的 `UNICODE` 编码是 `0x53bb`，此处的 `0x0225` 显然是错误的。起初怀疑是记事本在保存时，就以其他的编码当作 `UNICODE` 写入到了文件中。`WinDbg` 载入，跟踪到 `notepad!SaveFile`，仔细看了一下，保存的过程如下：`WideCharToMultiByte(GetACP(), ..., lpEditWCharBuf, ...);`然后直接 `WriteFile` 写入到 `txt` 文件中，并没有进行任何附加的操作。那么多余的 `0xFEFF` `UNICODE` 签字和 `0x0225` 哪儿来的呢？无奈自己手写了 `CreateFile`、`ReadFile`，终于看清，`txt` 文件里保存的确实是“去”的 `ANSI` 编码的两个字节：`0xA5C8`。原来 `UE` 在以 16 进制方式查看有问题的 `txt` 文件时，也犯了跟记事本一样的解码错误。

重新从记事本载入文件入手。从 `LoadFile` 跟踪到 `NpOpenDialogHookProc`，一直到 `fDetermineFileType`，终于找到了问题的根源：`notepad!IsTextUTF8` 函数。先说说 `fDetermineFileType`，其汇编代码不长，如下：

```asm
; int __stdcall fDetermineFileType(LPVOID lpBuffer,int cb)

_fDetermineFileType@8 proc near        ; CODE XREF: NpOpenDialogHookProc(x,x,x,x)+24A p

lpBuffer        = dword ptr  8
cb              = dword ptr  0Ch

                mov     edi, edi
                push    ebp
                mov     ebp, esp
                push    edi
                mov     edi, [ebp+cb]
                xor     eax, eax
                cmp     edi, 1
                jbe     short loc_10023FF
                push    esi
                mov     esi, [ebp+lpBuffer]
                movzx   ecx, word ptr [esi]
                cmp     ecx, 0BBEFh
                jz      short loc_10023F0
                cmp     ecx, 0FEFFh
                jz      short loc_10023D7
                cmp     ecx, 0FFFEh
                jz      short loc_10023EC
                push    edi             ; cb
                push    esi             ; lpBuffer
                call    _IsInputTextUnicode@8 ; IsInputTextUnicode(x,x)
                test    eax, eax
                jz      short loc_10023DC
loc_10023D7:
                xor     eax, eax
                inc     eax
                jmp     short loc_10023FE
loc_10023DC:
                push    edi
                push    esi
                call    _IsTextUTF8@8   ; IsTextUTF8(x,x)
                neg     eax
                sbb     eax, eax
                and     eax, 3
                jmp     short loc_10023FE
loc_10023EC:
                push    2
                jmp     short loc_10023FD
loc_10023F0:
                cmp     edi, 2
                jbe     short loc_10023FE
                cmp     byte ptr [esi+2], 0BFh
                jnz     short loc_10023FE
                push    3
loc_10023FD:
                pop     eax
loc_10023FE:
                pop     esi
loc_10023FF:
                pop     edi
                pop     ebp
                retn    8
_fDetermineFileType@8 endp
```

翻译成 C 代码更直观：

```c
int __stdcall fDetermineFileType(LPVOID lpBuffer,int cb)
{
  int iType = 0;
  WORD wSign = 0;
  if( cb <= 1 )
    return 0;
  wSign = *(PWORD)lpBuffer;
  switch( wSign )
  {
  case 0xBBEF:
    {
      if( cb >= 3 && (PBYTE)lpBuffer[3] == 0xBF)
        iType = 3;
    }
    break;
  case 0xFEFF:
    {
      iType = 1;
    }
    break;
  case 0xFFFE:
    {
      iType = 2;
    }
    break;
  default:
    {
      if( !IsInputTextUnicode( lpBuffer, cb ) )
      {
        if( IsTextUTF8( lpBuffer, cb ) )
          iType = 3;
      }
      else
        iType = 1;
    }
  }
  return iType;
}
```

当记事本打开任何一个文件时，首先会调用这个函数确定这个文本文件的编码方式。对于 `ANSI` 编码的 `GBXXXX` 标准的中文字符来说，直到 `IsTextUTF8` 函数返回 `FALSE`，`Notepad.exe` 才会以 `CP_ACP` 的方式调用 `MultiByteToWideChar`，直到这时中文字符才会被正确解析，也就是说，如果 `IsTextUTF8` 返回了 `TRUE`，那么 `ANSI` 的中文字符就会被当成 `UTF8` 编码转换成 `Unicode` 再显示出来。问题的根源就是 `IsTextUTF8` 了。看 `MS` 的家伙们是怎么写的 `IsTextUTF8`：

```asm
; __stdcall IsTextUTF8(x,x)
_IsTextUTF8@8   proc near               ; CODE XREF: fDetermineFileType(x,x)
Buffer          = dword ptr  8
Size            = dword ptr  0Ch

                mov     edi, edi
                push    ebp
                mov     ebp, esp
                push    esi
                xor     esi, esi
                xor     ecx, ecx
                inc     esi
                xor     edx, edx
                cmp     [ebp+Size], ecx
                jle     short FALSE
Loop:
                mov     eax, [ebp+Buffer]
                mov     al, [ecx+eax]
                test    al, al
                jns     short MayBeAscii
                xor     esi, esi
MayBeAscii:
                test    edx, edx
                jnz     short LeftBytes
                cmp     al, 80h
                jb      short LoopContinue
BytesCount:
                shl     al, 1
                inc     edx
                test    al, al
                js      short BytesCount
                dec     edx
                jz      short FALSE
                jmp     short LoopContinue
LeftBytes:
                and     al, 0C0h
                cmp     al, 80h
                jnz     short FALSE
                dec     edx
LoopContinue:
                inc     ecx
                cmp     ecx, [ebp+Size]
                jl      short Loop
                test    edx, edx
                ja      short FALSE
                test    esi, esi
                jz      short TRUE
FALSE:
                xor     eax, eax
                jmp     short QUIT
TRUE:
                xor     eax, eax
                inc     eax
QUIT:
                pop     esi
                pop     ebp
                retn    8
_IsTextUTF8@8   endp
```

还是翻译成 C：

```c
BOOL IsTextUTF8( LPSTR lpBuffer, int iBufSize )
{
  /*
  0zzzzzzz;
  110yyyyy, 10zzzzzz
  1110xxxx, 10yyyyyy, 10zzzzzz
  11110www, 10xxxxxx, 10yyyyyy, 10zzzzzz
  */
  int iLeftBytes = 0;
  BOOL bUtf8 = FALSE;
  if( iBufSize <= 0 )
    return FALSE;
  for( int i=0;i<iBufSize;i++)
  {
    char c = lpBuffer[i];
    if( c < 0 )        //至少有一个字节最高位被置位
      bUtf8 = TRUE;
    if( iLeftBytes == 0 )//之前尚无 UTF-8 编码的字符的前导字节，或者是下个字符。
    {
      if( c >= 0 )  //0000 0000 - 0100 0000
        continue;
      do//统计出高位连续的的个数
      {
        c <<= 1;
        iLeftBytes++;
      }while( c < 0 );
      iLeftBytes--;    //表示本字符的剩余字节的个数；
      if( iLeftBytes == 0 )//最高位是 10，不能作为 UTF-8 编码的字符的前导字节。
        return FALSE;
    }
    else
    {
      c &= 0xC0;       //1100 0000
      if( c != (char)0x80 )//1000 0000 对于合法的 UTF-8 编码，非前导字节的前两位必须为。
        return 0;
      else
        iLeftBytes--;
    }
  }
  if( iLeftBytes )
    return FALSE;
  return bUtf8;
}
```

根据国际相关组织的规定，`UTF8` 用 1-4 个字节来表示 `UNICODE`，对于绝对值在 `7F` 以上的字符，`UTF-8` 用两个及以上的字节表示，其中第一个字节的高位连续的 1 的个数，表示“用 `UTF8` 编码表示的字符的字节数”，此字符其他的字节必须是以 10 开头。如某 `UTF8` 编码的字符，第一个字节为 `110X XXXX`，那么说明表示此字符的 `UTF-8` 编码要用到两个字节，其第二个字节必须是 `10YY YYYY`。否则就是非法的 `UTF8` 编码。

这样说来，只要其 `ANSI` 编码值为 `0x8yCx` `0xByCx` 和 `0x8yDx－0xByDx` 字符，都被会解释成 `UTF-8` 编码：

```c
WORD wChar = 0;
char szBuf[3]={0};
for( wChar = 0x80C0; wChar < 0xBFDF; )
{
  *(PWORD)szBuf = wChar;
  cout<<szBuf;
  if( (wChar & 0xFF) == 0xDF )//11011111
  {
    wChar += 0x100;
    wChar &= 0xFFC0;
  }
  else
    wChar += 1;
}
```



这个编码范围的汉字，绝大多数我还是不认识的，但常用的还是有不少。以至于你写个“小丫头”、“泉水”、“来去”、“千十”等常用字保存，再用记事本打开，看到的都是乱码。

至此，“去”字引起的乱码现象，终于得到了完满解决。
