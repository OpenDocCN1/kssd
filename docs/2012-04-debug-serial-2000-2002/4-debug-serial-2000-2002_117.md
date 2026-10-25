
# Unfoxall 2.0

> 原文：[`www.pediy.com/kssd/pediy01/forum058.htm`](https://www.pediy.com/kssd/pediy01/forum058.htm)

《Unfoxall 2.0 增强版完美破解方法》

首先非常感谢 TCL 朋友，如果不是他把 Unfoxall 2.0 增强版(未注册版本)送给我，现在我也不会把这篇文章贴出来！

Unfoxall 的确是一个非常优秀的 foxpro 反编译软件，它取用一机一注册码的方法来对用户进行限制，真讨厌！

**破解工具：**
`SoftIce for win95/98`

**破解方法：**

1. 运行 `unfoxall`
2. 用鼠标单击 `unfoxall` 菜单->“帮助”, 然后再单击“关于 unfoxAll”
3. 待弹出对话框后请再单击“注册”按钮

   录入如下资料：

   用户姓名：`000ye`
   
   用户单位：`china`

4. 单击“生成待注册文件”按钮
5. 这时系统会在本目录下生成一个名为 `unfoxAll.ini` 的文件
6. `ctrl+D`, 激活 `softIce`
7. `bpx hmemcpy`
8. 按 `F5`
9. 用鼠标单击“注册 unfoxAll”按钮，这时程序被拦
    1. `BD \*`, 按 `F12`, 直到进入 `unfoxAll` 的领空(不知按多少下)
    2. `bpx 0048FC64` (这个步骤非常重要)
    3. 按 `F5`，然后再根据实际情况操作，这时系统会出现两个提示框，当你按“确认”后，系统会自动退出(这是好现象)
    4. 重新启动 `unfoxAll`，这时程序会在 `xxxx:0048FC64` 处被拦
       ```
       xxxx:0048FC5F  MOV EAX, [006E0CFC]    ; d eax 这是真正的注册码
       xxxx:0048FC64  MOV EDX, [EBP-10]      ; d edx 这是你所谓的注册码
       xxxx:0048FC67  CALL 00403F18          ; 注册码比较的 call
       xxxx:0048FC6C  JNZ 0048FCA2           ; 不相等则跳
       ```
    5. `d eax`, 还不赶快把它抄下来？我这里看到的是 “`5304C7A1`”

**得到注册码后如何注册？**

1. 首先打开 `unfoxAll` 目录下的 `unfoxAll.ini` 文件
2. 这时你会看到如下东东：
   ```ini
   [UserInfo]
   UserName=000ye
   UserCompany=china
   UserKey=E90666CF    (这个东东因机器不同而不同)
   SerialNumber=[Not Registered]
   ```
3. 把其改为这样：
   ```ini
   [UserInfo]
   UserName=000ye
   UserCompany=china
   UserKey=E90666CF   
   SerialNumber=5304C7A1
   ```
   改完后请保存退出
4. 用鼠标单击 `unfoxall` 菜单->“帮助”, 然后再单击“关于 unfoxAll”
5. 待弹出对话框后请再单击“注册”按钮
6. 用鼠标单击“注册 unfoxAll”按钮
7. 选择 `unfoxall.ini` 文件，然后再按“打开”按钮
8. 最后根据实际情况操作，就这样，`unfoxAll` 已经被成功注册了，高兴吗？

<div align="right">
<br>
000ye [CCG]<br>
2000.05.17
</div>
