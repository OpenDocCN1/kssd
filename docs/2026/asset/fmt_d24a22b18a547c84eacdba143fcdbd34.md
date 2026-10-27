
# [原创][分享]DLL动态链接库的实现及调用实例（VC++，Delphi）

该压缩包包括：1.动态链接库(DLL)的编译实例，包括`_cdecl`和`stdcall`两种常见函数调用约定的实现，并且解决了名字修改问题，在实例代码中有详细的说明！2.针对不同的函数调用约定，以及动态调用DLL库和静态调用DLL库分别提供了调用实例的源代码和程序！3.压缩包中包括VC++调用DLL的实例源码和Delphi调用DLL的实例源码和说明(利用前面VC++生成的DLL)！4.其中VC++调用DLL实例中还简单说明了计算`string`的长度，比较，大小写转换等功能实现的方法！

## （一）dll目录说明

### 一、`dll_cdecl`目录

- 为了在VC++下使用`zjlj.lib`联接编译，静态调用用,`_cdecl`调用约定
- 例如`str`目录下的VC++编译文件使用！`zjlj.lib`联接编译（静态调用）
- 编译好的dll文件可以用VC++6自带的工具`Depend`查看

### 二、`dll_stdcall_def`目录

- 是为其他编译环境调用dll使用，`_stdcall`调用约定,可静态，动态调用（解决名字修改问题）
- 例如`delphi7`目录下的delphi编译文件使用！（动态调用）

### 三、`dll_cdecl_def`目录

- 是为其他编译环境调用dll使用，`_cdecl`调用约定,可静态，动态调用（解决名字修改问题）
- 例如`str`目录下的VC++编译文件使用！`zjlj.lib`联接编译（动态调用）

## （二）example目录说明

1. `str`目录：在VC++下使用`zjlj.lib`联接编译最后调用dll的实例（编译环境设置调入dll）
2. `strnew(cdecl)静态`目录：在VC++下使用`zjlj.lib`联接编译最后（静态）调用dll的实例（代码调入dll）
3. `strnew(cdecl)动态`目录：在VC++下使用显试（动态）`cdecl`调用dll的实例
4. `strnew(stdcall)静态`目录：在VC++下使用`zjlj.lib`联接编译最后（静态）调用dll的实例（代码调入dll）
5. `strnew(stdcall)动态`目录：在VC++下使用显试（动态）`stdcall`调用dll的实例
6. `delphi7`目录：delphi编译文件调用dll文件的实例

注意：由于压缩包里面包含了`delphi7`编译的实例，360杀毒软件可能会把它当成病毒进行警告

该压缩包包括：1.动态链接库(DLL)的编译实例，包括`_cdecl`和`stdcall`两种常见函数调用约定的实现，并且解决了名字修改问题，在实例代码中有详细的说明！2.针对不同的函数调用约定，以及动态调用DLL库和静态调用DLL库分别提供了调用实例的源代码和程序！3.压缩包中包括VC++调用DLL的实例源码和Delphi调用DLL的实例源码和说明(利用前面VC++生成的DLL)！4.其中VC++调用DLL实例中还简单说明了计算`string`的长度，比较，大小写转换等功能实现的方法！

## （一）dll目录说明

### 一、`dll_cdecl`目录

- 为了在VC++下使用`zjlj.lib`联接编译，静态调用用,`_cdecl`调用约定
- 例如`str`目录下的VC++编译文件使用！`zjlj.lib`联接编译（静态调用）
- 编译好的dll文件可以用VC++6自带的工具`Depend`查看

### 二、`dll_stdcall_def`目录

- 是为其他编译环境调用dll使用，`_stdcall`调用约定,可静态，动态调用（解决名字修改问题）
- 例如`delphi7`目录下的delphi编译文件使用！（动态调用）

### 三、`dll_cdecl_def`目录

- 是为其他编译环境调用dll使用，`_cdecl`调用约定,可静态，动态调用（解决名字修改问题）
- 例如`str`目录下的VC++编译文件使用！`zjlj.lib`联接编译（动态调用）

## （二）example目录说明

1. `str`目录：在VC++下使用`zjlj.lib`联接编译最后调用dll的实例（编译环境设置调入dll）
2. `strnew(cdecl)静态`目录：在VC++下使用`zjlj.lib`联接编译最后（静态）调用dll的实例（代码调入dll）
3. `strnew(cdecl)动态`目录：在VC++下使用显试（动态）`cdecl`调用dll的实例
4. `strnew(stdcall)静态`目录：在VC++下使用`zjlj.lib`联接编译最后（静态）调用dll的实例（代码调入dll）
5. `strnew(stdcall)动态`目录：在VC++下使用显试（动态）`stdcall`调用dll的实例
6. `delphi7`目录：delphi编译文件调用dll文件的实例

注意：由于压缩包里面包含了`delphi7`编译的实例，360杀毒软件可能会把它当成病毒进行警告

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-6-23 14:27

被lovecra编辑

，原因：

上传的附件：

- [`dllexample.rar`](javascript:void(0);)

  （2.40MB，77次下载）
