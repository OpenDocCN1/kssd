
![](https://bbs.kanxue.com/upload/attach/202006/617776_HUCM8WKY6CT29J5.png)

## 0x3 调用系统字库

游戏中通常用`CreateFontIndirectA`来选择字库，函数如下：

```
HFONT CreateFontIndirectA( const LOGFONTA *lplf);

typedef struct tagLOGFONTA {

  LONG lfHeight;

  LONG lfWidth;

  LONG lfEscapement;

  LONG lfOrientation;

  LONG lfWeight;

  BYTE lfItalic;

  BYTE lfUnderline;

  BYTE lfStrikeOut;

  BYTE lfCharSet; //0x17

  BYTE lfOutPrecision;

  BYTE lfClipPrecision;

  BYTE lfQuality;

  BYTE lfPitchAndFamily;

  CHAR lfFaceName[LF_FACESIZE];

} LOGFONTA, *PLOGFONTA, *NPLOGFONTA, *LPLOGFONTA;

```

日文游戏显示乱码原因多半是`lfCharSet`值输入为1（`ANSI_CHARSET` 0 `BALTIC_CHARSET` 1，跟随系统），语言根据徐通决定，我们要改成0x80 日语，汉化后要改成0x86 中文。其中`lfCharSet`大多在`CreateFontIndirectA`上面。

比如说改为，`mov dword ptr ds:[532798], 80000000`，修改乱码如图所示。

![](https://bbs.kanxue.com/upload/attach/202006/617776_JZ2HJSWD92ARVCG.jpg)

## 0x4 枚举系统字库

游戏中选择字体的时候通常用`EnumFontFamiliesA`或`EnumFontFamiliesExA`来枚举可用字库，通常会在`FONTENUMPROCA`回调函数中，通过`charset`来过滤其他字库。

如：

```
mov edi, dword ptr ss:[ebp + 8] ;arg1      
cmp byte ptr ds:[edi + 17], 80  ;japanese
```

这个函数声明如下, `charset`就在`LOGFONT`结构体里， `charset` 0x80为日语， 0x86为中文。

```
int CALLBACK EnumFontFamProc(

    ENUMLOGFONT FAR *lpelf,   // pointer to logical-font data

    NEWTEXTMETRIC FAR *lpntm, // pointer to physical-font data

    int FontType,             // type of font

    LPARAM lParam             // address of application-defined data  );

int EnumFontFamiliesExA(

  HDC           hdc,

  LPLOGFONTA    lpLogfont,

  FONTENUMPROCA lpProc,

  LPARAM        lParam,

  DWORD         dwFlags);
typedef struct tagENUMLOGFONT { // elf 
    LOGFONT elfLogFont; //这项结构如tagLOGFONTA所示
    BCHAR    elfFullName[LF_FULLFACESIZE];
    BCHAR    elfStyle[LF_FACESIZE];
} ENUMLOGFONT;
```

## 0x5 其他

### (1) 系统语言检测相关

一般来说常用的函数是`GetSystemDefaultLCID`，返回值是系统语言代码，英文 0x409，日文 0x411， 简中0x804，繁中0x404。

### (2) 游戏窗口乱码修复

游戏窗口标题可能存储在游戏资源文件里，不一定在exe里面，通过`CreateWindowEx`来初始化窗口标题，或者`SetWindowText`。

### (3) 点阵字模

`GetGlyphOutlineA` 将字符转换成点阵来输出，有些游戏的字体输出都是调用此函数，一般都会在调用这个函数之前进行字符边界校验。具体与上面类似，不再赘述。函数声明为：

```
DWORD GetGlyphOutlineA(
  HDC            hdc,
  UINT           uChar,
  UINT           fuFormat,
  LPGLYPHMETRICS lpgm,
  DWORD          cjBuffer,
  LPVOID         pvBuffer,
  const MAT2     *lpmat2);
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-6-12 16:07

被devseed编辑

，原因：
