
# [原创]1-Click PC Tuneup软件-破解实录-[上][中][下]-(图文+算法+注册机源码)

标 题: 【原创】1-Click PC Tuneup软件-破解实录-[上][中][下]-图文并茂

作 者: 猪头三

网 站: [b8eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4R3^5y4X3q4K6L8g2)9J5k6h3y4G2L8b7`.`.](https://bbs.kanxue.com/elink@5beK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4R3^5y4X3q4K6L8g2)9J5k6h3y4G2L8b7`.`.)

时 间: 2013-10-15 13:53:35

链 接: http://bbs.pediy.com/showthread.php?t=180072

## [序言]

脱离玩破解已经好几年了, 由于前段时间在休息, 闲来玩玩, 随便找了一款国外软件1-click pc tuneup软件下手. 本人玩破解的原则是: 随便玩玩, 不跟经济利益挂钩, 选择无壳软件. 破解的目的是复习和巩固逆向思维, 而不是耗费时间和生命去研究各种刁蛮的加壳, 用自动脱壳工具就Ok了.

## [本文介绍]

本文章一共分为10部分, 完全图文并茂, 分享在破解过程中的思维想法, 阅读本文需要一定的汇编语言, C/C++语言, 程序运行原理, 有开发软件实际经验, 熟悉Windows操作系统理论, 有基本的算法理论基础, 技术难度中等偏上.

1> 初步了解破解环境的部署

2> 初步了解1-click pc tuneup软件的主程序基本特征

3> 利用ollydbg定位你的处理注册号的函数

4> 多次陷入困境

5> 通过Qt类库的QString类相关函数找到处理注册号的函数

6> 开始分析处理注册号的函数

7> 不要满足直接获取注册号码

8> 分析注册号的输入规则

9> 分析注册号码的生成算法

10> 为1-click pc tuneup软件写注册机

## [请做好基础知识储备]

上面已经提到需要相关的基础知识, 如果您在阅读文章之前基础不扎实, 有可能你不会明白文章所要表达的含义. 破解领域太广太深, 接触和学习破解之前, 最好都需要会编程, 这是必须要具备的技能.

## >>>>>>>>> 第一部分: 初步了解破解环境的部署 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

要破解一款软件(主要目的是获取该款软件的注册号码), 不光需要扎实的汇编语言, C/C++语言, 程序运行原理, 熟悉Windows操作系统理论, 最好能具备有一定的软件开发经验, 以及基本的算法理论基础. 假设你这些技能都具备了, 那么我们可以选择3个工具来进行破解某款软件:

**1. ollydbg : 一款Windows操作系统平台上, 针对32位汇编的分析调试工具. 在没有源码可参考的情况下,用来分析二进制代码有着非常强大的优势和好处. 该款软件属于动态分析范畴**

![](https://bbs.kanxue.com/upload/attach/201310/20378_40a9ff61f669dd45.png)

**2. ida pro : 一款支持Windows、Mac Os X 和 Linux的多平台反编译和调试工具. 该款软件属于静态分析工具, 但随着功能的不断强大, 已经支持动态分析.**

![](https://bbs.kanxue.com/upload/attach/201310/20378_76533a49d2cfce8d.png)

**3. Detect it Easy : 一款专门分析程序文件格式的软件. 当然还有其他相关的代替软件比如: PeID**

![](https://bbs.kanxue.com/upload/attach/201310/20378_a78a30c02bb6e2d7.png)

其实破解一款软件需要的工具不止上面提到的3款, 这里仅仅只介绍了重要的也是不可缺少的. 如果你需要破解一个复杂度较高的软件, 那么应该根据实际情况考虑使用多种辅助破解工具来配合. 但这里就不做详细介绍了, 因为不是本文的核心.

## >>>>>>>>> 第二部分: 初步了解1-click pc tuneup软件的主程序基本特征 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

今日我们随便选择了一款国外软件名为1-click pc tuneup, 这是一款Windows操作系统维护工具, 虽然破解的目标软件已经选择好了, 但这款软件是否符合我的能力范围来进行破解呢？大家都知道, 矛与盾都是对立发展的, 我作为破解方那就是"矛", 1-click pc tuneup这款软件就是被破解方也就是“盾”。如果这个“盾”过于强大, 而我的"矛"过去脆弱, 那就是我的破解能力还达不到击破这个软件的水平. 因此要破解一款软件, 首先要衡量自己是实力, 你当前的实力能破解到哪种类型的软件. 因此根据这个思考原则, 我目前的实力也就只能破解那些采用Delphi VC++工具开发的软件, 并且还没有进行二次解密过的软件, 也就说我们通常说的"加壳".

![](https://bbs.kanxue.com/upload/attach/201310/20378_2dae0804cef01dd6.png)

下面我们就通过Detect it Easy这款程序来了解1-click pc tuneup这款软件的基本特征, 也就是需要了解这个软件是用什么工具开发的, 并且到底这个程序是否被加过密(加壳).

**1. 启动 Detect it Easy -> 单击右下角"Open PE"按钮 -> 选择1-click pc tuneup软件的主程序"`PCtuneup.exe`"**

这时候你会发现Detect it Easy已经侦测并分析出1-click pc tuneup这款是用什么开发工具写的, 以及是否被加过密(加壳), 很幸运的是1-click pc tuneup这款软件没有加过密(加壳)而且还是VC++开发的, 这完全符合我的破解能力范围.

![](https://bbs.kanxue.com/upload/attach/201310/20378_aa6da744135c4317.png)

## >>>>>>>>> 第三部分: 利用ollydbg定位处理注册号的函数 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

一般来说破解目标软件并获取正确的注册号码, 我们第一步会尝试先输入一个无效的测试注册号并观察目标软件会什么反应, 比如: 错误提示信息, 是否弹出错误提示对话框. 那么我们就会通过这2个重要的信息定位到目标软件处理注册号的函数, 一旦定位成功, 那么就表示破解已经迈向成功的第一步了.

**1. 下面我们开始启动`ollydbg`, 选择"`File`"->"`Open`", 弹出文件选择对话框之后, 找到1-click pc tuneup软件的主程序"`PCtuneup.exe`"并按下"打开按钮".**

![](https://bbs.kanxue.com/upload/attach/201310/20378_745e9646792f561b.png)

**2. 然后在`ollydbg`的菜单上，选择"`Debug`"->"`Run`", 这时我们可以看到1-click pc tuneup软件已经运行起来了, 这就意味着`ollydbg`已经在调试1-click pc tuneup软件, 里面的汇编代码就是1-click pc tuneup软件由C/C++语言开发并被反编译过后的汇编代码.**

![](https://bbs.kanxue.com/upload/attach/201310/20378_9ba7acea28f4d241.png)

**3. 单击1-click pc tuneup软件界面右上角的下拉按钮, 选择"`Activate Procude`",这时就弹出注册号输入对话框. 此时我们尝试输入一个错误的测试注册号, 比如: `123456789` 然后按下"`Activate Now`"之后会提示一个错误信息"`Invaild registion code! Please make sure your regisratin code matches the current version. `"**

![](https://bbs.kanxue.com/upload/attach/201310/20378_809b459f3ed52cb4.png)

**4. 好了, 现在我们有了这个长长错误信息提示字符串, 有什么作用呢? 其实原理很简单, 如果我们有软件开发经验的话, 我们脑海会马上浮现出这样一个验证注册号码的代码逻辑:**

```
char *pchar_input_key = null ;
pchar_input_key = get_input_key() // 获取用户输入注册号码
bool bool_IsValid = true ;
bool_IsValid = check_input_key(pchar_input_key) ; // 校验注册号码的合法性
if (bool_IsError == false)
{
   // 注册失败
   ShowMsg("Invaild registion code!") // 注册号错误
}
else
{
   // 注册成功
}
```

通过上面的代码逻辑, 我们很清楚的知道只要能找到"`Invaild registion code!`"这个字符串的显示位置, 那么我们就很快在当前的位置附近发现校验注册号正确性的函数, 比如`check_input_key()`这个函数. 有了这个思路之后, 我们如何通过`ollydbg`定位到"`Invaild registion code! Please make sure your regisratin code matches the current version. `"这个错误信息字符串呢?

**5. 定位错误信息字符串: 用鼠标右键单击`ollydbg`界面上的汇编代码区域, 然后选择”`Ultra String Reference`“->"`1 Find ASCII`" 或者 ”`2 Find UNICODE`“都可以, 这时会弹出"`Ultra String Reference Plugin`"窗口并显示1-click pc tuneup软件在运行过程中所需要到的字符串.**

![](https://bbs.kanxue.com/upload/attach/201310/20378_891b57cf848e057e.png)

![](https://bbs.kanxue.com/upload/attach/201310/20378_4e8fd304ac1fc5db.png)

可是很遗憾的事情发生了, 我们并未找到"`Invaild registion code! Please make sure your regisratin code matches the current version. `"这个错误信息提示字符串, 估计软件的作者针对这个错误信息作了加密处理让我们不能这么简单的找到这个字符串, 也就意味着我们很难通过这样的方式来定位到处理注册号的函数.

## >>>>>>>>> 第四部分: 多次陷入困境 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

既然错误信息提示字符串定位的方式行不通, 那还有什么其他方法可以定位到处理注册号的函数吗？ 当然有, 只要我们有基本的软件开发经验, 那么还是有办法的, 比如通过找C/C++标准的字符串处理函数来定位到处理注册号的函数. 因此我们只需要思考假设要写一个验证注册号码的函数`check_input_key()`的时候可能会需要到什么字符串处理呢？肯定会需要 `stricmp()` `strcat()` `strstr()` 这类字符串, 比如假设下面的伪代码

```
bool check_input_key(char *pchar_param_Input)
{
   if(stricmp("123456789", "xxxxxxxxxxxx") <> 0)
   {
       // 错误的注册号
      return false;
    }
   return ture ;
}
```

通过上面的伪代码可知, `stricmp()`是用来跟正确的注册号比较, 如果不等于0, 那么就说明”`123456789`“不是正确的注册号. 同理根据这样的思路, 只要我们通过`ollydbg`找到这个`stricmp()`所在的位置, 那么就相当于找到处理注册号的函数, 而且还是处理注册号的函数的内部, 这样更加可以直接看到校验注册号的具体代码, 比利用错误信息字符串提示的方式来定位, 更加精确.

**1. 定位类似`stricmp()`函数: 用鼠标右键单击`ollydbg`界面上的汇编代码区域, 然后选择”`Search for`“->"`All intermodular calls`", 这时会弹出"`Found intermodular calls`"窗口并显示1-click pc tuneup软件在运行过程中所需要到的所有函数.**

![](https://bbs.kanxue.com/upload/attach/201310/20378_241f8787bc6fb389.png)

![](https://bbs.kanxue.com/upload/attach/201310/20378_1906ee8c823bfaa2.png)

**2. 赶紧查看是否有`stricmp()` `strcat()` `strstr()`有没有这类字符串处理函数, 但完全傻眼了, 完全没有调用到这类字符串处理函数, 反而全部大量充斥着`QtCrot4.??xxxxxxxxxxx`、`QtGui.??xxxxxxxxxxxx`等等这些带有Qt标示符的函数, 我们又一次陷入的僵局.**

## >>>>>>>>> 第五部分: 通过Qt类库的QString类相关函数找到处理注册号的函数 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

大家如果看到这里, 可能有点迷茫了, `QtCrot4.??xxxxxxxxxxx`、`QtGui.??xxxxxxxxxxxx` 这些到底是什么函数呢？其实我们可以通过google搜索就知道这些就是Qt开发类库, 是用C++编写的, 由此可以证明1-click pc tuneup这款软件是用Qt类库开发的. 那么既然是用Qt库开发的, 此时用于处理字符串的Qt类库到底是什么呢？是不是我们可以认为只要知道Qt类库的字符串处理函数, 就可以定位到处理注册号的函数的内部呢？如同>>>第四部分: 多次陷入困境<<<描述哪样. 经过google搜索, 我们知道Qt类库处理字符串是由`QString`类来处理的, 因此我们只需要通过`ollydbg`对1-click pc tuneup软件内部所有用到`Qstring`类的代码都下个"断点", 那么就有可能会定位到处理注册号的函数, 而且还是处理注册号的函数的内部.

**1. 定位`Qstring`类的所有相关函数: 用鼠标右键单击`ollydbg`界面上的汇编代码区域, 然后选择”`Search for`“->"`All intermodular calls`", 这时会弹出"`Found intermodular calls`"窗口并显示1-click pc tuneup软件在运行过程中所需要到的所有函数.**

**2. 在"`Found intermodular calls`"窗口单击有包含有"`QtCore4.??0QString@@QAE@ABV0@@Z`"字符串的其中一行, 然后右键选择: “`Set breakpoint on every call to ??QString@@QAE@ABV0@@Z`”**

![](https://bbs.kanxue.com/upload/attach/201310/20378_ed0f60ebd4922d00.png)

**3. 按照第2步的方式, 也对包含有"`QtCore4.??0QString@@QAE@PBD@Z`"字符串或者包含有“`QtCore4.??0QString@@QAE@XZ`”进行处理.**

![](https://bbs.kanxue.com/upload/attach/201310/20378_e54c5d12ad33a347.png)

**4. 就这样通过上面3步介绍, 已经对`QString`类的所有函数进行了"断点"设置, 然后返回1-click pc tuneup软件的注册对话框, 再次按下"`Activate Now`", 此时`ollydbg`就会停顿(断下)在第一个调用`QString`类相关字符串处理函数的代码行. 那么我们如何确定断下来的代码行, 就是处在处理注册号的函数的附近呢？这就要观察`ESP`和`EBP`这2个堆栈显示的信息是否包含有你的测试注册号"`123456789`"了, 如果有, 就表明我们已经处在处理注册号的函数的附近, 如果没有, 就在继续按下`F9`之后, 再一次会调用`QString`类相关字符串处理函数的代码行, 这时你再一次观察`ESP`和`EBP`这2个堆栈显示的信息是否包含有你的测试注册号"`123456789`"，依次反复进行, 直至看到测试注册号"`123456789`"为止. (重要: `ollydbg`每停顿(断下)一个调用`QString`类相关字符串处理函数时，你就必须观察`ESP`和`EBP`这2个堆栈框口, 在观察的时候小范围的用鼠标中间滚动, 上下查看数据)**

![](https://bbs.kanxue.com/upload/attach/201310/20378_805df64ab63c7a40.png)

通过上图的显示, 我们就可以知道

```
000D48DB   .  FF15 F8F20E00 CALL DWORD PTR [<&QtCore4.??0QString@@QA>;  QtCore4.??0QString@@QAE@PBD@Z
```

这行代码就是处在处理注册号的函数的附近.

**5. 找到这行代码之后, 我们就顺着这行代码往上一行一行看, 直至看到:**
