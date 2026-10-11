
```
MakeShellCode proc uses ebx esi edi, pShellCodeTo : LPVOID, dwExportFunctionRVA : DWORD, dwShellCodeRVA : DWORD
  mov ebx, pShellCodeTo
  assume ebx : ptr SHELLCODETO
  ;; 获取 JMP 的偏移,由于新节在代码的后面,所以想减后取补码并减去 0E9h 占用的
        ;; 1 个字节就等于要跳转的偏移.
  mov esi, dwShellCodeRVA
  sub esi, dwExportFunctionRVA
  neg esi
  dec esi
  lea edi, \[ebx\].pShellCode
  add edi, dword ptr \[ebx\].dwShellCodeSize
  mov al, 0E9h
  mov byte ptr \[edi\], al
  inc edi
  mov dword ptr \[edi\], esi
  add \[ebx\].dwShellCodeSize, 05h
  assume ebx : nothing
  ret
MakeShellCode endp
```

最后要说明一下 这个小程序分析脚本没有做什么错误处理。请按照正确的格式编写 呵呵。。 也没有备份原文件等功能,使用前请做好备份工作。`do.txt` 是脚本例子后面的 shellcode 是随便写的  

没有真正的用途... 附件代码中还有一个支持库的代码如果要编译程序请先把 `Support` 编译为 `lib` 的库.然后在与 `ExportHandler` 进行链接。希望大家喜欢。。。  

![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `ExportHandler_bin.rar` |
| --- | --- | --- |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `ExportHandler_src.rar` |
| --- | --- | --- |

| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) | | `do.txt` |
| --- | --- | --- |
