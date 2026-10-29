
然后就是下午对布局文件的继续熟悉，感觉写布局代码越来越像写安卓代码的布局了。下面贴上全部代码，里面已经写了比较详细的注释，相信有编码基础的读者是可以看懂的：

```python
# -*- coding: utf-8 -*-

from PyQt4.QtGui import *
from PyQt4.QtCore import *
import sys
import os


class GridLayout2(QWidget):

    def __init__(self, parent=None):
        QWidget.__init__(self, parent)
        self.setWindowTitle('Android_APK')

        ###第一行
        self.apk_path = QLabel('APk Path')
        self.apk_path_edit = QLineEdit()

        ###第二行，三个按钮，这里第三个按钮的点击事件还没写
        file_search = QPushButton("Search_File")
        decode = QPushButton("Apk_Decode")
        build = QPushButton("Apk_Build")

        ###寻找文件按钮的点击事件
        self.connect(file_search, SIGNAL('clicked()'), self.openFile)

        ###反编译按钮的点击事件
        self.connect(decode, SIGNAL('clicked()'), self.apkD)

        grid = QGridLayout()
        grid.setSpacing(10)
        grid.addWidget(self.apk_path, 1, 0)
        grid.addWidget(self.apk_path_edit, 1, 1)

        grid.addWidget(file_search, 2, 0)
        grid.addWidget(decode, 2, 1)
        grid.addWidget(build, 2, 2)

        self.setLayout(grid)
        self.resize(350, 300)

    def openFile(self):
        ##调用`QFileDialog`的相关方法来打开文件选择框
        s = QFileDialog.getOpenFileName(
            self,
            "Open file dialog",
            "/",
            "Python files(*.apk)"
        )

        ##定义全局变量，用于接收需要反编译的apk的路径
        global APK_STR
        APK_STR = str(s)
        self.apk_path_edit.setText(APK_STR)

    def apkD(self):
        ###上面`openFile`方法中定义的全局变量，打印一下路径，看看是否正确#####
        print(APK_STR)

        ###apktool的命令使用#####
        apkToD = 'java -jar apktool.jar d ' + APK_STR

        ###调用`os.system`命令来执行cmd的相关命令###
        os.system(apkToD)


app = QApplication(sys.argv)
qb = GridLayout2()
qb.show()
sys.exit(app.exec_())
```

## 效果截图

`Search_file` 的点击事件：

![](https://bbs.kanxue.com/upload/attach/201409/357578_afadza2n82bltqf.png)

`Apk_Decode` 的点击事件：

![](https://bbs.kanxue.com/upload/attach/201409/357578_wrzxbif9pero4xg.png)

## 总结

上面的大家可以看到，我们已经可以实现两个功能了，即打开 `window` 找文件，然后进行反编译。但是在反编译完成以后，我们需要对里面的 `smali` 文件进行操作，然后再回编译文件、签名，才能对手机进行安装。

所以下一篇文章，我们就主要是编写回编译的方法以及对 `apk` 签名的方法。

## 0x4：第三天的最终成果

为了更方便地使用我们自己编写的工具，所以我们对于第三篇文章中的 `search_file` button 来进行处理。我们使用拖拽文件到 `QLineEdit` 控件上面来显示文件路径，而不用打开 `window` 来选择，这样对我们的操作来说更加方便一点。

### 拖拽事件的处理

拖拽事件经过一晚上的查询以及翻看 Java 已经写好的工具，终于找到 `dragEnterEvent` 和 `dropEvent` 两个事件。关于这两个事件的详细介绍，自行 Google 搜索一下即可。

要想使用这个事件，我们需要重写一下 `QLineEdit` 控件，并且在其 `init` 方法里面设置 `setAcceptDrops` 为 `True` 来接收拖拽事件。

所以现在的问题就是我们如何来获取得到拖入文件的路径呢？

经过查询，`mimeData` 有一些方法可以参考。经过测试，`urls()` 属性还是比较贴近我们所需的，但是还需要对其进行处理得到文件路径。

详细代码如下：

```python
def dropEvent(self, event):
    ###获取拖放过来的文件的路径
    st = str(event.mimeData().urls())

    print(st)

    ###这里没发现好的api，所以用`mimeData()`的`urls`属性了，`text()`不好用，所以下面就是字符串截取，来还原路径
    a = '///'
    b = ')]'

    num2 = st.index(b) - 1
    num = st.index(a) + 3
    st = st[num:num2]

    ##这里定义全局变量，拖入文件或者文件夹的路径
    global APK_STR
    APK_STR = st

    self.setText(st)
```

### 重新做一下布局文件

详细代码如下，有详细注释，这里不再多说：

```python
QWidget.__init__(self, parent)

self.setWindowTitle('Android_APK')

###第一行
self.apk_decode = QPushButton("Apk_Decode")
self.apk_path_edit = MyEditText(self)

###第二行，三个按钮，这里第三个按钮的点击事件还没写
self.apk_build = QPushButton("Apk_Build")
self.apk_build_edit = MyEditText()

###第三行
self.sign = QPushButton("Apk_Sign")
self.apk_sign_edit = MyEditText()

##第四行
self.shootcut = QPushButton("Shoot_Cut")
self.shootcut_text = QLabel("please keep moile with usb")

###布局文件的编写，四行两列
grid = QGridLayout()
grid.setSpacing(10)

grid.addWidget(self.apk_decode, 1, 0)
grid.addWidget(self.apk_path_edit, 1, 1)

grid.addWidget(self.apk_build, 2, 0)
grid.addWidget(self.apk_build_edit, 2, 1)

grid.addWidget(self.sign, 3, 0)
grid.addWidget(self.apk_sign_edit, 3, 1)

grid.addWidget(self.shootcut, 4, 0)
grid.addWidget(self.shootcut_text, 4, 1)

###设置窗口的高宽，将gridview放入大布局中
self.setLayout(grid)
self.resize(450, 300)
```

效果图如下：

![](https://bbs.kanxue.com/upload/attach/201409/357578_kma7m0p83guqw5x.png)

## 完整的代码

下面附上完整的代码，有兴趣的朋友，直接测试即可。记得先配置好 `PyQt` 相关开发环境。
