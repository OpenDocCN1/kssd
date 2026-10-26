
# [原创]寻找QQ连连看棋盘数组基址

大家好，今天来给大家介绍QQ连连看的棋盘数组基址。

我们让旗盘的第一格就是左上角的第一格，让他为零，这样方便我们搜索。

CE、搜索 未知的初始值

![](https://bbs.kanxue.com/upload/attach/201601/684330_xae07jrbtikynff.jpg)

CE，搜索 精确数值 0

![](https://bbs.kanxue.com/upload/attach/201601/684330_xdmuixqsjwtx7y7.jpg)

CE 搜索 比0大的数值

![](https://bbs.kanxue.com/upload/attach/201601/684330_logo5iv6g8i57zm.jpg)

就这样反复搜索直到出现四个。

![](https://bbs.kanxue.com/upload/attach/201601/684330_qkm1ns5vom0dwo6.jpg)

然后关掉游戏重新开始，记住CE别关，再用CE打开游戏，然后提示是否载入列表？是，现在只有一个是有数值的，那就时棋盘基址，也是第一个格子的基址，这样你可以试着改变他的数值，1,2,3都可以，看到了吧，图案变了，然后就是对了。

![](https://bbs.kanxue.com/upload/attach/201601/684330_758u8uwj7ejzg4x.jpg)

最后用我给你的源码，在指定位置，填入基址，就可以了。

![](https://bbs.kanxue.com/upload/attach/201601/684330_744ikaykzzdsybg.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`30NN98ISEVDWVAMRT]3`4S.jpg](javascript:void(0);)  
  （73.67kb，13次下载）

- [3}11JOU(H`6U9CK3E89J1KG.jpg](javascript:void(0);)  
  （87.05kb，13次下载）

- [B78})1{8GY[(Q`)7%LR`\_PD.jpg](javascript:void(0);)  
  （80.42kb，15次下载）

- [FHWY]MOQE0$U$R3}WE6}8DO.jpg](javascript:void(0);)  
  （91.14kb，10次下载）

- [K%M{(I(}7}9GK2FS[GO]X%M.jpg](javascript:void(0);)  
  （70.98kb，24次下载）

- [RC]2`6912DA5~QI[P~Q(310.png](javascript:void(0);)  
  （38.78kb，13次下载）

- [llk\_wg源代码.zip](javascript:void(0);)  
  （58.01kb，55次下载）
