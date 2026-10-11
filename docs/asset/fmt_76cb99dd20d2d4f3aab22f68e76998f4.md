
# 基于 Winamp 输入模块编写音乐播放器的一点心得

> 原文：[`www.pediy.com/kssd/pediy11/110376.html`](https://www.pediy.com/kssd/pediy11/110376.html)

--------=========================----------------

Goldenspdier[G-Spider 月窟]

--------=========================----------------

于 2010 年 4 月 4 日星期天，八小时未进米食，一气呵成，纯 Win32 汇编环境(`masm32 V10.0`)。

参考其中自带的 example(`LCD 界面设计 2000 by Thomas Bleeker [exagone]`)。

在几十 KB 的代码中游走实属不易，愿为看雪做一点贡献。 完成之后心情超好啦~~~~

### 使用说明：(初始版本）

1. 点选 `menu` 弹出文件窗口，得到音频文件[支持 `mp3` 格式]
2. 点 `play` 播放（注意不要重复点击该按钮，否则不能保证流畅播放）
3. 调节音量有 `+`, `-` 按钮
4. 部分功能有待完善

### 更新说明：  （更新版本,已上传）

1. 接口实现上有很大调整，有助于扩展成支持多种格式。
2. 支持 `mp3` 格式,新增对 `.FLAC` 无损音频文件的支持，以满足音乐发烧友对音质的要求

### 使用说明：

1. 点选 `menu` 弹出文件窗口，得到 `mp3`, `flac` 音频文件
2. 点 `play` 播放（注意不要重复点击该按钮，否则不能保证流畅播放）
3. 调节音量有 `+`, `-` 按钮
4. 部分功能有待完善

### 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `lcd.rar` |
| :--- | :--- |

之前并没有考虑要扩展支持多种音频格式，对于多种格式支持时出现了小麻烦。

开始时，准备用 `In_Module` 中的

```
int (*IsOurFile)(char *szFile); //  检查文件格式
```

发现不好用，无论与否好像都返回 0，这样我就没办法了,并且占 `CPU` 很多。

于是自己搞个扩展名识别也行的，如对于 `.mp3` 和 `.flac` 格式分别调用各自的模块就搞定了。

所以要实现扩展名识别。

------------------------------------

`PathFindExtension`  //得到文件扩展名 `api`

Searches a path for an extension.

```
LPTSTR PathFindExtension(
    LPCTSTR pPath  //输入参数是一个路径地址，返回扩展名首址；
      //如"D:\masm32\lcd\lcd.asm",则可返回".asm"的首址
    );      //汇编中返回值在 eax 中
```

Parameters

`pPath`

[in] Pointer to a null-terminated string of maximum length `MAX_PATH` that  contains the path that contains the extension for which to search.

Return Values

Returns the address of the `.` preceding the extension within `pPath`  if an extension is found, or the address of the trailing `NULL` character otherwise.

Requirements

  Version 4.71 and later of `Shlwapi.dll`

  Windows `NT/2000`: Requires `Windows 2000` (or `Windows NT 4.0` with `Internet Explorer 4.0` or later).

  Windows `95/98/Me`: Requires `Windows 98` (or `Windows 95` with `Internet Explorer 4.0` or later).

  Header: Declared in `Shlwapi.h`.  //masm32 中用 `include Shlwapi.inc`

  Import Library: `Shlwapi.lib`.

------------------------------------

功能实现如下:

```
.data
stOpenFileName  OPENFILENAME    <?>
szTitleSave  db  "Open MUSIC file...",0
szExt    db  '*.*',0
szMp3    db  '.mp3',0
szFilter  db  'MUSIC Files(*.mp3,*.flac)',0,'*.*',0,0

.data?
szBuffer  db  256   dup  (?)

.........

;*********************************************************************

//利用托此控件来得到我们所要的路径地址（在 szBuffer 缓存中）
_GetFileName  proc

    mov  stOpenFileName.Flags,OFN_PATHMUSTEXIST or OFN_FILEMUSTEXIST
    mov  stOpenFileName.lStructSize,SIZEOF stOpenFileName
    mov  eax,hDlg
    mov  stOpenFileName(hWndOwner,eax
    mov  stOpenFileName.lpstrFilter,offset szFilter  ;扩展名
    mov  stOpenFileName.lpstrFile,offset szBuffer  ;文件名路径缓冲
    mov  stOpenFileName.nMaxFile,255      ;文件名缓冲长度
    mov  stOpenFileName.lpstrInitialDir,0
    mov  stOpenFileName.lpstrTitle,offset szTitleSave
    mov  stOpenFileName.lpstrDefExt,offset szExt

    invoke  GetOpenFileName,offset stOpenFileName

    .if  eax == FALSE
      ret
    .endif
    ret

_GetFileName  endp

;*********************************************************************

invoke   _GetFileName
invoke  PathFindExtension,offset szBuffer//得到扩展名地址（返回值在 eax 中）
invoke  lstrcmpi, offset szMp3, eax  //进行比较两个扩展名字符串是否相等（如果相等则返回 0)

.if  eax==0

  ........//做出相应动作
```
