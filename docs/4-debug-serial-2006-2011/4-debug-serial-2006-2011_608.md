
# 第一次 crack

> 原文：[`www.pediy.com/kssd/pediy12/91768.html`](https://www.pediy.com/kssd/pediy12/91768.html)

工具: `W32dsm8.93+`, `UltraEdit-32` (或者 `hiew`)

对象: 小榕的黑客字典 II(该黑客软件部分高级功能须注册才能使用)

## 1. 尝试注册

运行此软件,点注册,输入你的注册名和任意注册码,按注册

出现错误提示信息"注册名或注册码不正确!",将错误提示信息记下来

## 2. 备份

将 `UltraDict.exe` 复制两份,分别命名为 `UltraDict.exe`(作为备份)和 `U.exe`

## 3. 反汇编

运行 `W32dsm8.93+` 将 `U.exe` 反汇编

## 4. 查找

选择串式参考(String Data References),在窗口中找刚才的错误提示信息或可能是注册正确的提示信息:

```
"必须指定一个文件名!"
"不大写"
"错误!在高级选项中没有选中任何一项."
"大写"
"当前磁盘空间剩余:%ld KBytes"
...
"非常感谢您的支持,注册成功!"         ;注意
"黑客字典 II(限制版) (PID:9948)"
"黑客字典 II(注册给:%s) (PID:9948)"
...
"未指定数字范围,默认由'0'到'9'."
"未指定字母范围,默认由'a'到'z'."
"文件 C:\\CustBanyet.Def 没有找到."
"文件创建失败!"
"无辅音字母"
"E\_^d?"
"$"
"用 LF 和 CR 间隔"
"有辅音字母"
"韫?3 烂$′tC"
"注册名或注册码不正确!"         ;注意
"字典成功生成!"
"字典设置"
"字母在前面"
...
```

注意到一句:"非常感谢你的支持,注册成功!",怀疑可能是正确的提示信息

在该处双击鼠标左键,反复双击,发现只有一处调用,

回到主窗口停在下面的(#########所标的地方)

## 5. 分析

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401CF7(U)
|
:00401CFE 3BC3                 cmp eax, ebx
:00401D00 0F85DF000000         jne 00401DE5         ;关键跳转
:00401D06 6801100000           push 00001001
;中间省略------------------------------------------------------

* Possible StringData Ref from Data Obj ->"C:\\regbanyet.dat"         ;@@@@@@@@@
                              |
:00401DA5 68F4704300           push 004370F4

* Reference To: KERNEL32.SetFileAttributesA, Ord:0268h
                              |
:00401DAA FF158CB24200         Call dword ptr [0042B28C]

* Possible StringData Ref from Data Obj ->"非常感谢您的支持,注册成功!"         ;#########注册成功提示
                              |
:00401DB0 68D8704300           push 004370D8
:00401DB5 8D8C24EC000000       lea ecx, dword ptr [esp+000000EC]
:00401DBC E8AF010000           call 00401F70
:00401DC1 8D4C2410             lea ecx, dword ptr [esp+10]
:00401DC5 889C24B4010000       mov byte ptr [esp+000001B4], bl
:00401DCC E877EF0100           call 00420D48
:00401DD1 8BCD                 mov ecx, ebp
:00401DD3 E844B50100           call 0041D31C
:00401DD8 C78424B401000003000000 mov dword ptr [esp+000001B4], 00000003
:00401DE3 EB1C                 jmp 00401E01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401D00(C)
|

* Possible StringData Ref from Data Obj ->"注册名或注册码不正确!"         ;********注册失败提示
                              |
:00401DE5 68C0704300           push 004370C0
:00401DEA 8D8C24EC000000       lea ecx, dword ptr [esp+000000EC]
:00401DF1 E87A010000           call 00401F70
:00401DF6 C78424B401000002000000 mov dword ptr [esp+000001B4], 00000002
```

从 `#########` 所标的地方开始向上找,一直找到有如下命令时为止:`cmp`, `jne`, `je` 等等,找到

```
:00401D00 0F85DF000000         jne 00401DE5
```

程序之所以没有来到注册成功的地方主要就是这个跳转,

跳到了错误的地方(`********`所标的地方)

`00401D00` 处的跳转称为关键跳转

## 6. Crack

把绿色光条停在关键跳转,即

```
:00401D00 0F85DF000000         jne 00401DE5
```

在主窗口的状态栏找到关键跳转的偏移地址(实际修改地址)

```
@:00401D00 @Offset 00001D00h (h 表示十六进制)
```

实际修改地址是 `01D00`

用 `UltraEdit-32` 打开 `UltraDict.exe`, `Ctrl-G`, 键入 `0x01D00` (`0x` 表示十六进制)

查找到 `00001d00h`, 把 `85DF` 改为 `84DF`, 保存

再次运行 `UltraDict.exe` 输入你的注册名和任意注册码注册,

提示"非常感谢你的支持,注册成功!"

大功告成, 哈哈

总结: 此方法属于暴力爆破工程!

## 7. 原理

修改前: 输入的注册码不对时跳到出错的地方, 正确时来到对的地方

修改后: 输入的注册码不对时来到对的地方, 正确时跳到出错的地方

## 8. 后记

注册后发现黑客字典 II 没有在她的主文件夹生成任何保存注册信息的文件

开始怀疑可能是把注册信息保存在注册表

用 `Vmware` 虚拟的系统再次破解注册,

同时用软件扫描注册前后注册表的变化

也没有发现有保存注册信息的相关记录

难道是保存在 `UltraDict.exe` 里面, 这是不可能的!

因为重新解压出来的 `UltraDict.exe` 也是显示注册的!

后来无意打开 C 盘根目录, 发现多了一个隐藏的 `regbanyet.dat` 文件

用记事本打开一看, 哈哈, 里面就是刚刚注册的注册名和注册码

删除这个文件, 运行 `UltraDict.exe`, 果然显示没有注册

好奇心驱使再次运行 `W32dsm8.93+` 将 `UltraDict.exe` 反汇编

仔细查看发现(`@@@@@@@@@`所标的地方):

```
* Possible StringData Ref from Data Obj ->"C:\\regbanyet.dat"
```

正是提示注册成功后在 C 盘根目录创建了 `regbanyet.dat` 文件

手动在 C 盘根目录创建 `regbanyet.dat`, 不输入任何信息

再次运行 `UltraDict.exe`, 显示已经注册(原来注册名和注册码可以为空字符)

哈哈, 这也算是第二种 `Crack` 方法了

总结: `Crack` 的确需要运气+细心

; That's all.

### 上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || 黑客字典 II(中文版 200K）NameBanyet CodeO8G22M.zip

 |
