

# 加入 [BCG] 的第一篇破解文章：ZTZ-IE 网络浏览器 1.5 破解

加入 [BCG] 的第一篇破解文章，也是本人第一次写破解文章！（4 千字）

## 软件信息

- 软件：ZTZ-IE 网络浏览器 1.5
- 功能：一个多窗口浏览器，类似 MyIE，比 MyIE 的浏览速度快，但功能不如 MyIE 的多，有自己的特殊功能
- 软件下载地址：`http://sd.onlinedown.net/down/ZTZ-IE.zip`
- 破解使用工具：`W32DASM` 893 黄金中文版、`FI2.45`、`hiew640`

## 1. 使用 `FI` 检测是否加壳

使用 `FI` 检测此软件是否加壳。一看，此软件未加壳，是用 DELPHI 写的！可以直接用 `w32dasm` 来反汇编。

## 2. 启动软件并观察错误提示

启动软件，点击“帮助”→“软件注册”，在用户名和密码的框中随便输入几个数字（但一定要大于等于 4 位数字，否则会出错的）。单击“确定”后，出现一个“警告”提示：“密码不正确”。我们可以按“确定”，然后退出此浏览器。

## 3. 使用 `W32DASM` 反汇编并查找字符串

打开 `W32DASM`，加载 ZTZ-IE 浏览器的文件 `ZTZ-IE.exe`。之后，按下“参考”→“串式参考”，然后查找上面看到的出错提示框。我们找到“密码不正确”这一行并双击它，看到以下代码：

```
* Possible StringData Ref from Code Obj ->"警告"
            |
:0046B69A B94CB74600              mov ecx, 0046B74C
* Possible StringData Ref from Code Obj ->"密码不正确！"
            |
:0046B69F BA54B74600              mov edx, 0046B754
:0046B6A4 A15C514700              mov eax, dword ptr [0047515C]
:0046B6A9 8B00                    mov eax, dword ptr [eax]
:0046B6AB E8E40BFEFF              call 0044C294
:0046B6B0 A11C694700              mov eax, dword ptr [0047691C]
:0046B6B5 E8F6D7FDFF              call 00448EB0
```

我们往上看，经过比较发现：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046B5DB(C)
|
:0046B616 8D55F8                    lea edx, dword ptr [ebp-08]
:0046B619 8B86EC020000              mov eax, dword ptr [esi+000002EC]
:0046B61F E88C25FCFF                call 0042DBB0
:0046B624 8B45F8                    mov eax, dword ptr [ebp-08]
:0046B627 50                        push eax
:0046B628 8D55F0                    lea edx, dword ptr [ebp-10]
:0046B62B 8B86E0020000              mov eax, dword ptr [esi+000002E0]
:0046B631 E87A25FCFF                call 0042DBB0
:0046B636 8B45F0                    mov eax, dword ptr [ebp-10]
:0046B639 8D55F4                    lea edx, dword ptr [ebp-0C]
:0046B63C E863FDFFFF                call 0046B3A4
:0046B641 8B55F4                    mov edx, dword ptr [ebp-0C]
:0046B644 58                        pop eax
:0046B645 E84288F9FF                call 00403E8C-->
```

这里 `CALL` 进去，比较注册码。在这用 `trw200` 下断点，敲 F8 进入，得到：

用户名：`yangjunnan`　密码：`vzztwruuyq`

关键跳转在这里：

```
:0046B64A 7533                    jne 0046B67F
```

`jne 0046B67F` 这里就是要改的地方：不相等则跳到错误提示，直接 `nop` 掉它即可。后面的相关代码如下：

```
:0046B64C BA0B000000              mov edx, 0000000B
* Possible StringData Ref from Code Obj ->"Yire"
            |
:0046B651 B824B74600              mov eax, 0046B724
:0046B656 E839250000              call 0046DB94
:0046B65B 6A40                    push 00000040
* Possible StringData Ref from Code Obj ->"祝贺你"
            |
:0046B65D B92CB74600              mov ecx, 0046B72C
* Possible StringData Ref from Code Obj ->"密码正确，注册成功！"
            |
:0046B662 BA34B74600              mov edx, 0046B734
:0046B667 A15C514700              mov eax, dword ptr [0047515C]
:0046B66C 8B00                    mov eax, dword ptr [eax]
:0046B66E E8210CFEFF              call 0044C294
:0046B673 A11C694700              mov eax, dword ptr [0047691C]
:0046B678 E833D8FDFF              call 00448EB0
:0046B67D EB3B                    jmp 0046B6BA
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046B64A(C)
|
:0046B67F A180524700              mov eax, dword ptr [00475280]
:0046B684 83381E                  cmp dword ptr [eax], 0000001E
:0046B687 7C0F                    jl 0046B698
:0046B689 BA01000000              mov edx, 00000001
* Possible StringData Ref from Code Obj ->"Yire"
            |
:0046B68E B824B74600              mov eax, 0046B724
:0046B693 E8FC240000              call 0046DB94
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046B687(C)
|
:0046B698 6A40                    push 00000040
* Possible StringData Ref from Code Obj ->"警告"
            |
:0046B69A B94CB74600              mov ecx, 0046B74C
* Possible StringData Ref from Code Obj ->"密码不正确！"
            |
:0046B69F BA54B74600              mov edx, 0046B754
:0046B6A4 A15C514700              mov eax, dword ptr [0047515C]
:0046B6A9 8B00                    mov eax, dword ptr [eax]
:0046B6AB E8E40BFEFF              call 0044C294
:0046B6B0 A11C694700              mov eax, dword ptr [0047691C]
:0046B6B5 E8F6D7FDFF              call 00448EB0
```

## 总结

只要用 `hiew640`，修改文件：`0046B64A 7533 -> 9090`，即可破解成功！

用户名：`yangjunnan`　密码：`vzztwruuyq`

破解者：yangjunnan　Email：nccqy@163.com　OICQ：3597625

