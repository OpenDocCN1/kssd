
# 一个汇编的多列控件示例

> 原文：[`www.pediy.com/kssd/pediy11/117035.html`](https://www.pediy.com/kssd/pediy11/117035.html)

```
; 第一步 invoke CreateClass,parent,_x,_y,_w,_h    ;初始控件的大小
; 返回 exe 为一个结构,根据自己需要可以对控件重构事件,不需要就不要理它了，呵呵
; base struct
; hXTree dd ?
; hTv dd   ?
; hList dd ?
; Header dd ?
; XTreeProc dd ?
; TvProc dd ?
; ListProc dd ?
; HeaderProc dd ?
; NewListProc dd ?
; NewTvProc dd ?
; base ends
```

```
; 第二步 ;invoke InsertColumn,@lx,@pText ;添加 Header 列大小及名称
; invoke InsertColumn,50,CADD("PID")
; invoke InsertColumn,80,CADD("PPID")
; invoke InsertColumn,80,CADD("Handles")
; invoke InsertColumn,80,CADD("Threads")
```

```
; 第三步 ;invoke TreeAddItem,offset ClassName ,0  ;添加 Tree 节点
; invoke TreeAddItem,offset ClassName ,edx
; eax 为一个双向链表数据结构
; ecx 返回总的行数
; edx 返回一个 TREE 节点句柄
```

```
; 第四步 ;invoke ListAddItem,edx,1,offset ClassName;添加对应节点的多列数据
; invoke ListAddItem,edx,FALSE,1,offset ClassName
; tree 节点句柄，LIST 列，设置或获取一个节点，添加到 LIST 字符串
; invoke ListAddItem,edx,TRUE,2,offset ClassName
```

**至此收功完毕，以下是 LIB 类导出函数的使用说明(\^\_以后在完善)**

`CreateClass` PROTO :DWORD, :DWORD, DWORD,:DWORD

\*;初始一个 Header。列宽度，列标题\*

`InsertColumn` PROTO :DWORD,:DWORD

\*;插入一个 Tree 节点。字符缓冲区指定一个节点名，父节点\*

`TreeAddItem` PROTO :DWORD,:DWORD

\*;设置或获得一个节点的数据项。 Tree 节点句柄，(FLASE 为添加一个数据项/TRUE 为设置一个数据项),要获得或设置数据的列索引，字符缓冲区\*

`ListAddItem` PROTO :DWORD, :DWORD, DWORD,:DWORD

\*;获得附加额外的数据项（最多 256 个 DWORD）每一个节点。节点句柄，返回缓冲区\*

`GetData_L` PROTO :DWORD, :DWORD

\*;删除所有节点\*

`Deleteallitems` PROTO

\*;枚举所有节点. Tree 节点句柄,一个回调函数接受句柄\*

`EnuemAllNodes` PROTO :DWORD, :DWORD

\*;枚举节点。Tree 节点句柄,搜索类型(TVGN_XXX),搜索方式(TVGN_NEXT/PREVIOUS) ;是否深度搜索(FALSE/TRUE),要搜索的节点句柄,一个回调函数接受句柄\*

`EnuemChildNodes` PROTO :DWORD,:DWORD,DWORD,:DWORD,DWORD,:DWORD

\*;弹出一个窗口分离当前节点的子节点数据《见图右边》。是否深度\*

`@@PopWindow` PROTO :DWORD

```
;以下是举例，例出当前进程
invoke CreateClass,hWin,10,10,500,300
invoke InsertColumn,170,CADD("Process")
invoke InsertColumn,50,CADD("PID")
invoke InsertColumn,80,CADD("PPID")
invoke InsertColumn,80,CADD("Threads")
invoke R3_ProcessSnapshot ;PS：这里不关心枚举进程的方式呵呵
```

![](img/36505dcc4faf8ecbf8546e11559aa8cc.jpg)

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 多列 Xtree 类示例.rar |
