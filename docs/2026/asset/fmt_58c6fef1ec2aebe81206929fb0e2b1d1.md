
'+-------+---------------+-------------------------------------------+
|  0x02 | Designer      | A Visual Basic Designer for an Add.in     |
|  0x10 | Class Module  | A Visual Basic Class                      |
|  0x20 | User Control  | A Visual Basic ActiveX User Control (OCX) |
|  0x80 | User Document | A Visual Basic User Document              |
'+-------+---------------+-------------------------------------------+

## 7）`ObjectInfo`

这些东西十分重要，是提取VB工程元素的重要标志，从这里我们可以得到这个工程里有多少FORM,多少MODULE,以及他们的重要索引数据，可以说重要性和SECTION HEAD在EXE程序中重要性相同。

入口的地址是由`ProjectInfo_t`结构提供的(semi提供关于这里提供的资料少之又少，所以需要自己摸索）

从`ProjectInfo_t`开始指向的是一个`ObjectTable_t`,由`ObjectTable_t`提供第一个`OBJECT_t`结构的地址

```
typedef struct //这个是OBJECT 的总表，可以索引以后的每个OBJECT
{
    DWORD lNull1 As Long;                                                                                                                                                                //没有用的填充东西
    DWORD aExecProj;                                                                                                                                                                                //VA指向一块内存结构(研究下来既不没见着这个东西由什么用处
    DWORD aProjectInfo2;                                                                                                                                                                //VA指向Project Info 2
    DWORD Const1;                                                                                                                                                                                                //没有用的填充东西
    DWORD Null2;                                                                        //没有用的填充东西
    DWORD lpProjectObject As Long                                ' 0x14
    DWORD Flag1;                                                                                                                                                                                                //标志1
    DWORD Flag2;                                                                                                                                                                                                //标志2
    DWORD Flag3;                                                                                                                                                                                                //标志3
    DWORD Flag4;                                                                                                                                                                                                //标志4
    WORD fCompileType;                                                                                                                                                                        //Internal flag used during compilation
    WORD ObjectCount1;                                                                                                                                                                        //OBEJCT数量1????
    WORD iCompiledObjects;                                                                                                                                                        //编译后OBJECT数量
    WORD iObjectsInUse As Integer;                                                                                                                        //Updated in the IDE to correspond the total number ' but will go up or down when initializing/unloading modules.
    DWORD aObject;                                                                                                                                                                                        //VA指向第一个OBJECT_t结构，很重要
    DWORD Null3;                                                                                                                                                                                                //没有用的填充东西
    DWORD Null4;                                                                                                                                                                                                //没有用的填充东西
    DWORD Null5;                                                                                                                                                                                                //没有用的填充东西
    DWORD aProjectName;                                                                                                                                                                        //执行工程名字的字符串
    DWORD LangID1;                                                                                                                                                                                        //language ID1
    DWORD LangID2;                                                                                                                                                                                        //language ID2
    DWORD Null6;                                                                                                                                                                                                //没有用的填充东西
    DWORD Const3;                                                                                                                                                                                                //没有用的填充东西
    ' 0x54
}ObjectTable_t;
```

```
type struct//这个就是每个OBJECT的结构，
{
    DWORD aObjectInfo;                                        //VA 指向一个ObjectInfo_t类型,来显示这个OBJECT的数据
    DWORD Const1;                                                                //没有用的填充东西
    DWORD aPublicBytes;                                        //VA 指向公用变量表大小
    DWORD aStaticBytes;                                        //VA 指向静态变量表地址
    DWORD aModulePublic;                                //VA 指向公用变量表
    DWORD aModuleStatic;                                //VA 指向静态变量表
    DWORD aObjectName;                                        //VA 字符串,这个OBJECT的名字
    DWORD ProcCount;                                                // events, funcs, subs(时间\函数\过程)数目
    DWORD aProcNamesArray;                        //VA 一般都是0
    DWORD oStaticVar;                                                //OFFSET  从aModuleStatic指向的静态变量表偏移
    DWORD ObjectType;                                                //比较重要显示了这个OBJECT的实行,具体见下表
    DWORD Null3;                                              //没有用的填充东西
    //sizeof() = 0x30
}Object_t;
```

`Object_t.ObjectTyper` 属性...//重要的属性表部分

```
#########################################################
form:              0000 0001 1000 0000 1000 0011 --> 18083
                   0000 0001 1000 0000 1010 0011 --> 180A3
                   0000 0001 1000 0000 1100 0011 --> 180C3
module:            0000 0001 1000 0000 0000 0001 --> 18001
                   0000 0001 1000 0000 0010 0001 --> 18021
class:             0001 0001 1000 0000 0000 0011 --> 118003
                   0001 0011 1000 0000 0000 0011 --> 138003
                   0000 0001 1000 0000 0010 0011 --> 18023
                   0000 0001 1000 1000 0000 0011 --> 18803
                   0001 0001 1000 1000 0000 0011 --> 118803
usercontrol:       0001 1101 1010 0000 0000 0011 --> 1DA003
                   0001 1101 1010 0000 0010 0011 --> 1DA023
                   0001 1101 1010 1000 0000 0011 --> 1DA803
propertypage:      0001 0101 1000 0000 0000 0011 --> 158003
                      | ||     |  |    | |    |
[moog]                | ||     |  |    | |    |
HasPublicInterface ---+ ||     |  |    | |    |  （有公用的接口）
HasPublicEvents --------+|     |  |    | |    |  （有公用的事件）
IsCreatable/Visible? ----+     |  |    | |    |        （是否可以创建，可见）
Same as "HasPublicEvents" -----+  |    | |    |
[aLfa]                         |  |    | |    |
```
