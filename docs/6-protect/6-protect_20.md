
# getprocaddress asm 版本

> 原文：[`www.pediy.com/kssd/pediy10/83395.html`](https://www.pediy.com/kssd/pediy10/83395.html)

近日，钱老师讲 `DLL` 导出表，课堂用 `ASM` 代码实现 `GetProcAddr`，惜课堂时间太短，难免仓促，我现下把代码补全，算是狗尾续貂了。嘿嘿

```asm
MyGetProcAddr proc hMod:DWORD,ProcName:DWORD

  mov ebx,hMod

  assume ebx:ptr IMAGE_DOS_HEADER

  add ebx,[ebx].e_lfanew

  add ebx,4

  add ebx,sizeof IMAGE_FILE_HEADER

  assume ebx:nothing

  assume ebx:ptr IMAGE_OPTIONAL_HEADER32

  lea ebx,[ebx].DataDirectory

  assume ebx:ptr IMAGE_DATA_DIRECTORY

  mov ebx,[ebx].VirtualAddress

  add ebx,hMod;

  assume ebx:ptr IMAGE_EXPORT_DIRECTORY

  ;如果调用参数为名称。

  .if ProcName > 0ffffh;

    xor edx,edx  ;串指令会修改 ECX,这里用 EDX 做计数器。

    .while edx < [ebx].NumberOfNames

      mov edi,ProcName          ;获取字串长度是否。

      xor al,al

      mov ecx,-1

      repnz scasb

      not ecx                  ;ECX 为字串长度，包含\0

      ;字串比较

      mov edi,ProcName

      mov esi,[ebx].AddressOfNames

      add esi,hMod

      lea esi,[esi+4*edx]

      mov esi,[esi]

      add esi,hMod

      repz cmpsb

      lahf

      .if ecx == 0

        .break

      .endif

      inc edx

    .endw

    .if edx == [ebx].NumberOfNames  ;表示没找到

      mov eax,0

      ret

    .endif

    mov eax,[ebx].AddressOfNameOrdinals

    add eax,hMod;

    movzx eax,WORD ptr [eax+2*edx]

  ;序号方式调用。

  .else

    mov eax,ProcName

    sub eax,[ebx].nBase        ;从序号到索引

    .if eax < 0           ;判断是序号否在范围内

      xor eax,eax

      ret

    .endif

    mov edx,[ebx].NumberOfFunctions

    .if eax >edx

       xor eax,eax

       ret

    .endif

  .endif

  ;根据序号找寻函数地址

ORDER:

  mov edx,[ebx].AddressOfFunctions

  add edx,hMod;

  mov eax,[edx+4*eax]

  add eax,hMod

  ret

MyGetProcAddr endp
```

此段代码关键点在于：

1. 不能破坏堆栈环境
2. 不能动态调用 `DLL` 里的函数

在做溢出的时候是非常有用的，实乃 “居家旅行，杀人灭口”之必备。
