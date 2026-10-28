
![](https://bbs.kanxue.com/upload/attach/202106/921497_GKRYPTTUWF42VTA.jpg)

进入第三个计时器,功能为命令删除共享文件

![](https://bbs.kanxue.com/upload/attach/202106/921497_PYY4VNURWW3GVE4.jpg)

进入第四个计时器,发现功能为删除一些杀毒软件的启动项

![](https://bbs.kanxue.com/upload/attach/202106/921497_37YVFRXTNCFHEUM.jpg)

到此动态静态分析就结束了

## 四.总结主要行为

1. 自我复制,添加注册表自启动项
2. 释放文件,设置隐藏属性
3. 删除GHO文件
4. 结束安全软件进程,删除安全软件服务和启动项
5. 访问指定网址,下载病毒文件
6. 感染PE文件
7. 创建线程感染文件

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-6-16 21:15

被wx\_dopeusa91编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
