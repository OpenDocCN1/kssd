
# regmon 用法

> 原文：[`www.pediy.com/kssd/pediy05/pediy50324.htm`](https://www.pediy.com/kssd/pediy05/pediy50324.htm)

次数限制？169 字节轻松搞定

为了一个很狡猾的学习软件（驾驶员相关），浪费了不少时间，我和我的老机都很累了。。。今天想轻松一下，顺便找回一点自信，故从网上拖了一个《斗地主 4.8 build 462》下来。。。

软件名称：`斗地主 4.8 build 462`

保护主式：30 次使用限制和每次只能玩 7 局的限制

得力工具：`Regmon v4.12`

DIY 目的：解除 30 次使用限制（至于只能玩 7 局的限制，一方面我还有其它事得做，另一方面，我想每个象我一样的 `Cracker` 都很忙。。。好象玩这种游戏永远也不玩不到 7 局的。。。呵呵。。。为自己找了个好理由^Q^）

郑重声明：写这篇文章的目的只是为了和所有爱好 `crack` 的朋友交流，希望能取到抛砖引玉的效果。如果周军先生也看到的话，我要说的是，假使此文侵犯了你的利益，说明你的软件这方面很弱，确需改进！

哪哪哪哪哪哪哪哪哪哪哪哪哪哪 `Begin!` 哪哪哪哪哪哪哪哪哪哪哪哪哪哪

一、首先用 `winice` 跟踪，得知该软件的注册码验证是：输入注册码时在注册表里写入一个格式化的 `regcode`，当下次启动该软件时调用 `RegQueryvalueexA` 来读出并对比注册码。。。不过我们今次的目的只是为了解除 30 次使用限制，别的就不多说，考虑到作者会骂我。

二、用 `peid0.8` 分析主程序文件 `ddz.exe`，显视 `UPX 0.89.6 - 1.02 / 1.05 - 1.22 -> Markus & Lazlo`。。。用 `trw2000` 脱壳后能正常运行，但是点击其菜单按钮时就导致自动关机，又用 `winice` 及 `trw2000` 试拦了很久。。。终不得要领，不知其关机动作是调用了什么鬼东东，希望哪位高手能指点一二。。。

三、既然脱壳后找不到其关机的代码，也不想让老机再受折磨，只好放弃，另想别的办法。

四、打开 `regmon`，准备好后运行斗地主。。。在捕获的事件列表上看到了关于斗地主 4.8 的内容：

```
783    Ddz    OpenKey    HKLM\\SOFTWARE\\Microsoft\\VBA\\Monitors    NOTFOUND   


784    Ddz    OpenKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS    hKey: 0xCAA307F0   


785    Ddz    QueryValueEx    HKLM\\SoftWare\\DDZ\\Infomation\\ProductId    SUCCESS    "DDZ040020020378"   


786    Ddz    CloseKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS   


787    Ddz    OpenKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS    hKey: 0xCAA307F0   


788    Ddz    QueryValueEx    HKLM\\SoftWare\\DDZ\\Infomation\\LicenseNumber    SUCCESS    "4"------------------------>注意这里！指明程序已经运行了 4 次。。。


789    Ddz    CloseKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS   


790    Ddz    OpenKey    HKCU\\Control Panel\\International    SUCCESS    hKey: 0xCAA20060   


791    Ddz    OpenKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS    hKey: 0xCAA307F0   


792    Ddz    QueryValueEx    HKLM\\SoftWare\\DDZ\\Infomation\\SetupDir    SUCCESS    "G:\\DDZ"   


793    Ddz    CloseKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS   


794    Ddz    OpenKey    HKLM\\Software\\Microsoft\\Windows\\CurrentVersion    SUCCESS    hKey: 0xCAA307F0   


795    Ddz    QueryValueEx    HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\ProductName    SUCCESS    "Microsoft Windows 98"   


796    Ddz    CloseKey    HKLM\\Software\\Microsoft\\Windows\\CurrentVersion    SUCCESS   


797    Ddz    OpenKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS    hKey: 0xCAA307F0   


798    Ddz    QueryValueEx    HKLM\\SoftWare\\DDZ\\Infomation\\Version    SUCCESS    "4.8 Build 462"   


799    Ddz    CloseKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS   


800    Ddz    OpenKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS    hKey: 0xCAA307F0   


801    Ddz    QueryValueEx    HKLM\\SoftWare\\DDZ\\Infomation\\CPUCode    SUCCESS    "4622870839"   


802    Ddz    CloseKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS   


803    Ddz    OpenKey    HKLM\\Software\\Microsoft\\Windows\\CurrentVersion    SUCCESS    hKey: 0xCAA307F0   


804    Ddz    QueryValueEx    HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\ProgrameID    SUCCESS    "4"    ------------------------>注意这里！这里好指明了程序已经运行了 4 次。。。


805    Ddz    CloseKey    HKLM\\Software\\Microsoft\\Windows\\CurrentVersion    SUCCESS   


806    Ddz    OpenKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS    hKey: 0xCAA307F0   


807    Ddz    QueryValueEx    HKLM\\SoftWare\\DDZ\\Infomation\\regcode    SUCCESS    " "   


808    Ddz    CloseKey    HKLM\\SoftWare\\DDZ\\Infomation    SUCCESS   


809    Ddz    QueryValueEx    0xC1903230\\ASYCFILT    NOTFOUND   


```

五、大家看到 `regmon` 的强大之处了吧，一下子让我们得到了这么多珍贵的信息，不过，对于此次的目的，只要注意“`788    Ddz    QueryValueEx    HKLM\SoftWare\DDZ\Infomation\LicenseNumber    SUCCESS    "4"`”和“`804    Ddz    QueryValueEx    HKLM\Software\Microsoft\Windows\CurrentVersion\ProgrameID    SUCCESS    "4"`”这么两条就足够了，程序在每次启动时都要检查这两条注册表信息，如果>=30，就弹出注册对话框，点击取消或输入错误的注册码都会让你无福消受。。。。

六、现在，我们可以修改注册表了，将以上两项的值改为 0（一定要两项哦！只改一项的话是没用的，不信你试试^Q^),再次运行程序，点击“关于”，你看到了什么？---------已用次数：1 次。。。。呵呵，就这么简单！

七、用记事本写出“延时器”如下：

```
REGEDIT4

\[HKEY\_LOCAL\_MACHINE\\Software\\DDZ\\Infomation\]

"LicenseNumber"="0"

\[HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\]

"programeID"="0"
```

另存为`*.reg` 后，我们的目的达到了！因为保存后的 `reg` 文件只有 169 字节，故引出了文章的题目。。。。

八、总结，虽然采用了很“绿色”的方法解决了程序的次数限制，但美中不足的是当程序运行次数达到 30 次时都要运行一次我们写出的 `reg` 文件，怪只怪在下学艺不精。。。摄写此文的原因，一是希望启发象我这样的初学者，在 `pj` 时不要在一棵树上吊死，多一点灵感你就成功了一半！二是希望能得到高手们的赐教，象前面说的一样取到抛砖引玉的效果。

九、再见！

作者：飞龙狗狗    2002.10.8    希望保持文章的完整性  `4dd466d0e1e0cc123afbe504da2462ca`
