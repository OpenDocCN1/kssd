
# 文档结构说明

## 1. 汇编代码片段

```asm
        invoke  Findsorteddata, offset bookmark.data, eax
        .IF eax
            invoke  Setcpu, 0, \[eax\].t_bookmark.addr_, 0, 0, CPU_ASMHIST or CPU_ASMCENTER or CPU_ASMFOCUS
        .ENDIF
    .ENDIF
.ENDIF
ret
ODBG_Pluginaction endp

end DllEntryPoint
```

这个是我在 `plugin.inc` 中弄出来对照的的

## 2. 结构定义

- 排列测试结构：`t_aligntest`
- 字符串搜索结构：`t_hexstr`
- 汇编命令搜索模块结构：`t_asmmodel`
- 扩展命令搜索模块结构：`t_extmodel`
- 颜色配置结构：`t_scheme`
- 可选择颜色配置结构：`t_schemeopt`
- bar 结构：`t_bar`
- 分类数据域 header 结构：`t_sortheader`
- 广义的分类表结构：`t_sorted`
- 使用分类数据和 bar 窗口结构：`t_table`
- 上下文(context)引用结构：`t_reg`
- 操作数结构：`t_operand`
- 汇编监控结构：`t_disasm`
- 表达式赋值的 Result 结构：`t_result`
- 活动线程信息结构：`t_thread`
- 内存描述块结构：`t_memory`
- 广义堆块结构：`t_heap`
- 指向字符串资源的结构：`t_stringtable`
- 关于固定或修正的结构：`t_fixup`
- 调试数据符号变量结构：`t_symvar`
- 跳转数据元素结构：`t_jdest`
- 可执行模块描述结构：`t_module`
- 当前或通用状况 dump 数据的结构：`t_dump`
- 参考描述结构：`t_ref`
- INT3 断点描述结构：`t_bpoint`
- 硬件断点结构：`t_hardbpoint`
- 注入代码描述结构：`t_inject`
- 资源排序描述符结构：`t_sourceline`
- 窗口描述结构：`t_window`

## 3. 函数定义

### 3.1 常规函数

- 添加到序列：`Addtolist`
- 更新序列：`Updatelist`
- 建立索引窗口：`Createlistwindow`
- 错误：`Error`
- 消息：`Message`
- 信息线路：`Infoline`
- 前进：`Progress`
- 反射：`Flash`

### 3.2 数据格式化函数

- `Decodeaddress` proto C `addr_:dword`, `base:dword`, `addrmode:dword`, `symb:ptr byte`, `nsymb:dword`, `comment_:ptr byte`
- `Decoderelativeoffset` proto C `addr_:dword`, `addrmode:dword`, `symb:ptr byte`, `nsymb:dword`
- `Decodecharacter` proto C `s:ptr byte`, `c_:dword`
- `Printfloat4` proto C `s:ptr byte`, `f:real4`
- `Printfloat8` proto C `s:ptr byte`, `double_1:dword`, `double_2:dword`
- `Printfloat10` proto C `s:ptr byte`, `double_1:dword`, `double_2:dword`
- `Print3dnow` proto `s:ptr byte`, `f:ptr byte`
- `Printsse` proto `s:ptr byte`, `f:ptr byte`
- `Followcall` proto `addr_:dword`
- `IstextA` proto C `c_:byte`
- `IstextW` proto C `w:word`
- `Stringtotext` proto C `data:ptr byte`, `ndata:dword`, `text:ptr byte`, `ntext:dword`

### 3.3 数据输入函数

- `Getlong` proto C `title_:ptr byte`, `data:dword`, `datasize:dword`, `letter:byte`, `mode:dword`
- `Getlongxy` proto C `title_:ptr byte`, `data:dword`, `datasize:dword`, `letter:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Getregxy` proto C `title_:ptr byte`, `data:dword`, `letter:byte`, `x:dword`, `y:dword`
- `Getline` proto C `title_:ptr byte`, `data:dword`
- `Getlinexy` proto C `title_:ptr byte`, `data:dword`, `x:dword`, `y:dword`
- `Getfloat10` proto C `title_:ptr byte`, `fdata:ptr real10`, `tag:ptr byte`, `letter:byte`, `mode:dword`
- `Getfloat10xy` proto C `title_:ptr byte`, `fdata:ptr real10`, `tag:ptr byte`, `letter:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Getfloat` proto C `title_:ptr byte`, `fdata:dword`, `size_:dword`, `letter:byte`, `mode:dword`
- `Getfloatxy` proto C `title_:ptr byte`, `fdata:dword`, `size_:dword`, `letter:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Getasmfindmodel` proto C `model_:ptr t_asmmodel`, `letter:byte`, `searchall:dword`
- `Getasmfindmodelxy` proto C `model_:ptr t_asmmodel`, `letter:byte`, `searchall:dword`, `x:dword`, `y:dword`
- `Gettext` proto C `title_:ptr byte`, `text:ptr byte`, `letter:byte`, `type_:dword`, `fontindex:dword`
- `Gettextxy` proto C `title_:ptr byte`, `text:ptr byte`, `letter:byte`, `type_:dword`, `fontindex:dword`, `x:dword`, `y:dword`
- `Gethexstring` proto C `title_:ptr byte`, `hs:ptr t_hexstr`, `mode:dword`, `fontindex:dword`, `letter:byte`
- `Gethexstringxy` proto C `title_:ptr byte`, `hs:ptr t_hexstr`, `mode:dword`, `fontindex:dword`, `letter:byte`, `x:dword`, `y:dword`
- `Getmmx` proto C `title_:ptr byte`, `data:byte`, `mode:dword`
- `Getmmxxy` proto C `title_:ptr byte`, `data:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Get3dnow` proto C `title_:ptr byte`, `data:byte`, `mode:dword`
- `Get3dnowxy` proto C `title_:ptr byte`, `data:byte`, `mode:dword`, `x:dword`, `y:dword`
- `Browsefilename` proto C `title_:ptr byte`, `name_:ptr byte`, `defext:ptr byte`, `getarguments:dword`
- `OpenEXEfile` proto C `path:ptr byte`, `dropped:dword`
- `Animate` proto C `animation:dword`

### 3.4 绘制或绘制过程函数

- `SORTFUNC` typedef proto `:ptr t_sortheader`, `:ptr t_sortheader`, `:dword`
- `PSORTFUNC` typedef ptr `SORTFUNC`
- `DRAWFUNC` typedef proto `:ptr byte`, `:ptr byte`, `:ptr dword`, `:ptr t_sortheader`, `:dword`
- `PDRAWFUNC` typedef ptr `DRAWFUNC`
- `DESTFUNC` typedef proto `:ptr t_sortheader`
- `PDESTFUNC` typedef ptr `DESTFUNC`

- `Createsorteddata` proto C `sd:ptr t_sorted`, `name_:ptr byte`, `itemsize:dword`, `nmax:dword`, `sortfunc:PSORTFUNC`, `destfunc:PDESTFUNC`
- `Destroysorteddata` proto C `sd:ptr t_sorted`
- `Addsorteddata` proto C `sd:ptr t_sorted`, `item:dword`
- `Deletesorteddata` proto C `sd:ptr t_sorted`, `addr_:dword`
- `Deletesorteddatarange` proto C `sd:ptr t_sorted`, `addr0:dword`, `addr1:dword`
- `Deletenonconfirmedsorteddata` proto C `sd:ptr t_sorted`
- `Findsorteddata` proto C `sd:ptr t_sorted`, `addr_:dword`
- `Findsorteddatarange` proto C `sd:ptr t_sorted`, `addr0:dword`, `addr1:dword`
- `Findsorteddataindex` proto C `sd:ptr t_sorted`, `addr0:dword`, `addr1:dword`
- `Sortsorteddata` proto C `sd:ptr t_sorted`, `sort:dword`
- `Getsortedbyselection` proto C `sd:ptr t_sorted`, `index:dword`
- `Defaultbar` proto C `pb:ptr t_bar`
- `Tablefunction` proto C `pt:ptr t_table`, `hw:HWND`, `msg:UINT`, `wp:WPARAM`, `lp:LPARAM`
- `Painttable` proto C `hw:HWND`, `pt:ptr t_table`, `getline:DRAWFUNC`
- `Gettableselectionxy` proto C `pt:ptr t_table`, `column:dword`, `px:ptr dword`, `py:ptr dword`
- `Selectandscroll` proto C `pt:ptr t_table`, `index:dword`, `mode:dword`

### 3.5 name 函数

- `Insertname` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Quickinsertname` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Mergequicknames` proto C
- `Discardquicknames` proto C
- `Findname` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Decodename` proto C `addr_:dword`, `type_:dword`, `name_:ptr byte`
- `Findnextname` proto C `name_:ptr byte`
- `Findlabel` proto C `addr_:dword`, `name_:ptr byte`
- `Deletenamerange` proto C `addr0:dword`, `addr1:dword`, `type_:dword`
- `Findlabelbyname` proto C `name_:ptr byte`, `addr_:ptr dword`, `addr0:dword`, `addr1:dword`
- `Findimportbyname` proto C `name_:ptr byte`, `addr0:dword`, `addr1:dword`
- `Demanglename` proto C `name_:ptr byte`, `type_:dword`, `undecorated:ptr byte`
- `Findsymbolicname` proto C `addr_:dword`, `fname:ptr byte`

### 3.6 解构函数

- `Disasm` proto C `src:ptr byte`, `srcsize:dword`, `srcip:dword`, `srcdec:ptr byte`, `disasm:ptr t_disasm`, `disasmmode:dword`, `threadid:dword`
- `Disassembleback` proto C `block:ptr byte`, `base:dword`, `size_:dword`, `ip:dword`, `n:dword`, `usedec:dword`
- `Disassembleforward` proto C `block:ptr byte`, `base:dword`, `size_:dword`, `ip:dword`, `n:dword`, `usedec:dword`
- `Issuspicious` proto C `cmd:ptr byte`, `size_:dword`, `ip:dword`, `threadid:dword`, `preg:ptr t_reg`, `s:ptr byte`
- `Isfilling` proto C `offset_:dword`, `data:ptr byte`, `size_:dword`, `align_:dword`

### 3.7 汇编函数
