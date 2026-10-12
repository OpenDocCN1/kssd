
```
    DWORD LangID1;                                   //language ID1
    DWORD LangID2;                                   //language ID2
    DWORD Null6;                                     //没有用的填充东西
    DWORD Const3;                                     //没有用的填充东西
    ' 0x54
}ObjectTable_t;
```

```
type struct//这个就是每个 OBJECT 的结构，
{
    DWORD aObjectInfo;          //VA 指向一个 ObjectInfo_t 类型,来显示这个 OBJECT 的数据
    DWORD Const1;              //没有用的填充东西
    DWORD aPublicBytes;        //VA 指向公用变量表大小
    DWORD aStaticBytes;        //VA 指向静态变量表地址
    DWORD aModulePublic;       //VA 指向公用变量表
    DWORD aModuleStatic;       //VA 指向静态变量表
    DWORD aObjectName;         //VA 字符串,这个 OBJECT 的名字
    DWORD ProcCount;           // events, funcs, subs(时间\函数\过程)数目
    DWORD aProcNamesArray;     //VA 一般都是 0
    DWORD oStaticVar;          //OFFSET  从 aModuleStatic 指向的静态变量表偏移
    DWORD ObjectType;          //比较重要显示了这个 OBJECT 的实行,具体见下表
    DWORD Null3;               //没有用的填充东西
    //sizeof() = 0x30
}Object_t;
```

```
'Object_t.ObjectTyper 属性...//重要的属性表部分
'#########################################################
'form:              0000 0001 1000 0000 1000 0011  --> 18083
'                  0000 0001 1000 0000 1010 0011  --> 180A3
'                  0000 0001 1000 0000 1100 0011  --> 180C3
'module:            0000 0001 1000 0000 0000 0001  --> 18001
'                  0000 0001 1000 0000 0010 0001  --> 18021
'class:             0001 0001 1000 0000 0000 0011  --> 118003
'                  0001 0011 1000 0000 0000 0011  --> 138003
'                  0000 0001 1000 0000 0010 0011  --> 18023
'                  0000 0001 1000 1000 0000 0011  --> 18803
'                  0001 0001 1000 1000 0000 0011  --> 118803
'usercontrol:       0001 1101 1010 0000 0000 0011  --> 1DA003
'                  0001 1101 1010 0000 0010 0011  --> 1DA023
'                  0001 1101 1010 1000 0000 0011  --> 1DA803
'propertypage:      0001 0101 1000 0000 0000 0011  --> 158003
'                   | ||     |  |    | |    |
'\[moog\]            | ||     |  |    | |    |
'HasPublicInterface ---+ ||     |  |    | |    |  （有公用的接口）
'HasPublicEvents  --------|     |  |    | |    |  （有公用的事件）
'IsCreatable/Visible? ----     |  |    | |    |  （是否可以创建，可见）
'Same as "HasPublicEvents" -----  |    | |    |
'\[aLfa\]                           |  |    | |    |
'usercontrol (1)  -------------------+  |    | |    |  （用户控制）
'ocx/dll (1)  ---------------------------+    | |    |  （OCX/DLL）
'form (1)  -----------------------------------+  |    |  （是不是 FORM 是就是 1）
'vb5 (1)  ----------------------------------------+    |  （是不是 VB5 是就是 1）
'HasOptInfo (1)  -------------------------------------+  （有没有额外的信息信息由就是 1,决定是不是指向 OptionalObjectInfo_t 类似与 PEHEAD 里的 Optional 信息一样）
'                                                    |
'module(0)  -----------------------------------------+   （如果是 Module 模块就这里是 0）
```

```
typedef struct//这个是显示这个 OBJECT 信息的结构,每一个 OBJECT 都有一个
{
    WORD Flag1;
    WORD ObjectIndex;
    DWORD aObjectTable;
    DWORD Null1;
    DWORD aSmallRecord;        // when it is a module this value is -1 [better name?]
    DWORD Const1;
    DWORD Null2;
    DWORD aObject;
    DWORD RunTimeLoaded;      //[can someone verify this?]
    DWORD NumberOfProcs;
    DWORD aProcTable;
    WORD iConstantsCount;     // Number of Constants
    WORD iMaxConstants;       // Maximum Constants to allocate.
    DWORD Flag5;
    WORD Flag6;
    WORD Flag7;
    WORD aConstantPool;
    // sizeof() =  0x38
    'the rest is optional items[OptionalObjectInfo]
}ObjectInfo_t;
```

```
Private Type tOptionalObjectInfo ' if ((tObject.ObjectType AND &H80)=&H80)
    fDesigner As Long                             ' 0x00 (0d) If this value is 2 then this object is a designer
    aObjectCLSID As Long                          ' 0x04
    Null1 As Long                                 ' 0x08
    aGuidObjectGUI As Long                        ' 0x0C
    lObjectDefaultIIDCount As Long                ' 0x10  01 00 00 00
    aObjectEventsIIDTable As Long                 ' 0x14
    lObjectEventsIIDCount As Long                 ' 0x18
    aObjectDefaultIIDTable As Long                ' 0x1C
    ControlCount As Long                          ' 0x20
    aControlArray As Long                         ' 0x24
    iEventCount As Integer                        ' 0x28 (40d) Number of Events
    iPCodeCount As Integer                        ' 0x2C
    oInitializeEvent As Integer                   ' 0x2C (44d) Offset to Initialize Event from aMethodLinkTable
    oTerminateEvent As Integer                    ' 0x2E (46d) Offset to Terminate Event from aMethodLinkTable
    aEventLinkArray As Long                       ' 0x30  Pointer to pointers of MethodLink
    aBasicClassObject As Long                     ' 0x34 Pointer to an in-memory
    Null3 As Long                                 ' 0x38
    Flag2 As Long                                 ' 0x3C usually null
    ' 0x40 <-- Structure size
End Type
```

```
Type tProjectInfo2
    lNull1                  As Long                ' 0x00 (00d)
    aObjectTable            As Long                ' 0x04 (04d) Pointer to Object Table
    lConst1                 As Long                ' 0x08 (08d)
    lNull2                  As Long                ' 0x0C (12d)
    aObjectDescriptorTable  As Long                ' 0x10 (16d) Pointer to a table of ObjectDescriptors
    lNull3                  As Long                ' 0x14 (20d)
    aNTSPrjDescription      As Long                ' 0x18 (24d) Pointer to Project Description
    aNTSPrjHelpFile         As Long                ' 0x1C (28d) Pointer to Project Help File
    lConst2                 As Long                ' 0x20 (32d)
    lHelpContextID          As Long                ' 0x24 (36d) Project Help Context ID
    ' 0x28 (40d) <- Structure size
End Type
```

```
Type ObjectDescriptor
    lNull1      As Long                             '0x00 (00d)
    aObjectInfo As Long                             '0x04 (04d) Pointer to Object Info
    lConst1     As Long                             '0x08 (08d)
    lNull2      As Long                             '0x0C (12d)
    lFlag1      As Long                             '0x10 (16d)
    lNull3      As Long                             '0x14 (20d)
    aUnknown1   As Long                             '0x18 (24d)
    lNull4      As Long                             '0x1C (28d)
    aUnknown2   As Long                             '0x20 (32d)
    aUnknown3   As Long                             '0x24 (36d)
    aUnknown4   As Long                             '0x28 (40d)
    lNull5      As Long                             '0x2C (44d)
    lNull6      As Long                             '0x30 (48d)
    lNull7      As Long                             '0x34 (52d)
    lFlag2      As Long                             '0x38 (56d)
    fObjectType As Long                             '0x3C (60d) Flags for this Object
    '0x40 (64d) <- Structure Size
End Type
```

×××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××

未完待续！！！！！（还有很大大大大一部分类！！！）

××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××
