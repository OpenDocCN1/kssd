

# 自己动手写”Hshield”:第一章:立即动手写一个最让人看不起的”Hshield

> 原文：[`www.pediy.com/kssd/pediy11/113330.html`](https://www.pediy.com/kssd/pediy11/113330.html)

自己动手写”Hshield”保护软件   
                     第一章:立即动手写一个最让人看不起的”Hshield”  
  
  
                           千里之行始于足下.  
                                                                 ---古代人说的  
**1,1, 准备工作.**  
出于写程序，准备工作就跑不开硬件，软件这两方面。我们来看一下：  
1，硬件  
。一台计算机（Windows 操作系统）  
  
2，软件  
。安装正版编程软件 Microsoft Visual C++ 6.0（若是中国人可以安装翻版的）  
。随便一个没 anti-debug .的调试器.  （若没，可使用本人在附件中“赠送”的 OD）  
  
1,2.  1 分钟完成的”Hshield”  
  你相信不相信，一个”Hshield”关键只要 7 句话  ，请看。  
![`bbs.pediy.com/picture.php?albumid=176&pictureid=449`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=449)  
                   代码 1-1  \\chapter\\a\\HideDll\\HideDll.cpp  
   
                 我们的最让人看不起眼的”Hshield”核心代码  
  
证明其有 Hshield 功能的步骤：  
1,先随意打开一个标题为”Test”的被保护进程(若无者,有空者可自制,但我已把这个放在源码 Test)  
   
![`bbs.pediy.com/picture.php?albumid=176&pictureid=443`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=443)  
                    图 1-1 :Test 作为测试程序  
   
![`bbs.pediy.com/picture.php?albumid=176&pictureid=451`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=451)  
                    图 1-2: 用冰丸查看 Test 线程模块  
  
  
用 OD 调试.  正常附加之后. 再 F9 运行.程序是绝无反应. (在 tool 里边有 od.exe)  
 (嘻嘻,因为它还没有 我们的”Hshield”保护)  
 ![`bbs.pediy.com/picture.php?albumid=176&pictureid=444`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=444)  
                          图 1-3: 用 OD 调试 Test 无反应  
  
  
  
2,同样.我们先打开程序 Test  
  这次先打开 launcher.exe 向 Test 发射.  
 ![`bbs.pediy.com/picture.php?albumid=176&pictureid=442`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=442)  
              图 1-4 :发射程序 launcher  
成功保护后会这样的提示:  
   
![`bbs.pediy.com/picture.php?albumid=176&pictureid=445`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=445)  
                        图 1-5: Launcher 成功向 Test 加载我们的”Hshield”  
(可以把 launcher.exe 关闭)  
   ![`bbs.pediy.com/picture.php?albumid=176&pictureid=450`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=450)  
                        图 1-6: 用冰丸查看被发射后的 Test 线程模块                 
  
  
  之后用 OD 附加,F9 运行程序.你还会显然发现其中的变化  
 ![`bbs.pediy.com/picture.php?albumid=176&pictureid=445`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=445)  
                     图 1-5:OD 调试 Test 被检测发现  
                    
     看.我们的“Hshield”就是这么牛 X. 一个待启动线程 DLL,一个发射台..然我还是自已表达太复杂了.  
    还是把问题用另一种方式表达.会简单些   
  
  
  
1.3 思路整理;  
  
 ![`bbs.pediy.com/picture.php?albumid=176&pictureid=448`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=448)  
                图 1-6: 各程序之间的关系  
   此刻从整体而言  “Hshield”最需要思考的是: HideDll.dll  .它的代码强度决定着整个”Hshield”的反调试强度.  
   
![`bbs.pediy.com/picture.php?albumid=176&pictureid=447`](http://bbs.pediy.com/picture.php?albumid=176&pictureid=447)  
读者可以通过 附加的源码. 任意增加检测调试器的方法. 组成自已特色的”Hshield”  
  
(发射台,Test 普通程序是次支,可以管可不管.).  
  
1,4 理解与扩展.  
如果对”Hshield”所用的技术.  
1,创建远程线程.  
2,检测调试器.  
  
对远程线程不熟可以读读<<Window 高级编程指南>>  --第十八章  打开进程边界  
看看出<<Cracking, The Anti.>>可以检测调试器.解惑..  
  
////////后记//////////  
第一次发文.才发现写文是件很累很累的事..  
明明做起来是很简单的事情.. 如果要用文字表达.则是很复杂的事.    
  
  
  
资料引用说明：  
反调试：  
1、<<脱壳的艺术>> hawking  
              2、<<windows anti-debugger reference>> Angeljyt  
              3、http://bbs.pediy.com   
              4、<<软件加密技术内幕>> 看雪学院  
              5、<<ANTI-UNPACKER TRICKS>> Peter Ferrie  
              6，  <<Cracking, The Anti.>>  
         国内也已经有脱壳大牛 shellwolf 编写“反调试技巧总结-原理和实现“作整体总结  
  
                 
  
编程资料：<<Window 高级编程指南>>  --第十八章  打开进程边界  
           Jeffrey Richter\[美\] 著         
  
           (网上皆其有下载)

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || chapter1.rar  
 |