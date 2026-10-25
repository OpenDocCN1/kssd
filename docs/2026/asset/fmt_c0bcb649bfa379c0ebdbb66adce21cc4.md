
可以看到函数头部地址为：`00007FF6780AD078` | `mov qword ptr ss:[rsp+8],rbx`

函数尾部地址为：`ret`

修改之后，函数头部地址为：`00007FF6780AD078` | `ret`

![![](https://bbs.kanxue.com/upload/attach/201707/214974_jl8zcuqhgajwfom.png)](https://bbs.kanxue.com/elink@1e2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6X3A6D9z5s2A6U0N6i4q4Z5k6$3q4B7N6$3k6G2L8g2)9J5k6i4m8F1k6H3`.`.)

图6 修改函数头部反汇编指令

修改之后，鼠标右键选择补丁-修补文件。

![![](https://bbs.kanxue.com/upload/attach/201707/214974_io71g4iihb4sm3x.png)](https://bbs.kanxue.com/elink@9feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6X3W2G2y4K6q4Y4y4r3W2A6K9r3t1@1M7$3@1K6P5q4)9J5k6i4m8F1k6H3`.`.)

图7 选择补丁

![![](https://bbs.kanxue.com/upload/attach/201707/214974_9mydl34bmsm1aps.png)](https://bbs.kanxue.com/elink@0e4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6U0W2E0P5h3c8D9x3K6c8T1L8i4y4E0x3h3q4H3M7#2)9J5k6i4m8F1k6H3`.`.)

图8 修补文件

小功告成！再次打开rar弹窗广告已经消失了。可是评估版本字样还在，追求完美可以选择使用资源管理工具去除字样。

![](https://bbs.kanxue.com/upload/attach/201707/214974_wn15zsxrl5hj7a5.png)

图9 去除弹窗

我使用了`Restorator`这款资源修改软件，不过使用这款软件的少侠们可就小心了。因为这款资源软件会自动修改`.exe`扩展名的关联，**请切记在虚拟机下运行。**

![![](https://bbs.kanxue.com/upload/attach/201707/214974_06ycrkfowl0veii.png)](https://bbs.kanxue.com/elink@dbcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8X3q4@1N6r3q4U0K9q4)9J5c8U0t1H3x3e0M7H3y4#2)9J5c8U0t1I4y4o6V1%4y4q4)9#2k6U0l9$3P5h3y4J5K9$3k6G2N6$3H3H3N6X3g2A6K9g2)9J5k6i4m8F1k6H3`.`.)

图10 导出资源文件.rc

导出资源文件`.rc`后，用`notepad++`打开`.rc`后缀的文件。修改【评估版本】为你想要修改的文字。然后再导入到资源中。操作比较简单，就不赘述了！

![](https://bbs.kanxue.com/upload/attach/201707/214974_fpzyx6vnkxcn737.png)

图11 大功告成！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
