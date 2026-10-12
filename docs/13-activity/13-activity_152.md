
# 我的第四题解法

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-273.htm`](https://www.pediy.com/kssd/pediy09/pediy09-273.htm)
>
> 思路和前面提出的解法不同，完全不关心解密函数位置。所以贴一下。
>
> 个人感觉是非常投机取巧型的，和出题者本意背道而驰，而且代码写的相当 #%^$%\*\*^&$，完全是能工作就行。![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ccfer 的解法才是理想解法。
>
> 解法依据很简单，基于观察到两个样本 `exe` 在堆栈中构造加密字符串的 `MOV BYTE PTR SS:[ESP+xxx],0xx` 指令是连续的。就是说，这些指令都是相邻的，或者中间间隔 1,2 条其它指令。所以找到所有连续的序列，再取出指令里的立即数就能得到加密字符串。
>
> 还能够根据 `esp` + `xxx` 里的 `xxx` 处理字符串被打乱顺序，或者重复写的情况，因为时间关系，没有处理。
>
> btw, `windows` 下编程太差。怎么读文件头疼了好久，没研究会怎么在 `windows` 下用内存映射方式读文件，最后读文件还是用标准 `c` 库的 `API`。![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

代码:

```c
static int read_file(unsigned char **ret,const char *name)
{
  unsigned char *buf;
  int fd;
  struct stat st;
  fd = _open(name,_O_RDONLY);
  if (fd < 0)
  {
    fprintf(stderr,"open failed\n");
    return -1;
  }
  fstat(fd,&st);
  buf = (unsigned char*)malloc(st.st_size);
  _read(fd,buf,st.st_size);
  close(fd);
  *ret = buf;
  return st.st_size;
}
static void decode(FILE *fp,unsigned char *buf,int *addrs,int start,int end)
{
  int i,j,pos;
  unsigned int v;
  unsigned char data[256];
  if ( end - start < 7 )
    return;
  for (j = 0,i = start; i <= end; i++)
  {
    /// 计算字符在指令中的偏移。
    v = *(unsigned int *)(buf + addrs[i]);
    v <<= 8;
    if (v == 0x2484c600)
      pos = 7;
    else if (v == 0x2444c600)
    {
      pos = 4;
    }
    else
      continue;
    data[j] = buf[ addrs[i] + pos ];
    j++;
  }
  data[j] = 0;
  decrypt(data);
  fprintf(fp,"%s\n",(char*)data);
}
#define MAX_ITEMS  128000
int main(int argc,char **argv)
{
  int i,j;
  int len;
  int start;
  unsigned char *file;
  int *addrs,*deltas;
  FILE *fp;
  if (argc == 1)
  {
    fprintf(stderr,"Usage: %s filename.exe\n",argv[0]);
    return -1;
  }
  len = read_file(&file,argv[1]);
  if (len <= 0)
  {
    fprintf(stderr,"read file failed\n");
    return -1;
  }
  addrs = (int*)malloc(4 * MAX_ITEMS);
  deltas = (int*)malloc(4 * MAX_ITEMS);
  /// 搜索所有 MOV BYTE PTR SS:[ESP+xxx],0xx 指令的地址
  len -= 4;
  for (i = 0,j = 0; i < len; i++)
  {
    unsigned int v = *(unsigned int *)(file + i);
    v <<= 8;
    /// MOV BYTE PTR SS:[ESP+xxx],0xx
    if ( v == 0x2484c600 || v == 0x2444c600 )
    {
      addrs[j] = i;
      if (j)
      {
        deltas[j - 1] = i - addrs[j - 1];
      }
      j++;
      if (j >= MAX_ITEMS)
      {
        fprintf(stderr,"Too many items, break\n");
        break;
      }
    }
  }
  deltas[j - 1] = 99;
  fp = fopen("out.txt","w");
  /// 这里是关键算法，找出“连续的”指令序列
  start = -1;
  for (i = 0; i < j; i++)
  {
    /// 指令间距 > 20 意味着一个连续指令序列结束
    if (deltas[i] > 20)
    {
      if (start >= 0)
      {
        decode(fp,file,addrs,start,i);
        start = -1;
      }
    }
    /// 指令间距 >=5 并且 <= 20 意味着连续
    else if ( deltas[i] >= 5 )
    {
      if (start >= 0)
      {
        continue;
      }
      start = i;
    }
    else
    {
      start = -1;
    }
  }
  free(file);
  free(addrs);
  free(deltas);
  fclose(fp);
  return 0;
}
```

这个好啊，不用定位解密函数的位置

我就担心自己找不到解密函数就 0 分了

本来想省掉堆栈，没想到中文的有反序，只好写了个正确率不高的：

```asm
.386p
.model                  flat, stdcall
option                  casemap : none
include                 windows.inc
include                 kernel32.inc
include                 user32.inc
includelib              kernel32.lib
includelib              user32.lib
find                    macro   r, p:VARARG
                        local   __cycle, __next, __back, __done
                        local   ofs, x
                        ofs     = 0
                        pushfd
__cycle:                for     x, <p>
                        cmp     byte ptr [r+ofs], x
                        jnz     __next
                        ofs     = ofs + 1
                        endm
                        jmp     __done
__next:                 test    dword ptr [esp], 10000000000b
                        jnz     __back
                        inc     r
                        jmp     __cycle
__back:                 dec     r
                        jmp     __cycle
__done:                 popfd
                        endm
.data
input                   db      'shit.exe', 0
output                  db      'fuck.txt',0
.code
include                 shit.inc
include                 mlde32.inc
start:                  push    offset output
                        push    offset input
                        call    strsux
                        call    ExitProcess
strsux                  proc    ifile, ofile
                        local   ibuf
                        local   obuf
                        local   isize
                        local   osize
                        local   pehdr
                        local   decode
                        local   xbl
                        local   xdl
                        local   fhandle
                        local   fuckr
                        pushad
                        push    1
                        push    0
                        push    ifile
                        call    LoadLibraryExA
                        test    eax, eax
                        mov     [ibuf], eax
                        jz      __exit
                        add     eax, [eax+3ch]
                        mov     [pehdr], eax
                        mov     edx, [eax+50h]
                        mov     [isize], edx
                        push    4
                        push    1000h
                        push    [isize]
                        push    0
                        call    VirtualAlloc
                        test    eax, eax
                        mov     [obuf], eax
                        jz      __kill
                        lea     edx, [esp+5*4]          ; pushad.edx
                        push    edx
                        push    4
                        push    [isize]
                        push    [ibuf]
                        call    VirtualProtect
                        test    eax, eax
                        jz      __kill
                        push    0
                        push    80h
                        push    2
                        push    0
                        push    1
                        push    0c0000000h
                        push    ofile
                        call    CreateFileA
                        test    eax, eax
                        mov     [fhandle], eax
                        jz      __kill
                        mov     esi, [ibuf]
                        find    esi, 52h, 0c6h, 04h, 2ah, 00h, 0e8h
                        std
                        find    esi, 8bh, 54h, 24h, 04h
                        cld
                        mov     [decode], esi
                        mov     esi, [ibuf]
                        mov     ebx, [isize]
                        add     ebx, esi
                        sub     ebx, 3                          ; for safe
```



```
__xref_cycle:           cmp     esi, ebx  
                        jge     __xref_end  
                        mov     eax, [esi]  
                        cmp     al, 0e8h  
                        jnz     __xref_next  
                        mov     eax, esi  
                        call    __parse_call  
                        cmp     eax, [decode]  
                        jnz     __xref_next  
                        std  
                        find    esi, 90h, 90h  
                        cld  
                        call    __disasm  
__xref_next:            inc     esi  
                        jmp     __xref_cycle  
__xref_end:  
__kill:                 push    [ofile]  
                        call    CloseHandle  
                        push    8000h  
                        push    0  
                        push    [obuf]  
                        call    VirtualFree  
                        push    [ibuf]  
                        call    FreeLibrary  
__exit:                 popad  
                        ret  
; ------------------------------------------------------------------------------  
; input: esi  
;  
; restart:  
;  
; 8D0424                lea     eax, dword ptr [esp]            xxxxxxxxx0010010000xxx10010001101  
; 8D4424 12             lea     eax, dword ptr [esp+12]         xxxxxxxxx0010010001xxx10010001101  
; 8D8424 78563412       lea     eax, dword ptr [esp+12345678]    xxxxxxxxx0010010010xxx10010001101  
;  
; grab r8:  
;  
; 32DB                  xor     bl, bl  
; B3 12                 mov     bl, 12  
; 33DB                  xor     ebx, ebx  
;  
; 32D2                  xor     dl, dl  
; B2 12                 mov     dl, 12  
; 33D2                  xor     edx, edx                        ?  
;  
; store:  
;  
; 881C24                mov     byte ptr [esp], bl  
; 885C24 12             mov     byte ptr [esp+12], bl  
; 889C24 78563412       mov     byte ptr [esp+12345678], bl  
; 881424                mov     byte ptr [esp], dl  
; 885424 12             mov     byte ptr [esp+12], dl  
; 889424 78563412       mov     byte ptr [esp+12345678], dl  
; C64424 12 12          mov     byte ptr [esp+12], 12  
; C68424 78563412 12    mov     byte ptr [esp+12345678], 12  
;  
; output:  
;  
; E8xxxxxxxx            call    decode  
;  
__disasm:               pushad  
                        call    __restart  
__disasm_cycle:         cmp     esi, ebx  
                        jge     __disasm_end  
                        
                        mov     eax, [esi]  
;                       mov     edx, 00000000111111111100011111111111b  
;                       and     edx, eax  
;                       cmp     edx, 00000000001001000000010010001101b  
;                       jz      __x0  
;                       mov     edx, 00000000111111111100011111111111b  
;                       and     edx, eax  
;                       cmp     edx, 00000000001001000100010010001101b  
;                       jz      __x0  
;                       mov     edx, 00000000111111111100011111111111b  
;                       and     edx, eax  
;                       cmp     edx, 00000000001001001000010010001101b  
;                       jz      __x0  
                        mov     edx, 0ffffffh  
                        and     edx, eax  
                        cmp     edx, 241c88h                    ; mov [esp], bl  
                        jz      __x1_0  
                        cmp     edx, 245c88h                    ; mov [esp+?], bl  
                        jz      __x1_0  
                        cmp     edx, 249c88h                    ; mov [esp+????], bl  
                        jz      __x1_0  
                        cmp     edx, 2444c6h                    ; mov [esp+?], ?  
                        jz      __x1_1  
                        cmp     edx, 2484c6h                    ; mov [esp+????], ?  
                        jz      __x1_2  
                        cmp     edx, 241488h  
                        jz      __x1_3  
                        cmp     edx, 245488h  
                        jz      __x1_3  
                        cmp     edx, 249488h  
                        jz      __x1_3  
                        cmp     al, 0e8h                        ; call  
                        jz      __x2  
                        cmp     ax, 0db33h                      ; xor ebx, ebx  
                        jz      __x3_0  
                        cmp     ax, 0db32h                      ; xor bl, bl  
                        jz      __x3_0  
                        cmp     al, 0b3h                        ; mov bl, ?  
                        jz      __x3_1  
                        cmp     ax, 0d233h                      ; xor edx, edx  
                        jz      __x4_0  
                        cmp     ax, 0d232h                      ; xor dl, dl  
                        jz      __x4_0  
                        cmp     al, 0b2h                        ; mov dl, ?  
                        jz      __x4_1  
                        cmp     al, 0c2h                        ; retn ?  
                        jz      __disasm_end  
                        cmp     al, 0c3h                        ; ret  
                        jz      __disasm_end                         
__disasm_next:          push    esi  
                        call    mlde32  
                        pop     esi  
                        add     esi, eax  
                        jmp     __disasm_cycle  
;__x0:                   jmp     __disasm_next                   ; some instruction isn't ordered :(  
__x1_0:                 mov     eax, [xbl]                      ; store byte  
                        jmp     __store_byte  
__x1_1:                 mov     al, [esi+4]  
                        and     [fuckr], 0  
                        jmp     __store_byte  
__x1_2:                 mov     al, [esi+7]  
                        and     [fuckr], 0  
__store_byte:           cmp     [fuckr], 0                      ; r8 cannot set at start  
                        jnz     __disasm_next  
                        stosb  
                        jmp     __disasm_next  
__x1_3:                 mov     eax, [xdl]  
                        jmp     __store_byte  
__x2:                   mov     eax, esi                        ; decode  
                        call    __parse_call  
                        cmp     eax, [decode]  
                        jnz     __disasm_next  
                        push    [obuf]  
                        call    deshit  
                        pop     eax  
                        
                        push    [obuf]  
                        call    lstrlenA  
                        mov     [osize], eax  
                        add     eax, [obuf]  
                        mov     word ptr [eax], 0a0dh            ; crlf  
                        add     [osize], 2  
                        push    FILE_END  
                        push    0  
                        push    0  
                        push    [fhandle]  
                        call    SetFilePointer  
                        push    0  
                        push    esp  
                        push    [osize]  
```



```
                push    [obuf]
                push    [fhandle]
                call    WriteFile
                call    __restart
                mov     byte ptr [esi], 0e9h            ; disable xref scannin'
                jmp     __disasm_next
__x3_0:                 xor     eax, eax                        ; grab bl
                jmp     __grab_bl
__x3_1:                 mov     al, [esi+1]
__grab_bl:              movzx   eax, al
                mov     [xbl], eax
                jmp     __disasm_next
__x4_0:                 xor     eax, eax
                jmp     __grab_dl
__x4_1:                 mov     al, [esi+1]
__grab_dl:              movzx   eax, al
                mov     [xdl], eax
                jmp     __disasm_next
__disasm_end:           popad
                retn
__restart:              mov     edi, [obuf]                     ; and restart
                pusha
                mov     ecx, [isize]
                xor     eax, eax
                rep     stosb
                popa
                or      [fuckr], 1
                retn
; -----------------------------------------------------------------------------
__parse_call:           add     eax, [eax+1]
                add     eax, 5
                retn
strsux                  endp
                        end     start
```
4dd466d0e1e0cc123afbe504da2462ca
