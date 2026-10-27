
## 内存转储分析

```
kd> dq 0xfffff8a0`02385560
fffff8a0`02385560 fffffa80`0ccb9dd0 fffffa80`0ccb9dd0
fffff8a0`02385570 00000001`00000003 fffff8a0`02385540
fffff8a0`02385580 fffffa80`0ccb9d10 fffff880`03df2480
fffff8a0`02385590 00000000`00000000 00000000`00000000
fffff8a0`023855a0 00320032`00320032 ffff0000`00000032
```

可以看出``0xfffff8a0`02385560``为结构的``0x20``偏移处，取出结构首地址。

```
kd> dq 0xfffff8a0`02385540
fffff8a0`02385540 00000000`00010100 00000000`00000000
fffff8a0`02385550 00000000`000a000a fffff8a0`023855a0
fffff8a0`02385560 fffffa80`0ccb9dd0 fffffa80`0ccb9dd0
fffff8a0`02385570 00000001`00000003 fffff8a0`02385540
fffff8a0`02385580 fffffa80`0ccb9d10 fffff880`03df2480
fffff8a0`02385590 00000000`00000000 00000000`00000000
fffff8a0`023855a0 00320032`00320032 ffff0000`00000032
```

可以看到除了个别地方（偏移`0x34`和偏移`0x2`处），其余都和代码所对应。继续分析接下来的代码。

```
PAGE:00000001404B6ED6 cmp eax, ebx
PAGE:00000001404B6ED8 mov esi, eax
PAGE:00000001404B6EDA jl short loc_1404B6F15
PAGE:00000001404B6EDC mov eax, 1
PAGE:00000001404B6EE1 add rbp, 20h
PAGE:00000001404B6EE5 add r13, 40h
PAGE:00000001404B6EE9 add [rdi+CallBackStr.flag], ax ; 将结构的偏移2处 置为1
PAGE:00000001404B6EED movzx ecx, [r12+OB_CALLBACK_REGISTRATION.OperationRegistrationCount]
PAGE:00000001404B6EF3 add r14d, eax
PAGE:00000001404B6EF6 cmp r14d, ecx
PAGE:00000001404B6EF9 jb loc_1404B6E43 ; 如果Count>1将会进入循环再次进入填充插入环节
PAGE:00000001404B6EFF jmp short loc_1404B6F0D
PAGE:00000001404B6F0D loc_1404B6F0D:
PAGE:00000001404B6F0D cmp esi, ebx
PAGE:00000001404B6F0F jge loc_1404B6FDD ; 判断插入链表的返回值 如果大于等于0 跳转
PAGE:00000001404B6FDD loc_1404B6FDD:
PAGE:00000001404B6FDD
PAGE:00000001404B6FDD cmp bx, [rdi+CallBackStr.flag] ; 如果偏移2处 为负数跳转
PAGE:00000001404B6FE1 jnb short loc_1404B7004
PAGE:00000001404B6FE3 lea rcx, [rdi+34h] ; 取出偏移0x34处地址
PAGE:00000001404B6FE7 mov r15d, 1
PAGE:00000001404B6FED
PAGE:00000001404B6FED loc_1404B6FED:
PAGE:00000001404B6FED or [rcx], r15d ; 此处便为设置偏移0x34处
PAGE:00000001404B6FF0 movzx eax, [rdi+CallBackStr.flag]
PAGE:00000001404B6FF4 add ebx, r15d
PAGE:00000001404B6FF7 add rcx, 40h
PAGE:00000001404B6FFB cmp ebx, eax
PAGE:00000001404B6FFD jb short loc_1404B6FED ; 此处便为设置偏移0x34处
PAGE:00000001404B6FFF mov r15, [rsp+48h+arg_8]
PAGE:00000001404B7004
PAGE:00000001404B7004 loc_1404B7004:
PAGE:00000001404B7004 mov [r15], rdi
PAGE:00000001404B7007 loc_1404B7007:
PAGE:00000001404B7007 mov eax, esi
PAGE:00000001404B7009
PAGE:00000001404B7009 loc_1404B7009:
PAGE:00000001404B7009
PAGE:00000001404B7009 mov rbx, [rsp+48h+arg_0]
PAGE:00000001404B700E mov rbp, [rsp+48h+arg_10]
PAGE:00000001404B7013 mov rsi, [rsp+48h+arg_18]
PAGE:00000001404B7018 add rsp, 20h
PAGE:00000001404B701C pop r15
PAGE:00000001404B701E pop r14
PAGE:00000001404B7020 pop r13
PAGE:00000001404B7022 pop r12
PAGE:00000001404B7024 pop rdi
PAGE:00000001404B7025 retn
```

以上便是对`ObRegisterCallbacks`的分析，如果存在错误，希望能予以指出。

## 调用 ObRegisterCallbacks 形成的结构

首先调用`ObRegisterCallbacks`，查看所形成的结构是怎么样的。

```
OB_CALLBACK_REGISTRATION ObCallBack = { 0 };
OB_OPERATION_REGISTRATION ObOperation = { 0 };

ObCallBack.Version = ObGetFilterVersion();
ObCallBack.OperationRegistrationCount = 1;
ObCallBack.RegistrationContext = NULL;
ObCallBack.OperationRegistration = &ObOperation;
RtlInitUnicodeString(&ObCallBack.Altitude, L"22222");

ObOperation.ObjectType = PsProcessType;
ObOperation.Operations = OB_OPERATION_HANDLE_CREATE | OB_OPERATION_HANDLE_DUPLICATE; // OB_OPERATION_HANDLE_CREATE = 1 OB_OPERATION_HANDLE_DUPLICATE = 2
ObOperation.PreOperation = ObProcess; // ObProcess在本次实验中地址为0xfffff880`03df2480
ObOperation.PostOperation = NULL;

NTSTATUS Status;
Status = ObRegisterCallbacks(&ObCallBack, &Handle);
```

## 查看 PsProcessType 中的链表结构

然后查看`PsProcessType`中所形成的链表结构。

```
kd> dt _OBJECT_TYPE 0Xfffffa80`0ccb9d10
nt!_OBJECT_TYPE
+0x000 TypeList         : _LIST_ENTRY [ 0xfffffa80`0ccb9d10 - 0xfffffa80`0ccb9d10 ]
+0x010 Name             : _UNICODE_STRING "Process"
+0x020 DefaultObject    : (null)
+0x028 Index            : 0x7 ''
+0x02c TotalNumberOfObjects : 0x2b
+0x030 TotalNumberOfHandles : 0x10e
+0x034 HighWaterNumberOfObjects : 0x34
+0x038 HighWaterNumberOfHandles : 0x127
+0x040 TypeInfo         : _OBJECT_TYPE_INITIALIZER
+0x0b0 TypeLock         : _EX_PUSH_LOCK
+0x0b8 Key              : 0x636f7250
+0x0c0 CallbackList     : _LIST_ENTRY [ 0xfffff8a0`02385560 - 0xfffff8a0`02385560 ]
```

然后查看`CallbackList`：

```
kd> dq 0xfffff8a0`02385560
fffff8a0`02385560 fffffa80`0ccb9dd0 fffffa80`0ccb9dd0
fffff8a0`02385570 00000001`00000003 fffff8a0`02385540
fffff8a0`02385580 fffffa80`0ccb9d10 fffff880`03df2480
fffff8a0`02385590 00000000`00000000 00000000`00000000
fffff8a0`023855a0 00320032`00320032 ffff0000`00000032
```

可以较为清晰地看出几个值：

- 偏移`0`处为`LIST_ENTRY`，与类型对象中的链表相连；
- 偏移`0x20`处为类型对象的地址，即`PsProcessType`；
- 偏移`0x28`处为`ObProcess`。

接下来分析`ObRegisterCallbacks`，得到具体的结构对应项。
