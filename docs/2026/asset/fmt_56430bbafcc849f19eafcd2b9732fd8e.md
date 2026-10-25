
``` 
  mov eax, dword ptr [eax]

:004AD6FD E88238F8FF              call

00430F84

:004AD702 E951060000              jmp 004ADD58

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004AD6B9(C), :004AD6F4(C)

|

\* Possible StringData Ref from Data Obj ->"Load Database"

            |

:004AD707 B868DE4A00              mov eax,

004ADE68

:004AD70C E8E7F0FDFF              call

0048C7F8

:004AD711 8B45FC

mov eax, dword ptr [ebp-04]

:004AD714 80B86053000000          cmp byte ptr [eax+00005360],

00

:004AD71B 7417

  je 004AD734

:004AD71D A12C6F4B00              mov eax,

dword ptr [004B6F2C]

:004AD722 8B00

  mov eax, dword ptr [eax]

:004AD724 8B80E4010000            mov eax, dword

ptr [eax+000001E4]

\* Possible StringData Ref from Data Obj ->"Loading database..."

            |

:004AD72A BA80DE4A00              mov edx,

004ADE80

:004AD72F E88C57F9FF              call

00442EC0
```

将`:004AD6B9 754C jne 004AD707`改为`:744C je 004ad707`。运行一下，果然注册成功，再也不会出现注册提示框了。
