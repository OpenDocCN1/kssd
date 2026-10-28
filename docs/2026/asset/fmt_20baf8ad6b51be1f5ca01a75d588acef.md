
`+0x060 DeviceQueue`      : `\_KDEVICE\_QUEUE`

`+0x074 Dpc`              : `\_KDPC`

`+0x094 ActiveThreadCount` : `0`

`+0x098 SecurityDescriptor` : `(null)`

`+0x09c DeviceLock`       : `\_KEVENT`

`+0x0ac SectorSize`       : `0`

`+0x0ae Spare1`           : `1`

`+0x0b0 DeviceObjectExtension` : `0x822d7658` `\_DEVOBJ\_EXTENSION`

`+0x0b4 Reserved`         : `(null)`

从这里我们知道了，这个驱动程序`i8042prt`创建了两个设备，这两个设备互相形成链式结构，这个链表以`NULL`表示

结束，驱动对象通过 `+04 struct \_DEVICE\_OBJECT \*DeviceObject` 可以找到这个链。设备对象通过 `+0c struct \_DEVICE\_OBJECT \*NextDevice` 链在一起。设备对象通过 `+08 struct \_DRIVER\_OBJECT \*DriverObject` 可以找到创建它的驱动的驱动对象。

要想获得更多的信息，还要了解一下设备栈的概念，一个物理设备的驱动任务，通常由几个驱动程序一层一层的共同完成。每层一个设备对象，他们联系在一起，组成一个设备栈。一个设备栈上的设备对象，从上自下，联系在一起。从下自上，也联系在一起。还以上面的那个键盘驱动为例，继续探索下设备栈的相关东西。

### 设备栈

使用命令`!devstack`可以观看设备栈中的内容

```
kd> !devstack 822d7310
  !DevObj   !DrvObj            !DevExt   ObjectName
  8212a030  \Driver\Kbdclass   8212a0e8  KeyboardClass0
  8212ceb8  \Driver\DKbFltr    8212cf70
> 822d7310  \Driver\i8042prt   822d73c8
  822dc888  \Driver\ACPI       8239bea0  00000076
!DevNode 82130ca8 :
  DeviceInst is "ACPI\PNP0303\4&cd4a22c&0"
  ServiceName is "i8042prt"
```

其中只有两个驱动对象有名字`\Driver\Kbdclass` ，`\Driver\ACPI`

我们从上往下依次看

#### 第一个

```
kd> !devobj 8212a030
Device object (8212a030) is for:
 KeyboardClass0 \Driver\Kbdclass DriverObject 81ebf858
Current Irp 00000000 RefCount 0 Type 0000000b Flags 00002044
Dacl e13b5f84 DevExt 8212a0e8 DevObjExt 8212a1c8
ExtensionFlags (0000000000)
AttachedTo (Lower) 8212ceb8 \Driver\DKbFltr
Device queue is not busy.
```

设备`KeyboardClass0`属于驱动`\Driver\Kbdclass`

```
kd> dt \_DEVICE\_OBJECT 8212a030
ntdll!\_DEVICE\_OBJECT
   +0x000 Type             : 0n3
   +0x002 Size             : 0x198
   +0x004 ReferenceCount   : 0n0
   +0x008 DriverObject     : 0x81ebf858 \_DRIVER\_OBJECT
   +0x00c NextDevice       : (null) --
   +0x010 AttachedDevice   : (null) --上面没有其他设备对象
   +0x014 CurrentIrp       : (null)
   +0x018 Timer            : (null)
   +0x01c Flags            : 0x2044
   +0x020 Characteristics  : 0
   +0x024 Vpb              : (null)
   +0x028 DeviceExtension  : 0x8212a0e8 Void
   +0x02c DeviceType       : 0xb
   +0x030 StackSize        : 7 ''
   +0x034 Queue            : \_\_unnamed
   +0x05c AlignmentRequirement : 0
   +0x060 DeviceQueue      : \_KDEVICE\_QUEUE
   +0x074 Dpc              : \_KDPC
   +0x094 ActiveThreadCount : 0
   +0x098 SecurityDescriptor : 0xe13b5f70 Void
   +0x09c DeviceLock       : \_KEVENT
   +0x0ac SectorSize       : 0
   +0x0ae Spare1           : 0
   +0x0b0 DeviceObjectExtension : 0x8212a1c8 \_DEVOBJ\_EXTENSION
   +0x0b4 Reserved         : (null)
```

通过设备扩展可以看到下面一层的设备

```
kd> dt \_DEVICE\_EXTENSION 8212a1c8
ACPI!\_DEVICE\_EXTENSION
   +0x000 Flags            : 0x8212a030`0000000d
   +0x000 UFlags           : \_\_unnamed
   +0x008 Signature        : 0x10
   +0x00c DebugFlags       : 0
   +0x010 DispatchTable    : (null)
   +0x014 WorkContext      : WORK\_QUEUE\_CONTEXT
   +0x014 Fdo              : \_FDO\_DEVICE\_EXTENSION
   +0x014 Filter           : \_FILTER\_DEVICE\_EXTENSION
   +0x014 Pdo              : \_PDO\_DEVICE\_EXTENSION
   +0x058 WorkQueue        : EXTENSION\_WORKER
   +0x058 Button           : BUTTON\_EXTENSION
   +0x058 Thermal          : THERMAL\_EXTENSION
   +0x058 LinkNode         : LINK\_NODE\_EXTENSION
   +0x058 Dock             : DOCK\_EXTENSION
   +0x058 Processor        : \_PROCESSOR\_DEVICE\_EXTENSION
   +0x088 DeviceState      : 0 ( Stopped )
   +0x08c PreviousState    : 0 ( Stopped )
   +0x090 PowerInfo        : \_ACPI\_POWER\_INFO
   +0x10c DeviceID         : (null)
   +0x10c Address          : 0
   +0x110 InstanceID       : (null)
   +0x114 ResourceList     : (null)
   +0x118 PnpResourceList  : (null)
   +0x11c OutstandingIrpCount : 0n0
   +0x120 ReferenceCount   : 0n0
   +0x124 HibernatePathCount : 0n0
   +0x128 RemoveEvent      : (null)
   +0x12c AcpiObject       : (null)
   +0x130 DeviceObject     : (null)
   +0x134 TargetDeviceObject : (null)
   +0x138 PhysicalDeviceObject : 0x00010008 \_DEVICE\_OBJECT
   +0x13c ParentExtension  : 0x7466744e \_DEVICE\_EXTENSION
   +0x140 ChildDeviceList  : \_LIST\_ENTRY [ 0xa030001 - 0x20616349 ]
   +0x148 SiblingDeviceList : \_LIST\_ENTRY [ 0x40000 - 0x0 ]
   +0x150 EjectDeviceHead  : \_LIST\_ENTRY [ 0x8212a318 - 0x8212a318 ]
   +0x158 EjectDeviceList  : \_LIST\_ENTRY [ 0xa080003 - 0x7266744e ]
```

嘿嘿，刚才找了下`strct`这个命令怎么不能用，把`F:\WINDDK\7600.16385.1\Debuggers\w2kfre`下的`kdex2x86.dll`直接拷贝到`system32`目录下，然后使用命令`!kdex2x86.strct`就可以查看啦，似乎比`dt`命令简洁多了

看效果：

```
kd> !kdex2x86.strct DEVOBJ\_EXTENSION 8212a1c8
struct   \_DEVOBJ\_EXTENSION (sizeof=36)
+00 int16    Type =                              000d
+02 uint16   Size =                              0000
+04 struct   \_DEVICE\_OBJECT \*DeviceObject =      8212A030
+08 uint32   PowerFlags =                        00000010
+0c  \*Dope =                                     00000000
+10 uint32   ExtensionFlags =                    00000000
+14 void     \*DeviceNode =                       00000000
+18 struct   \_DEVICE\_OBJECT \*AttachedTo =        8212CEB8--直接找到下一层设备了
+1c struct   \_LIST\_ENTRY FileObjectList
+1c    struct   \_LIST\_ENTRY \*Flink =             00000000
+20    struct   \_LIST\_ENTRY \*Blink =             00000000
```

#### 第二个

依旧如前

```
kd> !devobj 8212ceb8
Device object (8212ceb8) is for:
  \Driver\DKbFltr DriverObject 82339398
Current Irp 00000000 RefCount 0 Type 0000000b Flags 00002004
DevExt 8212cf70 DevObjExt 8212cfd0
ExtensionFlags (0000000000)
AttachedDevice (Upper) 8212a030 \Driver\Kbdclass
AttachedTo (Lower) 822d7310 \Driver\i8042prt
Device queue is not busy.
```

```
kd> !strct device\_object 8212ceb8
struct   \_DEVICE\_OBJECT (sizeof=184)
+00 int16    Type =                              0003
+02 uint16   Size =                              0114
+04 int32    ReferenceCount =                    00000000
+08 struct   \_DRIVER\_OBJECT \*DriverObject =      82339398
+0c struct   \_DEVICE\_OBJECT \*NextDevice =        81EBFCD8
+10 struct   \_DEVICE\_OBJECT \*AttachedDevice =    8212A030
+14 struct   \_IRP \*CurrentIrp =                  00000000
+18 struct   \_IO\_TIMER \*Timer =                  00000000
+1c uint32   Flags =                             00002004
+20 uint32   Characteristics =                   00000000
+24 struct   \_VPB \*Vpb =                         00000000
+28 void     \*DeviceExtension =                  8212CF70
+2c uint32   DeviceType =                        0000000b
+30 char     StackSize =                         06                       .
+34 union    \_\_unnamed62 Queue
+34    struct   \_LIST\_ENTRY ListEntry
+34       struct   \_LIST\_ENTRY \*Flink =          00000000
+38       struct   \_LIST\_ENTRY \*Blink =          00000000
+34    struct   \_WAIT\_CONTEXT\_BLOCK Wcb
+34       struct   \_KDEVICE\_QUEUE\_ENTRY WaitQueueEntry
+34          struct   \_LIST\_ENTRY DeviceListEntry
+34             struct   \_LIST\_ENTRY \*Flink =    00000000
+38             struct   \_LIST\_ENTRY \*Blink =    00000000
+3c          uint32   SortKey =                  00000000
+40          byte     Inserted =                 00                       .
+44       function \*DeviceRoutine =              00000000
+48       void     \*DeviceContext =              00000000
+4c       uint32   NumberOfMapRegisters =        00000000
+50       void     \*DeviceObject =               00000000
```
