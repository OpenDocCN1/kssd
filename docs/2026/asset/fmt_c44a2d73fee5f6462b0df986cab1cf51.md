
# [原创]逆向分析ObRegisterCallbacks学习回调结构

首先调用`ObRegisterCallbacks`，查看所形成的结构是怎么样的。

```
OB\_CALLBACK\_REGISTRATION ObCallBack = { 0 };
OB\_OPERATION\_REGISTRATION ObOperation = { 0 };
ObCallBack.Version = ObGetFilterVersion();
ObCallBack.OperationRegistrationCount = 1;
ObCallBack.RegistrationContext = NULL;
ObCallBack.OperationRegistration = &ObOperation;
RtlInitUnicodeString(&ObCallBack.Altitude, L"22222");
ObOperation.ObjectType = PsProcessType;
ObOperation.Operations = OB\_OPERATION\_HANDLE\_CREATE | OB\_OPERATION\_HANDLE\_DUPLICATE;//OB\_OPERATION\_HANDLE\_CREATE = 1 OB\_OPERATION\_HANDLE\_DUPLICATE = 2
ObOperation.PreOperation = ObProcess; //ObProcess在本次实验中地址为0xfffff880`03df2480
ObOperation.PostOperation = NULL;
NTSTATUS Status;
Status = ObRegisterCallbacks(&ObCallBack, &Handle);
```

然后查看`PsProcessType`中所形成的链表结构

```
kd> dt \_OBJECT\_TYPE 0Xfffffa80`0ccb9d10
nt!\_OBJECT\_TYPE
+0x000 TypeList : \_LIST\_ENTRY [ 0xfffffa80`0ccb9d10 - 0xfffffa80`0ccb9d10 ]
+0x010 Name : \_UNICODE\_STRING "Process"
+0x020 DefaultObject : (null)
+0x028 Index : 0x7 ''
+0x02c TotalNumberOfObjects : 0x2b
+0x030 TotalNumberOfHandles : 0x10e
+0x034 HighWaterNumberOfObjects : 0x34
+0x038 HighWaterNumberOfHandles : 0x127
+0x040 TypeInfo : \_OBJECT\_TYPE\_INITIALIZER
+0x0b0 TypeLock : \_EX\_PUSH\_LOCK
+0x0b8 Key : 0x636f7250
+0x0c0 CallbackList : \_LIST\_ENTRY [ 0xfffff8a0`02385560 - 0xfffff8a0`02385560 ]
```
然后查看`CallbackList`
```
kd> dq 0xfffff8a0`02385560
fffff8a0`02385560 fffffa80`0ccb9dd0 fffffa80`0ccb9dd0
fffff8a0`02385570 00000001`00000003 fffff8a0`02385540
fffff8a0`02385580 fffffa80`0ccb9d10 fffff880`03df2480
fffff8a0`02385590 00000000`00000000 00000000`00000000
fffff8a0`023855a0 00320032`00320032 ffff0000`00000032
```

可以较为清晰的看出几个值，

偏移0处为`LIST_ENTRY`与类型对象中的链表相连，

偏移0x20处为类型对象的地址即`PsProcessType`，

偏移0x28处为`ObProcess`

接下来分析 `ObRegisterCallbacks`得到具体的结构对应项
