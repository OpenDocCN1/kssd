
# 极速××××1.94

> 原文：[`www.pediy.com/kssd/pediy05/pediy50241.htm`](https://www.pediy.com/kssd/pediy05/pediy50241.htm)

## Java 程序的破解方法

**软件名称**：极速××××1.94（国产软件）

**未注册版限制**：
1. 无法使用软件的向导功能；
2. 不能自定义超时时间。

**破解工具**：`J++Extract` 1.1、`Jad` 1.5.7g。

**破解人**：飞鹰[BCG]

**E-mail**：`flithawk@163.com`

**网址**：`http://flithawk.longcity.net`

### 破解步骤

我一开始并不知道该软件是用 Java 语言编写，因为它是一个 `.exe` 可执行程序，当我用 `Trw` 2000 跟踪时才发现的，再加上该软件容量很小（只有 58KB），所以我才断定它是用 Java 编写的解释执行程序。对待这种解释执行的程序，最有力的工具就是相应的反编译器。

`J++Extract` 工具可以把用 Java 编写的 `EXE` 文件解包成相应的 class 文件（`*.class`）、资源文件（`*.gif`, `*.bmp`, `*.resources` 和其它）和相应的应用程序图标。把该软件解包后生成了三个 class 文件：`About.class`、`Net007Net.class`、`reg.class` 和一个批处理文件：`peedip.bat`，还有其它的一些文件这里就不提了，与研究软件的注册算法无关。运行 `peedip.bat` 批处理文件后出现了软件的主界面，所以这个批处理文件应该是整个软件解包后的主引导程序。从上面解包出来的 class 文件的文件名中我初步判断 `reg.class` 文件应该就是软件的“注册”窗体，其中必含有软件的注册算法；`About.class` 文件应该是软件的“关于”窗体；那么 `Net007Net.class` 文件就是软件的“主”窗体了。

`Jad` 工具可以把 CLASS 文件反编译成几乎和源码一模一样的代码。所以，我用 `Jad` 分别反编译了上面解包出来的三个 `.class` 文件。经过分析后，肯定了我一开始的猜测是正确的，`reg.class` 文件中果然存放着软件的注册算法，反编译后该文件中的代码如下所示：其中，“//”处是我自己加入的注释。

```java
import com.ms.wfc.app.Application;

import com.ms.wfc.core.*;

import com.ms.wfc.ui.*;

import java.io.*;

public class reg extends Form

{

    public reg()

    {

        components = new Container();

        userName = new Edit();

        regB = new Button();

        userNo = new Edit();

        label1 = new Label();

        label2 = new Label();

        label3 = new Label();

        label4 = new Label();

        label5 = new Label();

        button1 = new Button();

        initForm();

    }

    private void initForm() //该过程应该是初始化窗体

    {

        IResourceManager resources = new ResourceManager(this, "reg");

        setText("\u6CE8\u518C");

        setAutoScaleBaseSize(new Point(6, 12));

        setBorderStyle(1);

        setClientSize(new Point(309, 183));

        setIcon((Icon)resources.getObject("this_icon"));

        setMaximizeBox(false);

        setMinimizeBox(false);

        setStartPosition(1);

        userName.setLocation(new Point(152, 104));

        userName.setSize(new Point(144, 19));

        userName.setTabIndex(0);

        userName.setText("");

        regB.setLocation(new Point(8, 152));

        regB.setSize(new Point(72, 24));

        regB.setTabIndex(1);

        regB.setText("\u73B0\u5728\u6CE8\u518C");

        regB.addOnClick(new EventHandler(this, "regB_click"));

        userNo.setLocation(new Point(152, 128));

        userNo.setSize(new Point(144, 19));

        userNo.setTabIndex(2);

        userNo.setText("");

        label1.setForeColor(Color.HIGHLIGHT);

        label1.setLocation(new Point(8, 104));

        label1.setSize(new Point(144, 16));

        label1.setTabIndex(3);

        label1.setTabStop(false);

        label1.setText("\u8BF7\u8F93\u5165\u60A8\u7533\u8BF7\u7684\u7528\u6237\u540D :");

        label2.setForeColor(Color.HIGHLIGHT);

        label2.setLocation(new Point(8, 128));

        label2.setSize(new Point(144, 16));

        label2.setTabIndex(4);

        label2.setTabStop(false);

        label2.setText("\u8BF7\u8F93\u5165\u60A8\u6536\u5230\u7684\u6CE8\u518C\u7801 :");

        label3.setForeColor(Color.HIGHLIGHT);

        label3.setLocation(new Point(8, 8));

        label3.setSize(new Point(288, 32));

        label3.setTabIndex(5);

        label3.setTabStop(false);

        label3.setText("\u6CE8\u518C\u7248\u7684\u529F\u80FD\u66F4\u52A0\u5168\u9762\uFF0C\u66F4\u52A0\u65B9\u4FBF\uFF0C\u5982\u679C\u4F60\u89C9\u5F97\u503C\u5F97\u7684\u8BDD\u5C31\u6CE8\u518C\u5427\u3002\u5C31\u5F53\u7ED9\u6211\u4E70\u5305\u70DF\u5F97\u4E86^\_^"); //在该窗体中显示的中文字符

        label4.setForeColor(Color.HIGHLIGHT);

        label4.setLocation(new Point(8, 40));

        label4.setSize(new Point(288, 16));

        label4.setTabIndex(6);

        label4.setTabStop(false);

        label4.setText("\u6CE8\u518C\u65B9\u6CD5 :");

        label5.setForeColor(Color.HIGHLIGHT);

        label5.setLocation(new Point(8, 56));

        label5.setSize(new Point(288, 40));

        label5.setTabIndex(7);

        label5.setTabStop(false);

        label5.setText("\u6CE8\u518C\u65B9\u6CD5\u8BF7\u8BE6\u89C1\u6211\u7684\u4E3B\u9875\u4E0A\u7684\u8BF4\u660E\u3002\u7528\u771F\u5B9E\u59D3\u540D\u4F1A\u6709\u60CA\u559C\u7B49\u4F60\u3002");

        button1.setLocation(new Point(224, 152));

        button1.setSize(new Point(72, 24));

        button1.setTabIndex(8);

        button1.setText("\u4EE5\u540E\u518D\u8BF4");

        button1.addOnClick(new EventHandler(this, "button1_click"));

        setNewControls(new Control[] {

            button1, label5, label4, label3, label2, label1, userNo, regB, userName

        });

    }

    private void regB_click(Object source, Event e)

    {

        name();

    }

    public static void main(String args[])

    {

        Application.run(new reg());

    }

    private void button1_click(Object source, Event e)

    {

        dispose();

    }

    public void dispose()

    {

        super.dispose();

        components.dispose();

    }

    public void name()

    {

        // 从这里开始是软件的注册算法

        char a[] = new char[100]; //定义一个 100 维的字符数组 a

        a = userName.getText().toUpperCase().toCharArray(); //将你输入的用户名转换为大写赋给数组 a

        char b[] = new char[20]; //定义一个 20 维的字符数组 b

        for(int i = 0; i < a.length; i++) //第一次循环计算，并将计算出的每个结果依次赋给数组 b

            b[i] = (char)((a[i] + (i + a.length) * a.length) % 26 + 65);

        for(int i = a.length; i < 20; i++) //第二次循环计算，并将计算出的每个结果依次赋给数组 b

            b[i] = (char)((a[0] * i * i) % 26 + 65);

        // 到这里就可以算出 20 个字符，分别存放在 b[0]至 b[19]中

        String s = new String("SpeedIP-" + String.copyValueOf(b)); //"SpeedIP"+字符数组 b=字符串 s

        // 到此注册算法完毕，注册码的格式为：SpeedIP-********************

        if(userNo.getText().equals(s)) //判断你输入的注册码是否与软件产生的一样，不一样则注册失败

            try

            {

                FileOutputStream fout = new FileOutputStream("named.dll");

                //注册成功后，在软件当前目录下创建一个 named.dll 文件

                DataOutputStream o = new DataOutputStream(fout);

                o.writeBoolean(true); //写入已注册标志 01

                o.writeUTF(userName.getText()); //将你输入的用户名写入到 named.dll 文件中

                o.close();

                //关闭文件

                fout.close();

            }

            catch(Exception _ex) { }

        else

            MessageBox.show("\u8BF7\u4F7F\u7528\u6B63\u7248\u6CE8\u518C\u7801\uFF0C\u4E0D\u8981\u90A3\u4E48\u5C0F\u6C14\u561B\uFF0C\u8C22\u8C22\uFF01", "\u4E0D\u597D\u54E6"); //“注册失败”的提示信息

        dispose(); //释放窗体

    }

    Container components;

    Edit userName;

    Button regB;

    Edit userNo;

    Label label1;

    Label label2;

    Label label3;

    Label label4;

    Label label5;

    Button button1;

}
```

经过 `Jad` 工具反编译后的 CLASS 文件果然可读性很高，经过分析注册算法后，我们知道该软件的注册码格式为：字符串“`SpeedIP-`”再加上 20 位字符串，即 `SpeedIP-********************`，其中 `*` 代表一个字符。上面注册算法的分析，我是以 C 语言的方式来理解的，所以，我就用 C 语言来编写一个与之相对应的注册器吧！

```c
#include "stdio.h"

main()

{

    int i,x;

    char a[100],b[20];

    while(1)

    {

        printf("\n******** China Beginner's Cracking Group ********\n");

        printf("This KeyGen is write by flithawk[BCG]!\n");

        printf("E-mail:flithawk@163.com\n");

        printf("Please Input your name(1-100):");

        scanf("%s",a);

        if(strlen(a)>0 && strlen(a)<101) break;

    }

    x=strlen(a);

    for(i=0;i<x;i++)

        b[i]=(toupper(a[i])+(i+x)*x)%26+65;

    for(i=x;i<20;i++)

        b[i]=(toupper(a[0])*i*i)%26+65;

    b[i]='\0';

    printf("Your register number is:SpeedIP-%s\n",b);

    printf("****** Welcome to BCG-http://www.china.net ******");
}
```



}

该注册器在 `Windows Me` 下用 `TC2.0` 编译通过。

通过该注册器计算出来一个有用的注册信息：

用户名：`flithawk[BCG]`

注册码：`SpeedIP-FYIGHNWXAOCTCASUGCIY`

当我用 `flithawk[BCG]` 为名注册成功后，打开 `named.dll` 后，发现它的格式为：

```
00000000 | 0100 0D66 6C69 7468 6177 6B5B 4243 475D | . flithawk[BCG]
```

其中：`01` 就是已注册标志，`0D` 表示用户名的长度。

知道了 `named.dll` 文件的格式后，我们完全可以自己构造一个，免去用注册器算注册码的麻烦，下面是我自己构造的一个 `named.dll` 文件，内容为：

```
00000000 | 0100 0342 4347 | . BCG
```

经过测试发现完全可以注册成功。但好像该软件不支持用中文名进行注册。

至此，整个软件的破解宣告结束，谢谢大家看完！

Crack by 飞鹰[BCG] `flithawk@163.com` 2002.8.16

欢迎光临汉化新世纪: `http://www.hanzify.org` `4dd466d0e1e0cc123afbe504da2462ca`
