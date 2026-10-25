
# XP系统程序中开头的`MOV EDI,EDI`指令的解释

XP系统程序中开头的`MOV EDI,EDI`指令的解释：

在VS .NET 2003的`VC7\INCLUDE`目录中的`listing.inc`文件中定义了１到７个字节的无破坏性`NOP`操作的宏

`MOV EDI,EDI`就是两个字节的`NOP`

在程序中与`NOP`指令的意义相同。

为什么要用`MOV EDI,EDI`而不用两个`NOP`？

我的理解是：

用两个`NOP`指令耗费的CPU时钟周期要比用`MOV EDI,EDI`指令要长，为了提高效率，就采用了`MOV EDI,EDI`

## `listing.inc`文件的内容

```
;; LISTING.INC

;;

;; This file contains assembler macros and is included by the files created

;; with the -FA compiler switch to be assembled by MASM (Microsoft Macro

;; Assembler).

;;

;; Copyright (c) 1993, Microsoft Corporation. All rights reserved.

;; non destructive nops

npad macro size

if size eq 1

  nop

else

if size eq 2

   mov edi, edi

else

  if size eq 3

    ; lea ecx, [ecx+00]

    DB 8DH, 49H, 00H

  else

   if size eq 4

     ; lea esp, [esp+00]

     DB 8DH, 64H, 24H, 00H

   else

    if size eq 5

      add eax, DWORD PTR 0

    else

     if size eq 6

       ; lea ebx, [ebx+00000000]

       DB 8DH, 9BH, 00H, 00H, 00H, 00H

     else

      if size eq 7

        ; lea esp, [esp+00000000]

        DB 8DH, 0A4H, 24H, 00H, 00H, 00H, 00H

      else

        %out error: unsupported npad size

        .err

      endif

     endif

    endif

   endif

  endif

endif

endif

endm

;; destructive nops

dpad macro size, reg

if size eq 1

  inc reg

else

  %out error: unsupported dpad size

  .err

endif

endm
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
