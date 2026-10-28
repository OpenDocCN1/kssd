
`.data:00405068                     dd offset aHttpGoogleCoCk_25 ; "012K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6G2L8$3N6D9k6g2)9J5k6h3y4G2i4K6u0W2j5$3E0Q4x3V1k6K6k6h3q4J5j5$3S2Q4x3@1k6I4i4K6y4p5K9X3!0Z5L8W2)9J5b7X3y4W2L8X3q4Q4x3V1u0Q4x3U0k6I4N6h3!0@1i4K6y4n7i4K6u0W2i4K6u0W2i4K6u0W2`

`.data:0040506C                     dd offset aHttpGoogleCoCk_26 ; "c65K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6G2L8$3N6D9k6g2)9J5k6h3y4G2i4K6u0W2j5$3E0Q4x3V1k6K6k6h3q4J5j5$3S2Q4x3@1k6I4i4K6y4p5N6X3W2F1k6i4y4S2N6h3y4W2i4K6u0n7i4K6t1$3M7i4g2G2N6q4)9K6b7W2)9J5k6g2)9J5k6g2)9J5k6b7"`.

`.data:00405070                     dd offset aHttpGoogleCoCk_27 ; "555K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6G2L8$3N6D9k6g2)9J5k6h3y4G2i4K6u0W2j5$3E0Q4x3V1k6K6k6h3q4J5j5$3S2Q4x3@1k6I4i4K6y4p5M7$3E0J5K9h3I4D9k6i4S2Q4x3V1u0K6i4K6t1$3M7i4g2G2N6q4)9K6b7W2)9J5k6g2)9J5k6g2)9J5k6b7"`.

`.data:00405074                     dd offset aHttpAnswersMic ; "011K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1M7%4N6W2M7Y4y4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3M7s2u0G2N6q4)9J5y4Y4q4#2L8%4c8Q4x3@1u0Q4x3X3g2Q4x3X3g2Q4x3X3f1"`.

`.data:00405078                     dd offset aHttpMotherboar ; "4d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2G2N6r3S2W2M7X3u0G2j5i4u0V1i4K6u0W2N6X3W2U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2j5h3c8Q4x3V1k6%4j5i4c8U0K9q4)9J5k6q4)9J5y4Y4q4#2L8%4c8Q4x3@1u0Q4x3X3g2Q4x3X3g2Q4x3X3f1"`.

`.data:0040507C                     dd offset aHttpPlayClubpe ; "caaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8D9j5i4W2Q4x3X3g2U0L8s2g2T1M7r3g2F1k6%4g2A6L8W2)9J5k6h3y4G2L8g2)9J5y4Y4q4#2L8%4c8Q4x3@1t1"`.

`.data:00405080                     dd offset aHttpPcoptimize ; "2eaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8U0L8%4m8@1K9h3#2A6P5X3g2J5M7s2u0G2i4K6u0W2j5$3!0E0i4K6t1$3M7i4g2G2N6q4)9K6b7R3"`.

`.data:00405084                     dd offset aHttpSoftonicCo ; "6c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4G2k6Y4c8G2L8X3W2U0i4K6u0W2j5$3!0E0i4K6t1$3M7i4g2G2N6q4)9K6b7R3"`.

`.data:00405088                     dd offset aCalc ; "calc"`

`.data:0040508C                     dd offset File ; "notepad"`

`.data:00405090                     dd offset aCmd ; "cmd"`

`.data:00405094                     dd offset aWrite ; "write"`

`.data:00405098                     dd offset aRegedit ; "regedit"`

`.data:0040509C                     dd offset aExplorer ; "explorer"`

`.data:004050A0                     dd offset aTaskmgr ; "taskmgr"`

`.data:004050A4                     dd offset aMsconfig ; "msconfig"`

`.data:004050A8                     dd offset aMspaint ; "mspaint"`

`.data:004050AC                     dd offset aDevmgmtMsc ; "devmgmt.msc"`

`.data:004050B0                     dd offset aControl ; "control"`

`.data:004050B4                     dd offset aMmc ; "mmc"`

至此，整个`/watchdog`部分的分析就结束了。总结一下：当样本以`/watchdog`参数运行时，会出现一个窗口，然后会打开乱七八糟的网页和一些系统程序，弹出信息框且当有窗口创建时会把宽高修改为随机大小，然后蓝屏。由于存在MBR的修改，导致重启时电脑无法正常启动，且会播放彩虹猫的动画。

# 二、`/main`

![](https://bbs.kanxue.com/upload/attach/202110/826671_B653HKF82EUF4VT.jpg)

以`/main`参数运行的代码部分相对来说比较简单，首先是两个提示性的弹窗，然后就是创建了5个以`/watchdog`参数的进程，此处与前面判断进程数减少部分对应。最后是把自己的优先级到最高`HIGH_PRIORITY_CLASS(0x80)`。

# 三、MBR部分函数

### [1] `MoveMouse`-随机移动鼠

![](https://bbs.kanxue.com/upload/attach/202110/826671_2K35ZCWEH3ZEPPZ.jpg)

### [2] `ChangeDeskBg`-修改桌面背景

![](https://bbs.kanxue.com/upload/attach/202110/826671_GYJWMS5VMVXP6Z8.jpg)

### [3] `sub_401688`-枚举窗口

![](https://bbs.kanxue.com/upload/attach/202110/826671_6KS86S2GU8HRMEE.jpg)

### [4] `PlayBeep`-播放声音

![](https://bbs.kanxue.com/upload/attach/202110/826671_GJK68WB5VCP2D4M.jpg)

### [5] `sub_4016CD`-乱画桌面

![](https://bbs.kanxue.com/upload/attach/202110/826671_JRPEQAUHP9UZMJ5.jpg)

### [6] `autoKeydown`-自动按键

![](https://bbs.kanxue.com/upload/attach/202110/826671_VKJB4ZZ4JU9W26N.jpg)

### [7] `DrawIcon`-随机位置画Icon

![](https://bbs.kanxue.com/upload/attach/202110/826671_DK7X7W3CAPRFZF9.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-10-26 20:30 被PlaneJun编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
