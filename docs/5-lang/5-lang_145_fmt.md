
# Delphi 逆向分析---quick batch 逆向

> 原文：[`www.pediy.com/kssd/pediy10/80205.html`](https://www.pediy.com/kssd/pediy10/80205.html)

## Delphi 逆向分析---quick batch 逆向

元旦好无聊呀。独自一个人在成都。。。

Delphi 逆向分析中较困难的就是一些 VCL、RTL 库函数被 IDA 识别成 unknowlib，还有许多虚函数的调用，例如：`call [edx+0xC]`，此外 Pascal 的参数传递方式也比较让人不习惯。虽然有 DeDe 这样强大的逆向工具，但个人在处理以上问题时仍需要较强的手动分析逆向技巧。

### Delphi 的参数传递

- `Eax`：第一个参数
- `Edx`：第二个参数
- `Ecx`：第三个参数
- `[Esp+8]` 第四个，`[esp+c]` 第五个……（如果有的话）

### Meta 结构

每个类都有一个 Meta 结构，在这里我姑且可以把它认为是类的模板。以下是一个 Meta 结构。

（delphi 源码）

```delphi
type
  MyBaseClass=class
    Number:integer;
  public
    constructor Create(num:integer);virtual;
    class procedure StaticTest();virtual;
    procedure Test();virtual;
  end;

  MyChildClass=class(MyBaseClass)
    newNumber:integer;
  public
    constructor Create(num:integer);override;
    procedure Test();override;
    class procedure StaticTest();override;///类方法（静态方法）
  end;

  MySingleClass=class
    Number:integer;
  end;
```

（编译后的 Meta 结构）

从 IDA 中 copy 汇编出来，没对齐好，太丑了。这个我就简单写下结构，详细请看 `project9.idb`

```
MyChildClassMeta dd offset childVtr ; 指向虚函数表，如果没有虚函数则指向类名称
dd 7 dup(0) ; （7 个 dd 0，不知道有什么用，好像除了继承自 TComponent 和 Exception 的类，其他都是 7 个 dd 0）有谁发现了请告诉我 ^_^
dd offset aMychildclass ; 指向类名称字符串
dd 0ch ; 对象大小 Sizeof(MyChild)=虚函数表指针大小+2 个 int=0xC
dd offset MyBaseClassMeta ; 指向父类的 Meta 结构

######################

dd offset @System@TObject@SafeCallException$qqrp14System@TObjectpv ; System::TObject::SafeCallException(System::TObject*, void*)
dd offset nullsub_15
dd offset nullsub_30
dd offset @System@TObject@Dispatch$qqrpv ; System::TObject::Dispatch(void*)
dd offset nullsub_29
dd offset @Comctrls@TTreeNodes@GetFirstNode$qqrv ; Comctrls::TTreeNodes::GetFirstNode(void)
dd offset sub_4036A4
dd offset @System@TObject@$bdtr$qqrv ; System::TObject::~TObject(void)

######################

; （以上结构比较固定，好像在所有的类里面都是这样）

childVtr dd offset sub_44CCB8 ; （虚函数表）
    dd offset sub_.....
    dd offset sub_.....
db 12, 'MyChildClass'
```

### Delphi 中的构造函数

（source）

```delphi
constructor MyBaseClass.Create(num:integer);
begin
  Number:=num;
end;

constructor MyChildClass.Create(num:integer);
begin
  inherited Create(num);
  newNumber:=num+1;
end;
```

汇编：

```asm
--------------------------------------------------------
mov     ecx, 3
mov     dl, 1    ; alloc 是否分配空间
mov     eax, MyChildClassMeta
call    ChildCreate   ; ; 调用
--------------------------------------------------------

; int __fastcall ChildCreate(Meta ChildMeta, bool alloc, int num)
ChildCreate     proc near

CODE:0044CD38           push    ebx
CODE:0044CD39           push    esi
CODE:0044CD3A           push    edi
CODE:0044CD3B           test    dl, dl
CODE:0044CD3D           jz      short loc_44CD47
CODE:0044CD3F           add     esp, 0FFFFFFF0h
CODE:0044CD42           call    @System@@ClassCreate$qqrp17System@TMetaClasso ; System::__linkproc__ ClassCreate(System::TMetaClass*, bool) ; ; 根据 Meta 分配对象空间，并初始化默认值
CODE:0044CD47
CODE:0044CD47 loc_44CD47:
CODE:0044CD47           mov     esi, ecx
CODE:0044CD49           mov     ebx, edx
CODE:0044CD4B           mov     edi, eax
CODE:0044CD4D           mov     ecx, esi
CODE:0044CD4F           xor     edx, edx  ; dl=0 基类构造函数,不再分配空间
CODE:0044CD51           mov     eax, edi
CODE:0044CD53           call    BaseCreate
CODE:0044CD58           inc     esi
CODE:0044CD59           mov     [edi+8], esi
CODE:0044CD5C           mov     eax, edi
CODE:0044CD5E           test    bl, bl
CODE:0044CD60           jz      short loc_44CD71
CODE:0044CD62           call    @System@@AfterConstruction$qqrp14System@TObject ; System::__linkproc__ AfterConstruction(System::TObject*)
CODE:0044CD67           pop     large dword ptr fs:0
CODE:0044CD6E           add     esp, 0Ch
CODE:0044CD71
CODE:0044CD71 loc_44CD71:
CODE:0044CD71           mov     eax, edi
CODE:0044CD73           pop     edi
CODE:0044CD74           pop     esi
CODE:0044CD75           pop     ebx
CODE:0044CD76           retn
CODE:0044CD76 ChildCreate endp
```

构造函数经过编译后会自动添加两个参数 `Meta ChildMeta`, `bool alloc`。

`Meta` 类模板，`alloc` 是否分配空间。从汇编代码中可以看到，传入 `ChildMeta`，调用 `System@@ClassCreate` 为 `MyChild` 对象分配了 `0xC` 大小的空间，并初始化默认值（这里的初始化指变量设置为 0 或 nil，并非值构造函数内的初始化），然后再调用基类构造函数，可以看到，在调用基类构造函数时，`dl=0`;即此时不再分配空间。然后再继续完成 Child 类的初始化---`newNum:=num+1`.

有了以上知识我们就可以轻而易举地进行 delphi 程序的逆向了。

## Quick Batch 2.7 逆向

Quick Batch 是一个将 `bat` 转换成 `exe` 的工具（加密压缩后保存在 `exe` 文件中，2.7 版本的保存在 `append data` 中，3.0 以上版本保存在资源中），该工具还可以捆绑其他文件。因此经常被恶意程序所利用。

### 事先准备

1. delphi 的 VCL、RTL 源码（在 delphi 安装目录下 `source` 文件夹）；
2. Delphi 帮助文档。

### IDA 载入 Quick Batch 测试程序 test.exe

1. 找到 meta 定义部分，将所有的 meta 数据重命名；
2. 转到主程序部分

#### 主程序部分

由于 delphi 从 `start` 开始就直接是 `main` 函数（不像 C，有一段的先导代码然后才跳到 `main`），因此可以将 `start` 直接转换成函数，这样 IDA 就能直接标出局部变量。

1. 识别出构造函数，一般在跟在 Meta 后面的即是该类的构造函数，例如：

```asm
CODE:00418EC7           xor     ecx, ecx
CODE:00418EC9           mov     dl, 1
CODE:00418ECB           mov     eax, THKStreamsMeta
CODE:00418ED0           call    THKstreams@Create
```

2. 将虚函数重命名；例如：

```asm
CODE:00418FCC           mov     edx, ds:memStreamObj3
CODE:00418FD2           mov     eax, ds:strListObj1
CODE:00418FD7           mov     ecx, [eax]
CODE:00418FD9           call    dword ptr [ecx+6Ch]
```

此处 Ctrl+R，将基址改为 `TStringListMeta` 的虚函数表开始，就变成这样了：

`call (TstringList@LoadFromStream - _cls_Classes_TStringList)[ecx] ;`

3. 查 delphi 帮助，认清 RTL、VCL 类的继承关系，识别出各个参数含义

```asm
CODE:00418E94           push    ebp
CODE:00418E95           mov     ebp, esp
CODE:00418E97           mov     ecx, 9
CODE:00418E9             。。。。。。略过一些
CODE:00418EC7           xor     ecx, ecx
CODE:00418EC9           mov     dl, 1
CODE:00418ECB           mov     eax, THKStreamsMeta
CODE:00418ED0           call    THKstreams@Create ; ; ; 创建 THKS 对象
CODE:00418ED5           mov     ds:THKstreamsObj, eax
CODE:00418EDA           mov     eax, ds:THKstreamsObj
CODE:00418EDF           mov     byte ptr [eax+30h], 1 ; compressed=true
CODE:00418EE3           mov     eax, ds:THKstreamsObj
CODE:00418EE8           mov     byte ptr [eax+31h], 1 ; EnCrypted=true
CODE:00418EEC           mov     eax, ds:THKstreamsObj
CODE:00418EF1           add     eax, 32h        ; key
CODE:00418EF4           mov     edx, offset aQuickBatchFile ; "Quick Batch File Compiler"       ; ; ; quick batch as the keyword
        。。。。。。。再略过一些
CODE:00418F72           mov     eax, [ebp+var_18]
CODE:00418F75           mov     edx, ds:MemStreamObj1
CODE:00418F7B           call    ReadEncrpyData
CODE:00418F80           mov     ds:byte_41E960, al
CODE:00418F85           cmp     ds:byte_41E960, 0
CODE:00418F8C           jz      ReadFailedHandler
CODE:00418F92           mov     edx, ds:MemStreamObj1 ; ms
CODE:00418F98           mov     eax, ds:THKstreamsObj ; HKStream
CODE:00418F9D           call    HKStream@LoadFromStream ; ; ; ; Load 进来以后实际上已经解密了。
CODE:00418FA2           mov     ecx, ds:memStreamObj2
CODE:00418FA8           mov     edx, offset aBat ; "BAT"
CODE:00418FAD           mov     eax, ds:THKstreamsObj
CODE:00418FB2           call    HKS@GetStream ; ; 得到 BAT 段数据，放入 ms2
CODE:00419102           mov     ecx, ds:dword_41E968
CODE:00419108           mov     edx, ds:TmpPath
        。。。。略
CODE:00418FDC           mov     edx, ds:memStreamObj2
```



### `CODE:00418FE2`
```
mov eax, ds:StrListObj2
```
### `CODE:00418FE7`
```
mov ecx, [eax]
```
### `CODE:00418FE9`
```
call (TstringList@LoadFromStream - _cls_Classes_TStringList)[ecx]
```
;;; 重新将 `ms` 载入到 `StrList2` 中

### `CODE:00419113`
```
mov edx, [ebp+var_34]
```
### `CODE:00419116`
```
mov eax, ds:StrListObj2
```
### `CODE:0041911B`
```
mov ecx, [eax]
```
### `CODE:0041911D`
```
call (SaveToFile - _cls_Classes_TStringList)[ecx] ; 保存 strList2 到 bat 文件
```

## `THKStreams` 结构

```pascal
THKStreams = class(TComponent)
  private
    FCompressed, FEncrypted: Boolean;
    FKey: String;
    FOnAskForKey: TOnAskForKey;
    FOnCorrupt: TNotifyEvent;
    procedure WriteStr(S: String; Stream: TStream);
    function ReadStr(Stream: TStream): string;
    Procedure LoadFromStreamNor(ms: TStream);
    Procedure SaveToStreamNor(ms: TStream);
    function CheckGood(ms: TStream): boolean;
    procedure FoundCorrupt;
  public
    StreamList: TStringList;
    Constructor Create(AOWner: TComponent); override;
    Destructor Destroy; override;
    Procedure LoadFromFile(const Filename: string);
    Procedure SaveToFile(const Filename: string);
    Procedure AddStream(Const ID: string; Source: TStream);
    Procedure RemoveStream(Const ID: String);
    Procedure LoadFromStream(ms: TStream);
    Procedure SaveToStream(ms: TStream);
    Procedure GetStream(Const ID: string; Dest: TStream);
    Procedure ClearStreams;
  published
    property Compressed: boolean read FCompressed write FCompressed;
    property Encrypted: boolean read FEncrypted write FEncrypted;
    Property Key: string read FKey write FKey;
    Property OnAskForKey: TOnAskForKey read FOnAskForKey write FOnAskForKey;
    property OnCorrupt: TNotifyEvent read FOnCorrupt write FOnCorrupt;
  end;
```

`THKStreams` 数据保存在 `streamList` 结构中，像这样 `“BAT”bat data; “Files”FileData …..`

完了之后，汇编就非常容易阅读了。可以轻而易举地写出 `delphi` 解压源码。(解密代码请见 `\\hks\\UnpackQuickBatch.drp`)

`THKStreams` 这个类来自第三方，可以从网络上下载到。`Quick Batch` 的加密解密都是用了它。

### 上传的附件

| ![image](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `delphi.rar` |
| :--- | :--- |
| ![image](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `project9.part1.rar` [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=80205`](http://bbs.pediy.com/showthread.php?t=80205%20)] |
| ![image](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `project9.part2.rar` |
| ![image](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) | `test idb.zip` |
