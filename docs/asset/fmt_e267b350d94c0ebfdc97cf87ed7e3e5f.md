
# FlashGet

> 原文：[`www.pediy.com/kssd/pediy06/pediy6590.htm`](https://www.pediy.com/kssd/pediy06/pediy6590.htm)

**PEDIY** 之 我自己的 **FlashGet** 面向初学者

**FlashGet** 地球人都知道吧，偶可是从以前的 `JetCar` 一直用过来的。看它的宣传中颇为自豪的是对下载来的文件的管理功能，可以分成软件、游戏、音乐之类来安排。可是偶还是用得不太爽，像偶这样每天 `DOWN` 一堆乱七八糟东东的，几个月就能累积起几百个文件，不少珍贵或是超大的东东是绝对舍不得先删了等需要时再下的。于是乎在七八百个压缩包中寻找三天前下载的一份资料就成了我的必修课，痛苦啊~~~~~

最近忽发奇想，`FLASHGET` 的文件管理只是简单的把文件分成了几类，要是把文件按下载的日期来分别放入不同的文件夹，岂不是十分方便？正好借此练练偶的 `Reversing Engineer`。

先确定大概的方向，`FLASHGET` 有一个预先指定的下载目录，默认时为 `C:\downloads`，如果下载时不加改变就会保存到这里，我们的目标就是根据不同的日期自动改变这个下载目录。

这个设定在什么地方呢，自己找找吧，`INI` 文件里没有，注册表里~~~~找到乐！是 `HKEY_USERS\.DEFAULT\Software\JetCar\JetCar\Download default`，里面有个 `path` 键值就是了。（我的 `FLASHGET` 版本为 1.60，新版不知有没有变化）

怎么修改呢，我想的方法是在软件启动时从注册表读取信息之前把用系统日期表示的默认下载目录写进注册表中。先用 `C` 语言写个，看你的编程功力啦：

```
代码:
void F()
{
  char path[30]="D:\\downloads\\";
  char date[10]={0};
  DWORD dw;
  HKEY hKey;
  GetDateFormat(NULL,DATE_SHORTDATE,NULL,NULL,date,10);
  RegCreateKeyEx(HKEY_USERS,".DEFAULT\\Software\\JetCar\\JetCar\\Download default",0,0,
    REG_OPTION_VOLATILE,KEY_ALL_ACCESS,NULL,&hKey,&dw);
  strcat(path,date);
  RegSetValueEx(hKey,"path",0,REG_SZ,(byte*)path,30);
  RegCloseKey(hKey);
}
```

就把这个函数插进程序刚启动时就行了。呵呵~~~，往人家的程序写代码可不是那么简单的，先做一点准备工作，把需要的常量查出来（偶最讨厌这个！）

```
HKEY_USERS=80000003
DATE_SHORTDATE=1
REG_OPTION_VOLATILE=1
KEY_ALL_ACCESS=1F003F （这个是根据下面一堆值加出来的）
REG_SZ=1
#define KEY_QUERY_VALUE         (0x0001)
#define KEY_SET_VALUE           (0x0002)
#define KEY_CREATE_SUB_KEY      (0x0004)
#define KEY_ENUMERATE_SUB_KEYS  (0x0008)
#define KEY_NOTIFY              (0x0010)
#define KEY_CREATE_LINK         (0x0020)
#define STANDARD_RIGHTS_ALL             (0x001F0000L)
#define KEY_ALL_ACCESS          ((STANDARD_RIGHTS_ALL        |\\
                                 KEY_QUERY_VALUE            |\\
                                 KEY_SET_VALUE              |\\
                                 KEY_CREATE_SUB_KEY         |\\
                                 KEY_ENUMERATE_SUB_KEYS     |\\
                                 KEY_NOTIFY                 |\\
                                 KEY_CREATE_LINK)            \\
                                 &                            \\
                                 (~SYNCHRONIZE))
```

找这个有什么用呢？在我们平时编程时候，不管用 `VC` 或是 `DELPHI` 还是 `MASM` 什么，都会包含一个有大量的常量定义的“头文件”，免去了记忆各种参数值的麻烦，可是现在我们几乎是用机器码写程序，你总不能在 `HIEW` 里写上 `PUSH REG_SZ` 吧（或许有高人能给 `HIEW` 扩充一下也说不定）。

常量记好没有，准备工作还没完呢，找找我们用到的函数的地址，用 `LordPE` 就很好，记得选上那个 `View always FirstThunk`。其中 `GetDateFormat` 这个函数没有，用 `LordPE` 加入也很方便的，在 `KERNEL32.DLL` 里面。

```
GetDateFormat  [54001E]
RegCreateKeyEx  [4DC020]
RegSetValueEx  [4DC040]
RegCloseKey  [4DC028]
lstrcat    [4DC370]
```

好了，开始写代码了。我们在程序入口点处就强行跳转到后面的大片空白处写我们的代码：

原来的代码：
