
`00FACDEF   push 0FACE14             ASCII "103"`

`00FAD289   push 0FAD330             ASCII "42"`

`00FAD4EB   push 0FAD520             ASCII "151"`

`00FADAED   push 0FADD50             ASCII "85"`

`00FADB07   push 0FADD50             ASCII "85"`

`00FADC87   push 0FADD60             ASCII "55"`

`00FADCE6   push 0FADD70             ASCII "57"`

`00FADD2A   push 0FADD80             ASCII "60"`

`00FADE83   push 0FADF18             ASCII "45"       ；双击来到代码处`

`00FADEF0   push 0FADF28             ASCII "34"`

`00FAE4CE   push 0FAE638             ASCII "100"`

`00FAE5FE   add byte ptr ds:[eax],al  （初始 CPU 选择）`

`00FAE6B0   push 0FAE6D4             ASCII "150"`

`00FAE7DC   push 0FAE84C             ASCII "170"`

### 3、看到了上面的 45 号错误了吧

```
00FADE70     E8 A36CFDFF        call 00F84B18

00FADE75     8BE8              mov ebp,eax

00FADE77     892D D495FB00     mov dword ptr ds:[FB95D4],ebp

00FADE7D     3B6C24 28         cmp ebp,dword ptr ss:[esp+28]  ; 先在这里下 he 断点

00FADE81     74 0C           je short 00FADE8F        ; 如被检测到这里肯定没有跳

00FADE83     68 18DFFA00     push 0FADF18             ; ASCII "45"

00FADE88     E8 FB79FDFF     call 00F85888
```

### 4、在 00FADE7D 下好硬件执行断点后，我们忽略所有异常，重载，F9 运行中断在 00FADE7D，先来探探情况

堆栈 `ss:[0012FF5C]=5E2582B0`，`ebp=5E2582B0`

按这个值比较，`00FADE81` 是会跳的，因为我们这样调试并没有影响到内存校验嘛。那么 `5E2582B0` 就是内存校验值了。OK，情况明了

我们在数据窗中转存一下 `0012FF5C` 看看

```
0012FF14  B0 82 25 5E D0 C0 06 01  皞%^欣         ；这里也有校验值

0012FF1C  1F 2C F7 00 55 4B F8 00  ,?UK?

0012FF24  00 10 00 00 00 80 0E 00  ....€.

0012FF2C  00 7C FB 00 75 DE FA 00  .|?u 搡.

0012FF34  00 00 00 00 00 00 40 00  ......@.

0012FF3C  00 00 00 00 00 10 00 00  .......

0012FF44  00 00 00 00 00 00 00 00  ........

0012FF4C  00 00 00 00 00 00 00 00  ........

0012FF54  00 00 00 00 00 9E 0D 00  .....?.

0012FF5C  B0 82 25 5E 00 00 00 00  皞%^....         ；校验值
```

### 5、我们删除所有的断点包括刚才的硬件执行断点，重新在 00FADE70 下硬件执行断点，重载，F9 中断在 00FADE70，继续探探情况

```
00FADE70     E8 A36CFDFF        call 00F84B18          ; 这里就是计算校验值的了

00FADE75     8BE8              mov ebp,eax             ; 给 eax 返回 5E2582B0 校验值

00FADE77     892D D495FB00     mov dword ptr ds:[FB95D4],ebp  ; ebp 里也是校验值
```

### 6、我们来做一个小实验，重载，F9

```
00FADE70     E8 A36CFDFF        call 00F84B18          ; F8，注意观察转存中的 0012FF14 和 0012FF5C

00FADE75     8BE8              mov ebp,eax             ; 手动给 eax 清 0

00FADE77     892D D495FB00     mov dword ptr ds:[FB95D4],ebp  ; 0，0

00FADE7D     3B6C24 28         cmp ebp,dword ptr ss:[esp+28]  ; 0，5E2582B0

00FADE81     74 0C           je short 00FADE8F        ; 没跳，让人高兴的在后面

00FADE83     68 18DFFA00     push 0FADF18             ; ASCII "45"

00FADE88     E8 FB79FDFF     call 00F85888            ; 提示 Protection Error:45
```

需要注意的是，在我们手动更改 `eax=0` 的时候，`0012FF14` 和 `0012FF5C` 里面的值是没变的，都是校验值。

### 7、OK，那我们可以大胆的假设性（也是有事实根据的）分析一下

1. 若 `ASPr` 检测到内存校验，那么 `00FADE70     call 00F84B18` 将会给 `eax` 返回为 0 或者是除了正确校验值之外的其他值
2. 若 `ASPr` 没有检测到，那么一切如常

分析完了，我们就有必要到 `00FADE70     call 00F84B18` 去看看了

有的朋友可能会说，那我把 `00FADE81` 改成 `jmp` 不就不管什么情况都可以跳过内存校验吗？呵呵，你不要忘了，壳也有自身校验的，这也是为什么我们在使用补区段法的时候，还要修改 `Route Check` 的缘故了

如果在内存补丁当中这样去设置

![](img/facad77f3d4c00673b2b0e650bf9415c.jpg)

那么将会出现这样的错误提示

![](img/f4df1db09cc0187cb484eba129607a75.jpg)

### 8、重载，F9，F7

```
00F84B18     53                  push ebx

00F84B19     56                  push esi

00F84B1A     57                  push edi

00F84B1B     8BFA                mov edi,edx

00F84B1D     8BF0                mov esi,eax

00F84B1F     B2 01               mov dl,1

00F84B21     A1 20F8F700        mov eax,dword ptr ds:[F7F820]

00F84B26     E8 B9E0FEFF        call 00F72BE4

00F84B2B     8BD8                mov ebx,eax

00F84B2D     8BC3                mov eax,ebx

00F84B2F     8B10                mov edx,dword ptr ds:[eax]

00F84B31     FF12                call dword ptr ds:[edx]

00F84B33     8BD6                mov edx,esi

00F84B35     8BCF                mov ecx,edi

00F84B37     8BC3                mov eax,ebx

00F84B39     8B30                mov esi,dword ptr ds:[eax]

00F84B3B     FF56 04           call dword ptr ds:[esi+4]

00F84B3E     8BC3                mov eax,ebx

00F84B40     8B10                mov edx,dword ptr ds:[eax]

00F84B42     FF52 08           call dword ptr ds:[edx+8]

00F84B45     8BC3                mov eax,ebx

00F84B47     8B10                mov edx,dword ptr ds:[eax]

00F84B49     FF52 0C           call dword ptr ds:[edx+C]

00F84B4C     8B30                mov esi,dword ptr ds:[eax]

00F84B4E     8BC3                mov eax,ebx

00F84B50     E8 BFE0FEFF        call 00F72C14          ; 这里给 0012FF14 赋校验值

00F84B55     8BC6                mov eax,esi            ; 正常情况下 esi 里的值是校验值

00F84B57     5F                  pop edi              ; 需要注意的是在这整个过程当中 0012FF5C 始终没变，都是校验值

00F84B58     5E                  pop esi

00F84B59     5B                  pop ebx

00F84B5A     C3                  retn
```

上面我们分析了，通过改跳转是行不通的，调试发现 `call 00F84B18` 里面的值是不会被检测的

### 9、重载，F9，F7，修改之

```
00F84B18     8B4424 2C          mov eax,dword ptr ss:[esp+2C]  ; [esp+2C]=0012FF5C

00F84B1C     C3                  retn                   ; 回去吧

00F84B1D     8BF0                mov esi,eax

00F84B1F     B2 01               mov dl,1

00F84B21     A1 20F8F700        mov eax,dword ptr ds:[F7F820]
```

即从 `00F84B18` 开始的原始数据

`53 56 57 8B FA`

改之

`8B 44 24 2C C3`

![](img/856c3a3c3e341c2ce25e231090a7fb7c.jpg)

在 `Dup2` 或者 `keymake` 里面这样设置之后，就不会被检测到了，但会有 1%的不稳定性，呵呵，带壳的内存补丁基本上都会遇到这样的问题。

好啦，到此为止，`ASPr` 的内存校验算是分析完了。

## 四、补丁工具的功能

功能 ：

1. 可以根据用户输入的数据信息，生成相对应的补丁
2. 支持用户对数据的添加、修改、删除功能
3. 双击要修改的数据即可对原数据进行编辑
4. 支持数据排序功能，用户只需拖曳相应数据即可
5. 支持自动添加连续数据功能
6. 支持查看窗体名功能
7. 支持工程文件保存、打开功能
8. 增加了目标文件拖曳功能
9. 增加了工程文件拖曳功能
10. 增加了窗体捕捉功能
11. 增加了用户个人信息设置

后期更新：

1. 增加补丁程序和原程序整合为一个程序功能
2. 增加多个连续数据的支持
3. 增加补丁样式的选择

其他特点：

1. 水纹效果
2. 无标题栏拖曳效果

为了程序的美化等因素，使用了大量的第 3 方控件

关于补丁工具的演示请看演示录象

建议：最好不要使用“用户个人信息设置”，因为这样会影响到补丁的启动速度。

## 五、几款补丁工具的对比

几款补丁工具的对比可以看演示录象，一看就一目了然：）

## 六、补丁工具原理

就目前流行的几种补丁工具来看，都是使用通过查找进程名来定位目标程序，再通过目标地址利用原始数据对比来进行 Patch 补丁数据。然后经过本人测试发现，对付像 `ASPr` 一类的通过内存校验来完成反 Loader 功能的壳而言，此方法会被检测到。

于是本人转换了种方式看下文

主程序部分：把事先写好的补丁模块以资源形式加入到主程序中，根据用户填写的补丁信息生成一个 ini 文件，然后把 ini 文件捆绑到释放的补丁模块末尾,所以用 `PEID` 给补丁查壳是 `Borland Delphi 6.0 - 7.0 [Overlay]`

补丁部分：补丁程序释放捆绑在末尾的 ini 文件，再读取 ini 文件内的信息对目标程序进行补丁。 本补丁是通过窗体名定位目标程序并非常规补丁工具那样使用的是进程名来定位，这样就基本可以逃过壳的内存检测了。

经测试，若开有杀毒软件，会导致出现 `Dup2` 和 `KeyMake` 一样的不稳定情况，但是，不会有错误提示

## 七、源码

先看下流程图吧

![](img/f6ec15509c29fa10b0a542fd2c7756a8.jpg)

主程序源码

引用: `unit Unit1;`

```
interface

  uses
    Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
    Dialogs, ExtCtrls, SUIForm, StdCtrls, SUIButton, SUIEdit, XPMan,
    CnWaterImage, SUIMemo, jpeg, ComCtrls, SUIStatusBar, SUISideChannel,
    CheckLst, EnhListView, Inifiles, ShellAPI, About, SUIListView, FindWindow,
    SUIImagePanel, SUIGroupBox;

  type
    TForm1 = class(TForm)
      XPManifest1: TXPManifest;
      Mac: TsuiForm;
      GroupBox1: TGroupBox;
      CnWaterImage1: TCnWaterImage;
      NameTxt: TsuiEdit;
      TitileTxt: TsuiEdit;
      Generent: TsuiButton;
      Exit: TsuiButton;
      Label1: TLabel;
      Label2: TLabel;
      suiStatusBar1: TsuiStatusBar;
      OpenDialog1: TOpenDialog;
      SaveDialog1: TSaveDialog;
      look: TsuiButton;
      GroupBox2: TGroupBox;
```
