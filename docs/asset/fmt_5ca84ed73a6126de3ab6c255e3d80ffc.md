
## 结构定义

```
typedef struct
{
  WORD Flag1;//Integer                                     ' 0x00
  WORD EventCount;//Integer                                 ' 0x02
  DWORD Flag2;//Long                                       ' 0x04
  DWORD aGUID;//Long                                       ' 0x08
  WORD index;//Integer                                     ' 0x0C
  WORD Const1;//Integer                                    ' 0x0E
  DWORD Null1;//Long                                       ' 0x10
  DWORD Null2;//Long                                       ' 0x14
  DWORD aEventTable;//Long                                  ' 0x18
  BYTE Flag3;//Byte                                        ' 0x1C
  BYTE Const2;//Byte                                       ' 0x1D
  WORD Const3;//Integer                                    ' 0x1E
  DWORD aName;//Long                                       ' 0x20
  WORD Index2;//Integer                                    ' 0x24
  WORD Const1Copy;//Integer                                 ' 0x26
  // 0x28  <-- Structure Size
}Control_t;
```

## 工程分析与重建

看到这里我们已经可以通过这个表粗略地勾画出那个工程是怎样的，通过 `PROJECT_INFO` 和其他的一些东西，我们已经能够重建 `VBP` 文件，而且我们已经知道这个工程到底有多少源文件（`FRM`, `MOD` 不包括 `FRX`）。

接下来，我们来读取 `EXE` 文件中保存的数据来重建每个 `MOD`, `FRM`, `CLS`, `VBP`。

### 重建 `VBP`

我们本文不是主要介绍 VB 工程组文件的内部结构，所以只是一笔带过，`VBP` 文件内部格式和普通的配置文件类似都是(关键词,对应值)对的形式。

主要有这么几个字段我们要注意（输出的东西和 `SEMI` 一样，关于 `VERSION` 会比 `SEMI` 详细，将在之后介绍）：

*   `Type=Exe`（这个就是这个工程是什么我们当然是 `EXE`）
*   `Startup="frmMain"`（这个就是工程的启动项目，关于它我们后面要详细介绍）
*   `Description="Visual Basic Decompiler"`（这个是描述，`(char*)COMRegData+COMRegData->oNTSProjectDescription`）
*   `HelpFile=""`（帮助文件`(char*)VBHEADER+VBHEADER->oHelpFile` 就是其在文件中的偏移）
*   `Name="VBDecompiler"`（工程名字，获得方式和 `HELPFILE` 类似，只要看 `VBHEADER` 里面就有存储其偏移地址）
*   `Title="VBDecompiler"`（工程标题，获得方式和 `HELPFILE` 类似，只要看 `VBHEADER` 里面就有存储其偏移地址）
*   `ExeName32="SemiVBDecompiler"`（工程 `EXE32` 的名字，获得方式和 `HELPFILE` 类似，只要看 `VBHEADER` 里面就有存储其偏移地址）

**注意**：每找到一个 `OBJECT` 对应的文件，我们要将其添加到 `VBP` 工程里面，具体的方式是：

*   如果这个 `OBJECT` 是 `FRM` 文件，只要在 `VBP` 文件里面添加一条 `Form=窗体文件的名字`（文件名）（没有空格要求`=`以后紧跟）
*   如果这个 `OBJECT` 是 `MOD` 文件，只要在 `VBP` 文件里面添加一条 `Module=内部标识名;文件名`（之间需要一个分号来格开）
*   如果这个 `OBJECT` 是 `CLASS` 文件，只要在 `VBP` 文件里面添加一条 `Class=内部标识名;文件名`（之间需要一个分号来格开）

这样的话，在 `VBP` 文件中维护各个工程文件的工作就大致完成了。

其他重要的项还有例如 `OBJECT` 项（添加使用的控件）以及版本信息相关的（在后面的资源反编译一节会着重介绍）。

剩下的就是关于编译时的优化选项了，这些不是很重要，所以就没有加入。

其实 `VBP` 文件里包含了很多工程的细节，但是我们主要只要抓住重要的字段。

### 重建 `MOD`

`MOD` 文件比较简单，由于文件名可能和模块名不同，编译的时候舍弃了实际文件名，而用模块名来作为标识，所以我们生成的 `MOD` 文件的名字选用模块名，可能与原始的源文件组不同。

获得一个 `OBJECT` 之后，我们看 `Object_t.ObjectType` 通过查表我们能够确定它的性质。

我们确定这个是 `MOD` 文件之后，我们通过 `Object_t.aObjectName`（指向一个字符串）这个就是这个模块的名字，也是这个模块在编译后的文件中的标识。

我们用这个标识名来作为文件名，创建一个文件，然后我们通过 `Object_t.ProcCount` 知道这个 `MOD` 里存储着多少个 `FUNCTION` 和 `SUB`，并且由于 `MOD` 是全编译，我们得不到具体的 `SUB` 和 `FUNCTION` 的名字，这些名字在编译的时候被丢弃的所以我们只能知道到底有多少个 `SUB` 和 `FUNCTION`。

所以 `MOD` 的重建并不能得到什么东西，只能空创建一个文件然后最多写入：`'There is totally 100 methods in this module.But we can't show you them.:)`

还要记得一点就是我们要为这个 `BAS` 文件写上它的 `VB_ATTRIB`，具体的格式就是 `Attribute VB_Name = "模块的名字(内部标识名)"`。

这部分的具体代码重建（一般来说不能完全重建），我们只能有待更加强大的代码反编译来完成。

### 重建 `FRM`

`FRM` 和 `MOD` 不同的是我们可以得到 `FRM` 里面的所有控件的基本静态状态（属性），并且我们可以得到里面存储的 `SUB` 和 `FUNCTION` 的名字（不同于 `MOD`）。

同样如何知道这个 `OBJECT` 是一个 `FRM` 文件还是要查表。

知道它是一个 `FRM` 文件之后我们首先要了解一下 `FRM` 文件的结构：

`FRM` 文件是类似配置文件的格式存储的，主要有外部的 `VB_ATTRIB` 定义以及 `FORM` 成员的定义。

示例结构：

```
VERSION 5.00 //文件的编译版本
Begin VB.Form 内部标识名 //表示这个 FORM 的开始
Begin VB.VB 内部控件对象 内部标识名
End
End
Attribute VB_Name = "FORM 的内部标识名"
```

多重的嵌套就完成了这个 `FORM` 里面控件的从属关系（例如一个 `PICTUREBOX` 从属与 `FORM`，而一个 `TEXT` 从属于一个 `FRAME`）。

之后我们要得到这个 `FRM` 文件里面的控件信息和 `SUB` 以及 `FUNCTION` 的信息，来完成我们的 `FRM` 写入。

每个 `OBJECT` 里有一个 `TYPE` 表示这个 `OBJECT` 的属性，我们已经知道凡是是一个 `FRM` 都是有 `OptionalObjectInfo`。

这个结构告诉我们很多元素的索引，获取的方式十分简单，只要先确定有 `OptionalObjectInfo` 然后它的地址就是这个 `Object` 的`(char*)ObjectInfo+sizeof(ObjectInfo)`其实就是紧紧跟在 `ObjectInfo` 之后的。

结构的描述已经在上面列出了，这个结构中的信息十分复杂，我们要注意这么几个项目：

*   `ControlCount`：列出控件个数
*   `aControlArray`：指向控件表

控件表是一个紧紧挨着的一个指针数组，我们可以逐个读取获得信息。

伪代码：

```
for (i = 0 ; i<=this->ObjectTable->ObjectCount1 -1 ; ++i,++t)
{//循环获得所有的 OBJECT 的指针
  s = this->Get_OptionalObjectInfo(t);  //获取一个 OBJECT 的 OptionalObjectInfo
  if (s==NULL)  // it is a module and haven't any optional object info
    continue;
  cout<<"One frm found!"<<endl;//打印出：获得一个 FRM
  ct = this->Get_Control_t(s);//获取第一个 Control 的指针
  for (d = 0 ; d<=s->ControlCount-1 ; d++)//循环获得每一个 CONTROL 的索引
    cout<<"one control:"<<this->Get_Control_Name(ct++)<<endl;//获得了一个 Control 打印出它的名字
}

inline BYTE * VBEXE::Get_Control_Name(VBST::Control_t *x)
{
  return (this->buffer + this->sf.VA2Offset(x->aName));
}
```

这里其实是我的 `DECOMPILER` 的一些片段，仅仅是用作调试的还未成型所以就不敢贴出来丢脸了：）。

每个 `Control_t` 的 `aName` 是一个 `VA` 指向一个字符串就是表示这个控件的名字，至于控件的属性我们以后再说：）。

我们看看我的 `VBDECOMPILER` 输出的读取那个 `SEMI` 的 `DECOMPILER` 的东西：

```
Load file sucessfully!
Get contols in object
One frm found!
one control:mnuFile
one control:mnuHelpAbout
one control:txtFinal
one control:mnuFileOpen
one control:mnuTools
one control:Label1
one control:Form
one control:Label2
one control:mnuFileRecent1
one control:mnuFileSep1
one control:mnuFileRecent2
one control:mnuFileRecent3
one control:mnuFileRecent4
one control:mnuFileSep2
one control:lblObjectName
one control:mnuOptions
one control:mnuFileDebugProcess
one control:mnuToolsPCodeProcedure
one control:txtCode
one control:tvProject
one control:StatusBar1
one control:sstViewFile
one control:fxgEXEInfo
one control:picPreview
one control:mnuFileAntiDecompiler
one control:mnuFileExportMemoryMap
one control:mnuFileGenerate
one control:lstMembers
one control:lstTypeInfos
one control:mnuFileSaveExe
one control:txtBuffer
one control:txtFunctions
one control:txtEditArray
one control:lblArrayEdit
one control:buffCodeAv
one control:buffCodeAp
one control:mnuFileExit
one control:txtResult
one control:cmdCancel
one control:txtStatus
one control:mnuHelp
one control:FrameStatus
One frm found!
one control:imgFlame
one control:lblTitle
one control:TmrLight
one control:tmrIcon
one control:Form
One frm found!
one control:chkShowOffsets
one control:chkSkipCOM
one control:chkDumpControls
one control:cmdClose
one control:chkPCODE
one control:chkShowColors
one control:Form
One frm found!
one control:Class
One frm found!
one control:Class
One frm found!
one control:lblTitle
one control:cmdClose
one control:Form
one control:Label1
one control:lstProcedures
one control:txtView
```

到这里，我们以及能够基本将原来的工程组还原出来了，但是仍然有缺点：

1.  我们得不到（应该说实我水平不够）`VBP` 文件里的 `Reference`
2.  我们即将讨论关于 `VERSION` 信息的东西（要从一个 `EXE` 的 `RESOURCE` 中看出端倪）
3.  我们的 `CODE` 仍然还是一大问题
4.  我们的 `STARTUP` 开始段，（马上会研究出来的）
5.  我们还不知道哪个 `FRM` 有 `FRX` 文件哪个没有而且不能还原,而且 `FRM` 里面的控件还是不能获得属性
6.  `PCODE` 一点都没有讨论
7.  还有一些细节的信息没能弄出来

下一篇会说到 `VERSION INFO` 和控件属性的东西。

## VB6 程序消息入口

趁机也跟一个简单的。

**by** `nbw`

`vb` 程序加载后是个：

```
00401160 > $ 68 9C124000   push   0040129C
00401165   . E8 F0FFFFFF   call   <jmp.&MSVBVM60.#100>
```

没法调试。

从回收站里把笔记拉了出来，再不贴上来恐怕下次又要重新分析了。

这里说一下处理方法：

打开 `VB6` 运行库，搜索下面代码：

```
733BAE50       8B7424 08       mov   esi, [esp+8]
733BAE54       57              push  edi
733BAE55       8B40 10         mov   eax, [eax+10]
733BAE58       8B0E            mov   ecx, [esi]
733BAE5A       8B7C81 0C       mov   edi, [ecx+eax*4+C]   //这里 edi 指向消息入口
733BAE5E       . 57            push  edi                 ; Project1.00401470
```

最后的 `edi` 是消息入口地址。

比如刚加载程序的时候，断在这里，`edi` 就是程序的 `Load` 函数。

如果点一下程序上的按钮，也会断在这里，这时候 `edi` 就是这个按钮的 `click` 函数地址。

这些地址都是在一起的，比如：

```
00401470   . 816C24 04 30>sub   dword ptr [esp+4], 33          // 这个是 app.load
00401478   . E9 93040000   jmp   00401910
               sub   dword ptr [esp+4], xx   // 这个可能是 timer
               jmp   xxxxxxxx
```

需要注意的是 `sub` 命令后面的参数不能理解为 `win32` 里面的消息，比如这里的 `0x33` 代表程序加载，换个程序可能 `33` 代表 `click`。

谁有 `visual basic 5.0` 编译环境麻烦给我说一下，我没环境没法找 `vb5` 的。
