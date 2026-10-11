
# 事件分析与逆向工程笔记

`OnClick = Button1Click`------->触发的 `TButton` 事件名，并把指针给了 `OnClick` 做了代理，搜 1 下 `Button1Click` 可以定位到其事件入口

```
end
```

用 Dede 反编译没有找出触发 Button 按钮的事件地址，但是点击 Button 的确有注册 Neg 出现来...感觉里边有东东在作梗，终于在这里找出了眉毛来。

仔细看看 `TControl.Click` 里边调用了三个虚函数第一个是 `TAction` 类的 `TAction.OnExecute` 第二是 `TActionLink.Execute`，第三个才是 `TButton.FOnClick(Self)`

看上边的 Delphi Form 资源 `OnClick` 代理的是第三个函数的指针即是:`TButton.FOnClick(Self)` 这也是通常我们需要的 Button 事件。而其它两个被疏忽掉的事件才是真正本片的内幕，它可害苦了我，害的我苦苦遍历了这三个事件。纯熟于个人巧合，在简单的说一下，当用 Dede 编译的时候没有发现 Button 的事件处理地址，但是点击 Button 还出现 Neg，这说明有事件响应了，(后来才知道在 `TfrmMain` 中有个 `TActionList` 控件，它内部的 `Action` 对象代理这个 Button 的事件处理)，开始怀疑是反调试了反 Dede，其实都是这 `TAction` 对象在作梗，害的我掘出 VCL 一点点跟...本片莫非虚构的，若有雷同纯属巧合:=)

```
00456050   $  53             push    ebx
00456051   |.  8BD8          mov     ebx, eax
00456053   |.  66:83BB 22010>cmp     word ptr [ebx+122],0--->先比较 FOnClick 事件地址高二位字节是否为 0,如果为 0 说明没有 TButton.FOnClick(Self)事件
0045605B   |.  74 2D         je      short 0045608A--->TButton.FOnClick(Self)空否,就跳.
0045605D   |.  8BC3         mov     eax, ebx---->TButton 对象指针给 eax
0045605F   |.  8B10         mov     edx, [eax]----->TSpeedButton 虚函数表给 edx
00456061   |.  FF52 3C      call    [edx+3C]---------------------->虚函数 TAction.OnExecute,由 eax 返回 TAction 对象地址
00456064   |.  85C0         test    eax, eax---->空就跳
00456066   |.  74 22        je      short 0045608A
-----------------------------------------------------
00456096   |.  8BD3         mov     edx, ebx--->TSpeedButton 对象地址给 edx
00456098   |.  8B43 6C      mov     eax, [ebx+6C]---->取出 TSpeedButton 内包容对象 SpeedButtonActionLink
0045609B   |.  8B08         mov     ecx, [eax]--->SpeedButtonActionLink 对象虚表给 ecx
0045609D   |.  FF51 18      call    [ecx+18]--------------------虚函数 TActionLink.Execute(Self)这才是真正的事件处理
-----------------------------------------------------
004560A0   |.  EB 18        jmp     short 004560BA
004560A2   |>  66:83BB 22010>cmp     word ptr [ebx+122],>
004560AA   |.  74 0E        je      short 004560BA
004560AC   |.  8BD3         mov     edx, ebx
004560AE   |.  8B83 24010000mov     eax, [ebx+124]
004560B4   |.  FF93 20010000call    [ebx+120]  -------------->TButton.FOnClick(Self)
004560BA   |>  5B          pop     ebx
004560BB   \\.  C3         retn
```

## 终点

看 Delphi Form 资源定义

```
object acApply: TAction
    Caption = #24212#29992#27492#30456#26694
    Hint = #24212#29992#27492#30456#26694
    ImageIndex = 2
    OnExecute = acApplyExecute--------------------->这才是 TSpeedButton 的响应的事件.兜了一个大圈圈出来.
end
```

从上边的分析 `TSpeedButton` 的 `OnClick` 没有被赋值，而是把 `TSpeedButton` 事件加入 `TActionLink` 中关联了一个 `Action` 对象，并且这个 `TAction` 事件名为 `acApplyExecute`。

用 Dede 分析 `TActionLink` 也能得到目标地址。

直接来到 `TAction.OnExecute` 事件处理，也就是 `TSpeedButton` 的事件处理，Neg 窗体就在这个事件中出现了...

```
00563E30   /.  55            push    ebp
00563E31   |.  8BEC          mov     ebp, esp
00563E33   |.  83C4 C0      add     esp, -40---------->看下面提取来关键变量
00563E36   |.  53            push    ebx
00563E37   |.  56            push    esi
00563E38   |.  57            push    edi
00563E39   |.  33C9          xor     ecx, ecx
00563E3B   |.  894D C0      mov     [ebp-40], ecx
00563E3E   |.  894D C8      mov     [ebp-38], ecx
00563E41   |.  894D CC      mov     [ebp-34], ecx
00563E44   |.  894D D0      mov     [ebp-30], ecx
00563E47   |.  894D F4      mov     [ebp-C], ecx
00563E4A   |.  8BD8          mov     ebx, eax
00563E4C   |.  33C0          xor     eax, eax
```

### 提取局部变量

`PChar_[ebp-40]`, `PChar_[ebp-38]`, `pChar_[ebp-34]`, `PChar_[ebp-30]`, `PChar_[ebp-C]`

### 提取关键比较 Boolean 变量

`Boolean_[ebp-20]`, `Boolean_[ebp-21]`

```
00563E4E   |.  55            push    ebp
00563E4F   |.  68 FC415600   push    005641FC ----> 新 SEH 处理句柄
00563E54   |.  64:FF30       push    dword ptr fs:[eax]
00563E57   |.  64:8920       mov     fs:[eax], esp ----->SEH 构造完毕.
00563E5A   |.  C645 E0 00    mov     byte ptr [ebp-20], 0  -----> 初始化 Boolean 变量[ebp-20]:=false
00563E5E   |.  C645 DF 00    mov     byte ptr [ebp-21], 0  -------->初始化 Boolean 变量[ebp-20]:=flase
00563E62   |.  8B83 E0040000mov     eax, [ebx+4E0]  ------> [ebx]为 TfrmMain 类对象的堆地址[ebx+4E0]ImageEnView_ Main 对象
00563E68   |.  E8 170AFDFF   call    00534884 ----->检查 [ebx+4B8]_TImageEnIO
00563E6D   |.  8B80 64010000mov     eax, [eax+164]   ------> 定位到 TIOParamsVals 类对象...
00563E73   |.  8378 0C 00    cmp     dword ptr [eax+C], 0  --> [eax+c] 指向选择的文件名缓冲区指针
00563E77   |.  75 1A         jnz     short 00563E93  -->  如果文件名缓冲空就提示请选择图片对话框
00563E79   |.  6A 00         push    0
00563E7B   |.  B9 0C425600   mov     ecx, 0056420C --> "提示"
00563E80   |.  BA 14425600   mov     edx, 00564214-------->"请先选择图片"
00563E85   |.  A1 38695700   mov     eax, [576938]--------> TApplication 对象指针
00563E8A   |.  8B00          mov     eax, [eax]------>TApplication 虚表
00563E8C   |.  E8 B715F1FF   call    00475448------------>弹出请先选择图片对话框
00563E91   |.  EB 04         jmp     short 00563E97
00563E93   |>  C645 E0 01    mov     byte ptr [ebp-20], 1  -->  Boolean 变量[ebp-20]:=true
00563E97   |>  8B83 60030000mov     eax, [ebx+360]  --> 定位到 TComboBox 类对象对象.
00563E9D   |.  8B10          mov     edx, [eax]   -----> TComboBox 对象地址给 edx,准备取用户选择 CombBox 的索引值
00563E9F   |.  FF92 CC000000call    [edx+CC] ----->  TCustomCombo.GetItemIndex: Integer; SendMessage(Handle, CB_GETCURSEL, 0, 0);
00563EA5   |.  85C0          test    eax, eax      ----->  如果用户选择了大于 1 的 Combox 索引值,下边就提示注册
00563EA7   |.  7E 2B         jle     short 00563ED4   ----->  小于等于 0 就跳
00563EA9   |.  6A 24         push    24
00563EAB   |.  B9 24425600   mov     ecx, 00564224   --> "提示注册信息购买正版..."
00563EB0   |.  BA 34425600   mov     edx, 00564234  ----------> "提示注册信息购买正版..."
00563EB5   |.  A1 38695700   mov     eax, [576938]
00563EBA   |.  8B00          mov     eax, [eax]
00563EBC   |.  E8 8715F1FF   call    00475448  --------> Neg 对话框此时就出现了.上边有两个 Button 控件,
00563EC1   |.  83F8 06       cmp     eax, 6   ---------> 比较两 button ID 号...
00563EC4   |.  75 12         jnz     short 00563ED8  -------> 不等于 6 就跳
00563EC6   |.  BA 28435600   mov     edx, 00564328 ------->ASCII "http://www.softreg.com.cn/...."
00563ECB   |.  8BC3          mov     eax, ebx
00563ECD   |.  E8 96060000   call    00564568  -------->函数调用 Shell32.ShellExecuteA 输出 Soft's Web
00563ED2   |.  EB 04         jmp     short 00563ED8
00563ED4   |>  C645 DF 01    mov     byte ptr [ebp-21], 1  --------> Boolean 变量[ebp-20]:=true
00563ED8   |>  8A45 E0       mov     al, [ebp-20]  -----------> 把 Boolean 变量[ebp-20]给 al
00563EDB   |.  2245 DF       and     al, [ebp-21]
00563EDE   |.  0F84 ED020000je      005641D1  --------------> 这是关键 jump 跳了 game over 与 Boolean[ebp-20]、[ebp-21]有关系
```

## 图片资源结构

在软件目录中包含 `frames` 目录，`frames` 目录又有 0-7 个 8 个图片文件夹，0 包含了以 a,b,c 开头的文件，1 文件夹只包含了 b 开头图片文件，2-7 文件夹是放的是 c 开头的文件。

*   以 a 开头的图片文件(比如 `a1002.jpg` 放在 0 文件夹内),jpg 文件中是黑象素渲染的图片框
*   以 b 开头的图片文件(比如 `b1002.jpg` 放在 1 文件夹内),jpg 文件中是黑象素渲染的图片外框,白象素渲染的图片内框
*   以 c 开头的图片文件(比如 `c1002.jpg` 放在 2-7 文件夹内),jpg 文件中是彩色的渲染的图片外框,同样白象素渲染的图片内框

就形状颜色象素的复杂程度应该是 a<b<c，所以这个软件只处理为数不多的简单的图片，其他复杂的 b,c 类型文件都没有处理也是放在 1-7 下边的文件几乎都不能处理。

注意一点:在 0 文件夹内图片都是通明背景，而其他 1-7 文件夹内图片全是白色背景估计需要用 PhotoShop 等图形处理软件处理 1 下估计才能用。我没有试猜测而已。

### 全局常量

```
const char_[0056439C]=a;
const char_[00564390]=b;
const char_[005643A8]=c;
```

### 局部变量

*   `PChar_[ebp-40]`:存放以 c 开头的文件
*   `PChar_[ebp-38]`:存放以 b 开头的文件
*   `pChar_[ebp-34]`:存放以 a 开头的文件
*   `PChar_[ebp-30]`:存放以 b 开头的文件
*   `PChar_[ebp-C]`:存放文件名与路径全名

### 判断是否以 b 开头的图片文件

```
00563F7B   |.  8D55 D0       lea     edx, [ebp-30]
00563F7E   |.  8B45 F4       mov     eax, [ebp-C] ----> [ebp-C]存放选择的图片路径和名称
00563F81   |.  E8 DE55EAFF   call    00409564      ----> SysUtils.ExtractFileName(AnsiString):AnsiString;提取文件名
```
