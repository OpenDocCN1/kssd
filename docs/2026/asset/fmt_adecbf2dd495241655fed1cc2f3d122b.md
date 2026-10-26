
```
mov eax,dword ptr fs:[18]  IsDebuggerPresent
mov eax,dword ptr ds:[eax+30]
movzx eax,byte ptr ds:[eax+2]
mov eax,dword ptr ds:[7C88B358] GetACP
mov edi,edi   FreeEnvironmentStringsW
push ebp
mov ebp,esp
push dword ptr ss:[ebp+8]
mov eax,dword ptr fs:[18]
mov eax,dword ptr ds:[eax+30]
push 0
push dword ptr ds:[eax+18]
call dword ptr ds:[<&ntdll.RtlFreeHea>;
ntdll.RtlFreeHeap
movzx eax,al
pop ebp
mov eax,114A   CloseClipboard user32.dll
mov edx,7FFE0300
call dword ptr ds:[edx]
```
```
0043F5B6       $  FF15 F8624D00          call dword ptr ds:[<&kernel32.CreateToolhelp32Snapshot>]   ;
|\CreateToolhelp32Snapshot
0043F5BC       $  FF15 F4624D00          call dword ptr ds:[<&kernel32.Module32First>]              ;
|\Module32First
0043F5C2       $  FF15 F0624D00          call dword ptr ds:[<&kernel32.Module32Next>]               ;
|\Module32Next
0043F5C8       $  FF15 88624D00          call dword ptr ds:[<&kernel32.Process32First>]             ;
|\Process32First
0043F5CE       $  FF15 84624D00          call dword ptr ds:[<&kernel32.Process32Next>]              ;
\Process32Next
```
这里的`call`都要改为`jmp`，因为修复的时候出错了。

修复完之后`dump`就可以了。

`SetHandleCount`这个函数在输入表里面，可能是被删了，但我发现，`FreeEnvironmentStrings`有两个，所以将一个改为了`SetHandleCount`。

脱这个壳花了我一天多的时间。头都看大了。当初没用脚本。大部分都是手工活。不然几天都难弄好。

被加密的`call`总共有`0x150`之多，每一个调用的函数，前部分代码都被抽了，代码少的就抽空了。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
