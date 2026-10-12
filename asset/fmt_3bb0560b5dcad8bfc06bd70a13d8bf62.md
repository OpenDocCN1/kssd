
```
push    edx
pushfd
sub     esp, 20
lea     edx, dword ptr [esp]
push    edi
pop     dword ptr [edx+1C]
push    eax
pop     dword ptr [edx]
or      eax, edi
mov     dword ptr [edx+14], ebp
push    esi
pop     dword ptr [edx+18]
mov     esi, 42AFF2
push    ebx
pop     dword ptr [edx+C]
mov     ebx, 4716BE
push    ecx
pop     dword ptr [edx+4]
mov     edi, esp
lea     edi, dword ptr [edi+2C]
push    edi
push    edx
push    edx
pop     edi
add     edi, 20
mov     edi, dword ptr [edi]
push    edi
push    edx
pop     edi
lea     edi, dword ptr [edi+28]
push    dword ptr [edi]
pop     edi
lea     edi, dword ptr [edi-5]
lea     edi, dword ptr [edi+EBC]
push    edi
push    0
pop     edi
push    dword ptr fs:[edi]
push    1210620
lea     edx, dword ptr [1261F80]
call    edx
```

分析一下这段代码的功能可以知道，其实功能非常简单，只是保存现场后 `call 1261F80`，并把 6 个参数放入了堆栈：

- `[-2C]=esp+4` 返回地址指针
- `[-30]=esp-28` 返回时的堆栈-28
- `[-34]=[-8]` 返回时的寄存器状态
- `[-38]=[-0]-5+EBC` `[-0]-5=CALL XXXXXXXX` 的后一个字节，`EBC=CurrentProcessID`
- `[-3C]=fs:[0]` 异常入口
- `[-40]=1210620` 基本数据基址

其中加方括号的是相对于入口的堆栈偏移。

再来看看 `1261F80` 处的代码：

```
代码:
```
