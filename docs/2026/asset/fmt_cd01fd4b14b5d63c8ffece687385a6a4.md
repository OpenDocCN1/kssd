
# SoftICE与TRW2000安装与配制

SOFTICE 有几个平台的版本：DOS、Windows 3.0、Windows 95/98、Windows NT 等。由于现在最普及的操作系统是 Windows 95/98、Windows NT、Windows Millennium、Windows 2000，因此就讲讲 SOFTICE 在这几个平台安装时的一些注意事项。

- [一、SOFTICE for win9x安装与配制](https://www.kanxue.com/#1)
  - [1. 显卡安装](https://www.kanxue.com/#1)
  - [2. 鼠标安装](https://www.kanxue.com/#2)
  - [3. `Autoexec.bat`和`config.sys`配制](https://www.kanxue.com/#3)
  - [4. Symbol Loader](https://www.kanxue.com/#4)
  - [5. `winice.dat`配制](https://www.kanxue.com/#5)
- [二、SOFTICE for Windows Millennium](https://www.kanxue.com/#a)
- [三、SOFTICE for NT/2K安装与配制](https://www.kanxue.com/#6)
- [四、TRW2000安装](https://www.kanxue.com/#7)

## SOFTICE 的安装与配制

### 一、SOFTICE for win9x安装与配制

#### SOFTICE 安装

1. SOFTICE 目前最新版本是 4.05，如你的系统是 win9x，就请下载 for win9x 版本的 SOFTICE，建议下载 SOFTICE 的最新版本，这样稳定性好些。运行 `setup.exe` 开始安装，出现（图一）：

   ![图一](https://www.kanxue.com/image/les501.gif)

   （图一）

2. 然后点击下一步，输入安装序列号（图二）（序列号一般在安装软件的 `readme.txt` 或其它说明文件里）：

   ![图二](https://www.kanxue.com/image/les502.gif)

   （图二）

3. 然后下面几个画面是要求选定路径和安装组件，不久你会来到显卡配制对话框（图三）：

   ![图三](https://www.kanxue.com/image/les503.gif)

   （图三）

   在显卡配制里两种方法：

   1. 这种配制使 SOFTICE 激活状态时类似 DOS 全屏状态一样（也就是字符模式状态），在显卡列表选择你的显卡类型，Universal Video Driver 和 Use monochrome card/monitor 这两项不要选，然后点击 **Test** 按钮，在测试过程中你能看到各种颜色的字符，说明显卡测试通过，就可点击下一步了。

   2. 这种配制使 SOFTICE 在激活状态下类似 Windows 应用程序的一个窗口那样，这样在调试时可避免显示器不停地在图形和字符模式转换，对提高显示器寿命大有好处。配制时，显卡列表一栏忽略，不用配制，只要把 Universal Video Driver 这一项选上，然后 Test，如跳出如图四的对话框，测试通过。（本人强烈推荐这种方式）

   ![图四](https://www.kanxue.com/image/les504.gif)

   （图四）

4. 再下一步是鼠标的配制（图五）：

   ![图五](https://www.kanxue.com/image/les505.gif)

   （图五）

   现在鼠标常见的一般是串行口或 PS/2 接口，你可以根据自己的鼠标接口类型或位置选上合适的就可。如碰到鼠标在 SOFTICE 调试画面不能用或一用死机，可能是没选好正确的选项，你可在 SOFTICE 菜单里运行 `mouse setup` 这个菜单项，重新配制鼠标。

5. 最后一步是询问以何种方式装载 SOFTICE 的主文件 `winice.exe`（图六）：

   ![图六](https://www.kanxue.com/image/les506.gif)

   （图六）

   首先要了解 SOFTICE for 9x 版本是如何装载的，在 SOFTICE 的安装目录下有 `winice.exe` 这个文件，Windows 启动到纯 DOS 环境下，运行 `winice.exe` 这个文件，将装载 SOFTICE。

   安装时默认将 `C:\PROGRA~1\NUMEGA\SOFTIC~1\WINICE.EXE` 这一行放进 `Autoexec.bat`（自动批处理文件），这样 Windows 以后每次都运行 `Autoexec.bat` 这个文件，自动装载 SOFTICE。另外，你可根据自身需要配制启动模式，具体参考第 8 步。

6. 然后安装程序将复制文件到硬盘里，来到最后一个对话框（图七）：

   ![图七](https://www.kanxue.com/image/les507.gif)

   （图七）

   最后一项是电子注册，注册成功后，以后可在线升级 SOFTICE 版本，这里选最后一项 **Register later**。

7. 至此安装完成，重新启动 Windows，微机先到 DOS 下，自动或手动运行相应批处理文件，运行其内的 `winice.exe` 文件，装载 Windows。

8. 启动菜单配制：

   通过配制启动菜单，启动时根据自己的需要选择是否装载 SOFTICE。

   **`AUTOEXEC.BAT` 配制样例：**

   ```
   @ECHO OFF
   goto %config%
   :SICE
   C:\PROGRA~1\NUMEGA\SOFTIC~1\WINICE.EXE
   goto common
   :NORM
   goto common
   :common
   ```

   **`CONFIG.SYS` 配制样例：**

   ```
   [MENU]
   MENUITEM NORM,Windows 9x
   MENUITEM SICE,Windows 9x with SoftICE
   MENUDEFAULT NORM,2

   [NORM]

   [SICE]

   [common]
   ```

#### SOFTICE 配制

##### Symbol Loader 的使用

在开始菜单的 SOFTICE 程序组里有一项 Symbol Loader 快捷方式，运行后，在其菜单 **EDIT** → **SOFTICE Initialization Settings** 选项，打开后如图（图八）。这里你就可配制 SOFTICE 了。

![图八](https://www.kanxue.com/image/lesson51.gif)

（图八）

- **General 选项**

  在 Initialization string 里，你可填上需要 SOFTICE 一启动自动运行的命令。如：

- **Exports 选项**

  在这里可添加相关的 DLL 文件，以便在 SOFTICE 下拦截这些 DLL 的函数。特别是破解 VB 程序时，定要将 VB 运行库装载进去。

- **Keyboard Mappings 选项**

  这里配制各功能热键。如：`F5="^x;"` 用 F5 键代替命令 `X`。

- **Macro Definitions 选项**

  宏定义，你可定制各种命令宏，以方便平时的操作。

  如：`s7878="S 30:0 L ffffffff '78787878'"` 用命令 `s7878` 代替一串命令：`S 30:0 L ffffffff '78787878'`

- **Remote Debugging**

  利用网络远程调试配制。

> **注：**以上所有配制好后的参数，都保存在 `winice.dat` 文件里。

##### `winice.dat` 配制

在 Windows 9x 下 SOFTICE 配制除了用上面的方法外，也可通过文件 `winice.dat` 来实现的。Soft-ICE 在启动的时候通过它装入一些 DLL/EXE 的信息。

你可在 SOFTICE 安装目录下发现 `winice.dat`，可用任何文本编辑软件打开它（如记事本）。如我机子里的 `winice.dat`：（你可参考我的来修改你的 `winice.dat`）
