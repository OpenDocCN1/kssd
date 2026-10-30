
# [原创]360比赛第3题-我的解法

这题比较简单，大牛们不愿意发，我来发一下我的解法。

## 第3题—注册表保护挑战题

驱动加载后，创建一个内核态线程（driver-created thread）
```
0001054A xor eax, eax
0001054C push eax ; StartContext
0001054D push offset StartRoutine ; StartRoutine
00010552 push eax ; ClientId
00010553 push eax ; ProcessHandle
00010554 mov [ebp+ObjectAttributes.RootDirectory], eax
00010557 mov [ebp+ObjectAttributes.ObjectName], eax
0001055A mov [ebp+ObjectAttributes.SecurityDescriptor], eax
0001055D mov [ebp+ObjectAttributes.SecurityQualityOfService], eax
00010560 lea eax, [ebp+ObjectAttributes]
00010563 push eax ; ObjectAttributes
00010564 push 1F03FFh ; DesiredAccess
00010569 lea eax, [ebp+ThreadHandle]
0001056C push eax ; ThreadHandle
0001056D mov [ebp+ObjectAttributes.Length], 18h
00010574 mov [ebp+ObjectAttributes.Attributes], 200h
0001057B call ds:PsCreateSystemThread
```

线程的执行体为：
```
000104C8 ; void __stdcall StartRoutine(PVOID)
000104C8 StartRoutine proc near ; DATA XREF: sub_10542+B o
000104C8
000104C8 ObjectAttributes= OBJECT_ATTRIBUTES ptr -2Ch
000104C8 DestinationString= UNICODE_STRING ptr -14h
000104C8 Interval = LARGE_INTEGER ptr -0Ch
000104C8 Handle = dword ptr -4
000104C8
000104C8 mov edi, edi
000104CA push ebp
000104CB mov ebp, esp
000104CD sub esp, 2Ch
000104D0 or dword ptr [ebp+Interval+4], 0FFFFFFFFh
000104D4 push esi
000104D5 push offset SourceString ; "\\Registry\\Machine\\System\\360game"
000104DA lea eax, [ebp+DestinationString]
000104DD push eax ; DestinationString
000104DE mov dword ptr [ebp+Interval], 0FECED300h
000104E5 call ds:RtlInitUnicodeString
000104EB xor esi, esi
000104ED lea eax, [ebp+DestinationString]
000104F0 mov [ebp+ObjectAttributes.Length], 18h
000104F7 mov [ebp+ObjectAttributes.RootDirectory], esi
000104FA mov [ebp+ObjectAttributes.Attributes], 240h
00010501 mov [ebp+ObjectAttributes.ObjectName], eax
00010504 mov [ebp+ObjectAttributes.SecurityDescriptor], esi
00010507 mov [ebp+ObjectAttributes.SecurityQualityOfService], esi
0001050A
0001050A loc_1050A:
0001050A push esi ; Disposition
0001050B push esi ; CreateOptions
0001050C push esi ; Class
0001050D push esi ; TitleIndex
0001050E lea eax, [ebp+ObjectAttributes]
00010511 push eax ; ObjectAttributes
00010512 push 0F003Fh ; DesiredAccess
00010517 lea eax, [ebp+Handle]
0001051A push eax ; KeyHandle
0001051B call ds:ZwCreateKey
00010521 test eax, eax
00010523 jl short loc_1052E
00010525 push [ebp+Handle] ; Handle
00010528 call ds:ZwClose
0001052E
0001052E loc_1052E:
0001052E lea eax, [ebp+Interval]
00010531 push eax ; Interval
00010532 push esi ; Alertable
00010533 push esi ; WaitMode
00010534 call ds:KeDelayExecutionThread
0001053A jmp short loc_1050A
0001053A StartRoutine endp
```

该线程的作用是通过不断的循环，来创建`HKEY_LOCAL_MACHINE\System\360game`这个键值，并没有其他的保护删除的措施。

根据题目要求“程序只能运行于用户态，不能以任何形式直接执行`RING0`代码或修改`RING0`的内存”；那么我们只有在`RING3`下将上述的`StartRoutine`线程挂起，然后就可以直接删除了。

原理很清楚，实现就比较简单了，先找到系统的`system`进程，再找到其中的`StartRoutine`线程，然后将其挂起，之后将可以直接删除`HKEY_LOCAL_MACHINE\System\360game`这个键值了，详细实现见代码。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
