
# DOTNET 动态调试破解 Spoon，及 MSI 安装包文件替换技术

> 原文：[`www.pediy.com/kssd/pediy12/140557.html`](https://www.pediy.com/kssd/pediy12/140557.html)

**标 题**: DOTNET 动态调试破解 Spoon，及 MSI 安装包文件替换技术  
**作 者**: xiaolu  
**时 间**: 2011-09-24  
**链 接**: http://bbs.pediy.com/showthread.php?t=140557  

提到 Spoon 可能大家还会感到陌生，但是如果提及 `XenoCode` 那么研究过 `DOTNET` 的人应该都知道吧。`Spoon` 的前身就是 `XenoCode`，虽然没有了 `PostBuild` 这个混淆软件，但是虚拟化技术仍然在完善，`Spoon` 可以让 `DOTNET`、`JAVA` 和 `Flash` 程序脱离 `Framework` 在电脑上独立运行。最新的 `spoon` 已经可以支持 `DOTNET4.0` 了。  

网上破解 `Spoon` 的教程几乎没有，有的也只有 `Danny_Su` 直接提供的破解版本。而我最为羡慕的就是 `Danny` 提供的版本中拥有原版安装界面。经过几天的研究，终于实现了此功能，特分享给大家。  

首先可以利用 `NETUNPACK` 获得 `DOTNET` 的 `Assembly` 文件，而可以利用 `Ollydbg` 来获得非托管文件。早在 `XenoCode` 时代，网上就有了视频教程。获得脱壳的文件后，我们将使用一些其他的工具来替代传统的 `Reflector` 进行追踪。  

首先出场的是初级 `DOTNET` 调试软件 `DILE`。先运行 `Spoon`，然后使用 `DILE` 的 `Attach` 功能来附加到 `Spoon` 的进程上。  

![](img/acb892b292c968aedfbc0dab2b1589df.jpg)  

附加成功后，选择 `Spoon Studio` 中菜单中的 `Install License Certificate`  

![](img/a90b122902830c0bde5b04410f041114.jpg)  

在弹出的对话框中不填写任何东西，直接点击确定，结果 `Spoon` 系统内部出抛出异常，而此时的 `DILE` 会捕获这个异常。  

![](img/cf31aa41e6f5a02dd21a8c9b4d3d265e.jpg)  

`DILE` 会停在上图的代码处，然后我们选择 `Call Stack Panel`，就可以查看引起这个异常时哪里调用的了，然后我们双击上图中圈出的类。  

进入这个 `Spoon.Common.Accounts.X0615e395e52a99d0::_x5d487333ced49b69` 类后，我们按 `F9` 设置一个断点。按 `F5` 让程序跑飞后，我们回到 `Spoon` 再次输入证书。这次我们不填空了，而是直接输入 `Danny` 以前的破解版的 `License`  

![](img/0bf752f8a05430c12fee895832966fa5.jpg)  

点击 `OK` 后，程序会中断在我们前面设置的断点处。  

![](img/441406221a84daad6b21f3aa22dd6234.jpg)  

按 `F10` 单步，一路下来，并随时注意观察 `IL` 代码，灵活运行 `F11` 和 `F12` 单步调试能力，很快就可以定位到下图的类中。（具体怎么找到，就要看各位平时的调试能力了，多练习就会很快定位）  

![](img/dce71bd10e843fe64a876c59fb8daae7.jpg)  

这个时候 `ILSPY` 上场了，找到 `Spoon.Common.Accounts.dll` 的类 `x0615e395e52a99d0` 和方法 `x22ab5dfa6f12e902` 名，发现这个方法是有一个重载，如下图所示只要将这个方法的返回值全部设置为 `true` 就可以了。  

![](img/58b8cedc795119ba5f5892f6c677c6b3.jpg)  

我们再将 `ILSPY` 的反编译的语言设置为 `IL`，那么就可以看到这个方法的 `RVA` 位置。  

![](img/016b46e328ca078f894cb8e62dc1a1bf.jpg)  

打开 `CFF Explorer`，将 `Spoon.Common.Accounts.dll` 文件拖入到该工具中，首先找到强名的相关表，发现居然没有强名验证，这样我们破解就更简单了。  

![](img/084ea817248f727675448975bb94fad0.jpg)  

在 `Address Converter` 中输入前面我们找到的 `RVA` 地址 `0x4a90`，我们就快就可以定位 `Hex` 地址了。只要在这一段二进制中将返回值设置为 `true` 就行了，经常玩 `DOTNET` 破解的就应该知道只需要将二进制 `16` 修改为 `17` 就行了。我们找到这段 `IL` 代码中的五个 `16`，然后全部修改为 `17` 后保存即可。  

![](img/de9383d63c154c7504f6f7ca431aa116.jpg)  

重新使用 `ILSPY` 打开，我们发现，流程已经更改了。  

![](img/3bb0a7f0f12ce480293388b890c3af6e.jpg)  

我们重新注册，发现注册成功了，新建一个文件试一试，也没有提示过期了。破解到此已经结束，是不是很简单。破解完毕了，这并不是我们的终极目标，为了保持原汁原味，我们决定还将此文件打包回去。  

利用我们破解的程序将 `Spoon` 这么多的文件再打包成一个文件，将 `StartFile` 设置为 `Studio.exe`，在 `Runtime` 设置中，我们设置为 `DOTNET3.5 SP1`，打包结束后，我们获得了一个大小为 121 兆左右的单一 `Studio.exe` 文件。  

![](img/efe761cf01012a14de90f4ad39baef4d.jpg)  

现在我们应该想办法将这个文件替换到 `Spoon` 原版安装包中。我们需要三个程序：`MsiDb.exe`，`Orca`，`CABARC.EXE`。

## 1. 使用 Orca 打开安装包文件

在左侧找到 `Media`，在右侧记下流文件名叫做 `#disk1.cab`，这个压缩包将在第 2 步用到。

![](img/18603a91ff4ccec169768c14be9c6ace.jpg)

## 2. 使用 MsiDb 来提取这个 cab 文件

执行 `cmd` 命令：

```
msidb -x disk1.cab -d Setup-9.4.1860.0.msi
```

获得压缩文件 `disk1.cab`  

![](img/8c93523b21c7367d942c95d2c482bd99.jpg)  

这个时候可以直接使用 `winrar` 进行解压缩。发现有三个文件 `VAStudio.exe`，`Studio.chm`，`XStudio.exe` 其中 `VAStudio.exe` 就是我们要替换的文件。我们将破解的 `Studio.exe` 更名为 `VAStudio.exe` 替换原版文件。我们在执行下面的命令来重新制作 `cab` 压缩包：  

```
cabarc -m LZX:21 -r N disk1.cab "disk1\\*"
```

## 3. 替换新的 cab 压缩包

`cab` 压缩包制作好后，我们再使用 `MsiDb.exe` 将新的压缩包替换进去。首先删除原 `MSI` 文件中的 `disk1.cab` 文件流：  

```
msidb -k disk1.cab -d Setup-9.4.1860.0.msi
```

再次，添加新的的 `disk1.cab` 文件：  

```
msidb -a disk1.cab -d Setup-9.4.1860.0.msi
```

这时候，新的安装包就制作好了。我们测试一下，果然新的安装包既保持了原版风味，而且是破解版本。  

![](img/f9f13f6a10b16a65e81015aa44006718.jpg)  

## 文中提到的软件地址

- `Spoon`：[`www.spoon.net/Studio/`](http://www.spoon.net/Studio/)  
- `DILE`：[`sourceforge.net/projects/dile/files/DILE/`](http://sourceforge.net/projects/dile/files/DILE/)  
- `CFF Explorer`: [`www.ntcore.com/exsuite.php`](http://www.ntcore.com/exsuite.php)  
- `MsiDb.exe`，`Orca`，`CABARC.EXE` 可以自己 `Google` 之。  

欢迎大家和我联系 `chenxiaolu2012@gmail.com`  

文章最后：感谢 `BF` 给我的技术支持。是他将我带入了 `DOTNET` 破解的神奇领域。
