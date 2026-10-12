
# PEDIY 之为 EXE 增加读取 INI 配置功能

> 原文：[`www.pediy.com/kssd/pediy12/142658.html`](https://www.pediy.com/kssd/pediy12/142658.html)

反汇编是个精细活，`PEDIY` 更是精细活中的精细活。这是我第一次 `PEDIY` 的最大感受。

第一次做 `PEDIY`，在找切入跳转点和堆栈平衡上，反反复复折腾了几十次，历经两个半晚上，总算是成功了，整理了一下，拿出来跟初学 `PEDIY` 的朋友们一起交流一下，大牛飘过吧。

## 基本情况

软件名称：手机控件电脑关机

软件功能：此程序通过邮件方式远程控件电脑关机（没亲自测试）

`PEDIY` 原因：此程序启动后每次都必须输入邮箱地址及邮箱密码，为了更加方便使用，所以打算以 `INI` 形式保存邮箱地址及密码，在程序启动时通过读取 `INI` 文件来自动加载邮箱地址及邮箱密码（朋友提出来的要求，呵，以前没玩过 `PEDIY`，所以就顺便练一下手）。

介绍完了，就正式开工吧…

## 1. 在程序目录里新建一个 `INI` 文件

内容如下：

```ini
[section]
user=yjphskf2004@163.com
pass=lzwx
```

## 2. 需要用到两个 `API` 函数

读取 `INI` 所需要的 `API`：`GetPrivateProfileStringA`

将读出值写入编辑框的 `API`：`SendMessageA`

`OD` 载入程序，右键-查找-查找当前模块中的名称，没有找到 `GetPrivateProfileStringA` 这个 `API` 函数，于是用 `PETOOLS` 添加此输入表 `API` 函数，保存。

## 3. 增加区段

经过几次的折腾发现原有的 `.text` 代码段空间不够，于是又用 `PETOOLS` 工具增加了一个区段，详细过程不嗦了。区段地址：`0058500`，大小 `3000`（随便设的），如图：

## 4. 在新的区段里写入需要用到的字符串

如图：

```text
地址         字符串
00585000     .\config.ini
00585030     section
00585060     user
00585090     pass
```

## 5. 找代码跳转点

这里折腾了我很久，做之前翻了一些以前 `PEDIY` 方面的文章，几乎都是在消息循环处找位置跳转处理的（因为都是加个按钮加个菜单什么的，在消息循环处处理确实比较合适），但是这个不一样，这个要求我们在程序启动，界面显示之前就得处理。找来找去，终于找了个个人觉得比较合适的地方，大家都知道窗口创建的过程是 `RegisterClassEx` -> `CreateWindow` -> `ShowWindow` -> `UpdateWindow`，经过比较，发现 `CreateWindow`，也就是编辑框控件创建完毕的时机跳转比较合适，看一下此函数的参数：

```c
HWND CreateWindow(
    LPCTSTR lpClassName,
    LPCTSTR lpWindowName,
    DWORD dwStyle,
    int x,
    int y,
    int nWidth,
    int nHeight,
    HWND hWndParent,
    HMENU hMenu,   // 控件 ID(窗口标识)
    HINSTANCE hInstance,
    LPVOID lpParam
);
```

其中 `hMenu` 为控件 ID，可以用来判断当前创建的是哪个窗口（控件），而 `CreateWindow` 的返回值就是当前创建的窗口的句柄，刚好满足了我们的要求，具体思路如下：

在 `CreateWindow` 创建窗口完毕的时机，跳到我们的代码处理处，判断当前窗口是否是邮箱名编辑框，是则读取 `INI` 文件中邮箱名的配置信息，并以消息的形式发送给邮箱名编辑框，不是则判断当前窗口是否是密码编辑框，是则读取 `INI` 文件中密码的配置信息，不是则跳回原来代码处继续往下执行。

于是，`OD` 载入程序，下 `CreateWindowExA` 断点，成功断下，堆栈窗口上右键-反汇编窗口跟随，到如下位置：

```asm
0046D24C  |. E8 F8FEFFFF    call sjkzdngj.0046D149
0046D251  |. FF75 D0        push [local.12]                ; /lParam
0046D254  |. FF75 D4        push [local.11]                ; |hInst
0046D257  |. FF75 D8        push [local.10]                ; |hMenu
0046D25A  |. FF75 DC        push [local.9]                 ; |hParent
0046D25D  |. FF75 E0        push [local.8]                 ; |Height
0046D260  |. FF75 E4        push [local.7]                 ; |Width
0046D263  |. FF75 E8        push [local.6]                 ; |Y
0046D266  |. FF75 EC        push [local.5]                 ; |X
0046D269  |. FF75 F0        push [local.4]                 ; |Style
0046D26C  |. FF75 F4        push [local.3]                 ; |WindowName
0046D26F  |. FF75 F8        push [local.2]                 ; |Class
0046D272  |. FF75 FC        push [local.1]                 ; |ExtStyle
0046D275  |. FF15 78B54700  call dword ptr ds:[<&USER32.CreateWindow>; \CreateWindowExA
0046D27B  |. 8BF8           mov edi,eax                    // 从这行开始改成跳向我们读取配置的代码，修改后的代码见下面
0046D27D  |. E8 13FFFFFF    call sjkzdngj.0046D195
0046D282  |. 85C0           test eax,eax
0046D284  |. 75 0A          jnz Xsjkzdngj.0046D290
0046D286  |. 8B06           mov eax,dword ptr ds:[esi]
0046D288  |. 8BCE           mov ecx,esi
```

## 6. 写入代码

在 `0046d27b` 处开始修改代码，跳转到我们的代码处理的位置。下面直接贴代码：

### 跳转指令

**修改前：**

```asm
0046D24C  |. E8 F8FEFFFF    call sjkzdngj.0046D149
0046D251  |. FF75 D0        push [local.12]                ; /lParam
0046D254  |. FF75 D4        push [local.11]                ; |hInst
0046D257  |. FF75 D8        push [local.10]                ; |hMenu
0046D25A  |. FF75 DC        push [local.9]                 ; |hParent
0046D25D  |. FF75 E0        push [local.8]                 ; |Height
0046D260  |. FF75 E4        push [local.7]                 ; |Width
0046D263  |. FF75 E8        push [local.6]                 ; |Y
0046D266  |. FF75 EC        push [local.5]                 ; |X
0046D269  |. FF75 F0        push [local.4]                 ; |Style
0046D26C  |. FF75 F4        push [local.3]                 ; |WindowName
0046D26F  |. FF75 F8        push [local.2]                 ; |Class
0046D272  |. FF75 FC        push [local.1]                 ; |ExtStyle
0046D275  |. FF15 78B54700  call dword ptr ds:[<&USER32.CreateWindow>; \CreateWindowExA
0046D27B  |. 8BF8           mov edi,eax                    // 从这行开始改成跳向我们读取配置的代码，修改后的代码见下面
0046D27D  |. E8 13FFFFFF    call sjkzdngj.0046D195
0046D282  |. 85C0           test eax,eax
0046D284  |. 75 0A          jnz Xsjkzdngj.0046D290
0046D286  |. 8B06           mov eax,dword ptr ds:[esi]
0046D288  |. 8BCE           mov ecx,esi
```

**修改后：**

```asm
0046D24C  . E8 F8FEFFFF    call 35.0046D149
0046D251  . FF75 D0        push dword ptr ss:[ebp-0x30]    ; /lParam
0046D254  . FF75 D4        push dword ptr ss:[ebp-0x2C]    ; |hInst
0046D257  . FF75 D8        push dword ptr ss:[ebp-0x28]    ; |hMenu
0046D25A  . FF75 DC        push dword ptr ss:[ebp-0x24]    ; |hParent
0046D25D  . FF75 E0        push dword ptr ss:[ebp-0x20]    ; |Height
0046D260  . FF75 E4        push dword ptr ss:[ebp-0x1C]    ; |Width
0046D263  . FF75 E8        push dword ptr ss:[ebp-0x18]    ; |Y
0046D266  . FF75 EC        push dword ptr ss:[ebp-0x14]    ; |X
0046D269  . FF75 F0        push dword ptr ss:[ebp-0x10]    ; |Style
0046D26C  . FF75 F4        push dword ptr ss:[ebp-0xC]     ; |WindowName
0046D26F  . FF75 F8        push dword ptr ss:[ebp-0x8]     ; |Class
0046D272  . FF75 FC        push dword ptr ss:[ebp-0x4]     ; |ExtStyle
0046D275  . FF15 78B54700  call dword ptr ds:[<&USER32.CreateWindow>; \CreateWindowExA
0046D27B  .- E9 807F1100   jmp 35.00585200                 // 这里跳向读取配置的代码(00585200)
0046D280     90            nop
0046D281     90            nop
0046D282     90            nop
0046D283     90            nop
0046D284  . 75 0A          jnz X35.0046D290                // 过了读取配置的代码后，跳回这里继续往下走
0046D286  . 8B06           mov eax,dword ptr ds:[esi]
0046D288  . 8BCE           mov ecx,esi
```

### 配置读取的代码

这一段是写在我们新加的区段里的：

```asm
00585200   60              pushad                            ; 入栈
00585201   83FB 6E         cmp ebx,0x6E                        ; 比较是否是用户名编辑框(6E 是控件 ID,用 spy++可以得到)
00585204   74 21           je X35.00585227                     ; 是则跳
00585206   83FB 78         cmp ebx,0x78                        ; 比较是否是密码编辑框(78 是控件 ID,用 spy++可以得到)
00585209   74 61           je X35.0058526C                     ; 是则跳
0058520B   90              nop
0058520C   90              nop
0058520D   90              nop
0058520E   90              nop
0058520F   90              nop
00585210   90              nop
00585211   90              nop
00585212   90              nop
00585213   61              popad                               ; 出栈
00585214   90              nop
00585215   8BF8            mov edi,eax                         ; 补上跳转指令覆盖的代码
00585217   E8 797FEEFF     call 35.0046D195                    ; 补上跳转指令覆盖的代码
0058521C   85C0            test eax,eax                         ; 补上跳转指令覆盖的代码
0058521E  - E9 6180EEFF    jmp 35.0046D284                     ; 跳回原来的地方继续往下执行
00585223   90              nop
00585224   90              nop
00585225   90              nop
```



```
00585226    90              nop
00585227    A3 00515800     mov dword ptr ds:[0x585100], eax          ; 句柄保存到 585100 处
0058522C    68 00505800     push 35.00585000                          ; ASCII ".\config.ini"
00585231    6A 64           push 0x64
00585233    68 C0505800     push 35.005850C0
00585238    68 C0505800     push 35.005850C0
0058523D    68 60505800     push 35.00585060                          ; ASCII "user"
00585242    68 30505800     push 35.00585030                          ; ASCII "section"
00585247    FF15 31815800   call dword ptr ds:[<&kernel32.GetPrivate>]; 读取用户名配置
0058524D    68 C0505800     push 35.005850C0
00585252    6A 00           push 0x0
00585254    6A 0C           push 0xC
00585256    FF35 00515800   push dword ptr ds:[0x585100]
0058525C    FF15 DCB54700   call dword ptr ds:[<&USER32.SendMessageA>]; 把读出来的值以消息方式发送到用户名编辑框
00585262  ^ EB AF          jmp X35.00585213                           ; 跳
00585264    0000           add byte ptr ds:[eax],al
00585266    0000           add byte ptr ds:[eax],al
00585268    0000           add byte ptr ds:[eax],al
0058526A    0000           add byte ptr ds:[eax],al
0058526C    A3 00515800     mov dword ptr ds:[0x585100],eax           ; 句柄保存到 585100 处
00585271    68 00505800     push 35.00585000                          ; ASCII ".\config.ini"
00585276    6A 64           push 0x64
00585278    68 C0505800     push 35.005850C0
0058527D    68 C0505800     push 35.005850C0
00585282    68 90505800     push 35.00585090                          ; ASCII "pass"
00585287    68 30505800     push 35.00585030                          ; ASCII "section"
0058528C    FF15 31815800   call dword ptr ds:[<&kernel32.GetPrivate>]; 读取密码配置
00585292    68 C0505800     push 35.005850C0
00585297    6A 00           push 0x0
00585299    6A 0C           push 0xC
0058529B    FF35 00515800   push dword ptr ds:[0x585100]
005852A1    FF15 DCB54700   call dword ptr ds:[<&USER32.SendMessageA>]; 把读出来的值以消息方式发送到密码编辑框
005852A7  ^ E9 67FFFFFF     jmp 35.00585213                           ; 跳
```

`OK`，到目前为止。所有的工作都已经完成，`OD` 保存好文件后，测试，一切正常。

关键的只有两点，一是要找准代码跳转点，二是写代码的时候要注意堆栈平衡。思路最重要。

附件因为比较大。就不上传了。

因工作原因，少有时间来看雪逛了，今天难得，趁着朋友给的这程序，发个文章上来跟初学的朋友交流一下。水平有限，有啥不当之处，还请各位指正。

`PS`：现在论坛怎么不能直接贴图上来了呢。印象中以前是可以滴...
