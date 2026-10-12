
附上资源脚本

代码：

```
#define IDD\_DLG1 1000
#define IDC\_BTN1 1001
#define IDC\_CHK1 1002
#define IDC\_STC1 1003
#define IDC\_STC2 1004
#define IDC\_EDT1 1005
#define IDC\_STC3 1006
#define IDC\_EDT2 1007
#define IDC\_BTN2 1008
#define IDR\_VERSION 1
IDD\_DLG1 DIALOGEX 6,6,99,108
CAPTION "SCTaDD\_BY\_eASYSCt"
FONT 8,"MS Sans Serif",0,0
STYLE 0x10CA0000
BEGIN
  CONTROL "打开文件",IDC\_BTN1,"Button",0x50010000,4,1,92,15
  CONTROL "备份源文件",IDC\_CHK1,"Button",0x50010003,4,20,92,15
  CONTROL "",IDC\_STC1,"Static",0x50000000,2,40,94,13
  CONTROL "新区段名",IDC\_STC2,"Static",0x50000000,2,60,36,9
  CONTROL "",IDC\_EDT1,"Edit",0x50010000,46,57,50,15,0x00000200
  CONTROL "新区段大小",IDC\_STC3,"Static",0x50000000,2,77,42,9
  CONTROL "",IDC\_EDT2,"Edit",0x50010000,46,73,50,15,0x00000200
  CONTROL "增加区段",IDC\_BTN2,"Button",0x50010000,4,90,92,13
END
IDR\_VERSION VERSIONINFO
FILEVERSION 1,0,0,0
PRODUCTVERSION 1,0,0,0
FILEOS 0x00000004
FILETYPE 0x00000000
BEGIN
  BLOCK "StringFileInfo"
  BEGIN
    BLOCK "FFFF0000"
    BEGIN
      VALUE "FileVersion", "1.0.0.0\\0"
      VALUE "ProductVersion", "1.0.0.0\\0"
      VALUE "CompanyName", "hIMcrACk\\0"
    END
  END
  BLOCK "VarFileInfo"
  BEGIN
    VALUE "Translation", 0xFFFF, 0x0000
  END
END
```

玩命 大侠的代码在 http://bbs.pediy.com/showthread.php?p=467116 有详细注释的，像我一样的小菜虫可以去阅读~

其中函数 `LpstrToHex` 为自己优化后的字符串转 16 进制整型变量的函数，算法上不算高明，结构上不算美观，不过能够达到效用。

有需要的朋友可以拿去使用~~

期待大牛们逆出效率更高、结构更漂亮的函数，以供收藏……

天快亮了，就不多写了，最后把工具上传，喜欢就拿去用~

感谢你耐心的看到这里，为能忍受菜菜的笔记所感谢~

### 【经验总结】

PE 文件结构基础、asm 的练习、细心 coding、耐心 debuging 是最大收获

`tHAT is ALl`

THX!!!!

2008 年 11 月 23 日 6:16:55

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `SCTaDD.rar` |
| :--- | :--- |
