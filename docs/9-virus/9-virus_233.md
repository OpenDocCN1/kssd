

# ldj.exe 的分析！并附上部分专杀代码

> 原文：[`www.pediy.com/kssd/pediy12/138549.html`](https://www.pediy.com/kssd/pediy12/138549.html)

发改委网站被挂马，芳芳发现的，分析了一下  
一  
   
母体文件 ldj.exe 的分析：（母体文件在加载完 ocx 后代码开始混乱了，不知道是我的机子上还有别的木马，还是加载后的问题）  
主要有几个行为：1，生成 dbr99008.ocx 和 dbr31004.ocx  
2，复制 rundll.exe 到 C:\\WINDOWS\\SYSTEM32\\gbvgbv31.exe,并运行，加载上述文件，并注入到 explorer.exe 中  
3，Winston0\\default 方式交互  
4，解密出 4 个收信网址  
   
00401100 68 74 74 70 3A 2F 2F 67 http://g  
00401110 75 63 63 69 2E 74 6C 79 73 6A 2E 63 6F 6D 3A 39 ucci.tlysj.com:9  
00401120 39 36 32 2F 66 72 62 2F 72 62 2E 61 73 70 962/frb/rb.asp  
   
00401180 68 74 74 70 3A 2F 2F 67 http://g  
00401190 75 63 63 69 2E 74 6C 79 73 6A 2E 63 6F 6D 3A 39 ucci.tlysj.com:9  
004011A0 39 36 32 2F 66 74 2E 61 73 70 962/ft.asp  
   
00401208 68 74 74 70 3A 2F 2F 67 75 63 63 69 2E 74 6C 79 http://gucci.tly  
00401218 73 6A 2E 63 6F 6D 3A 39 39 36 32 2F 46 6F 6E 65 sj.com:9962/Fone  
00401228 39 2F 6A 75 73 74 2E 61 73 70 9/just.asp  
   
00401288 68 74 74 70 3A 2F 2F 76 35 2E 37 31 77 61 70 2E http://v5.71wap.  
00401298 63 6F 6D 3A 39 39 36 32 2F 46 6F 6E 65 39 2F 6A com:9962/Fone9/j  
004012A8 75 73 74 2E 61 73 70 ust.asp  
另外，还生成一种字体 ttf 文件，没发现有什么作用，应该是中间过度吧，不过一直在 fonts 文件夹下就没有找到这个文件！！  
   
母体加了 upx，到达 oep，像一些简单的函数，就不贴代码了  

代码:

[PRE0]

我们来看看解密函数（ror）：  
代码:

[PRE1]

代码:

[PRE2]

现在来看一看创建进程快照，并注入 explorer 的函数：  
代码:

[PRE3]

函数二：这个是真正的向进程中写入数据：  
代码:

[PRE4]

那我们就来看看函数 004019D7 中进行了什么操作：  
  
代码:

[PRE5]

这个就是注入函数了！  
  
代码:

[PRE6]

上述，通过 gbvgbv31.exe 运行 dbr31004.ocx,dbr99008.ocx 两个文件：  
  
C:\\WINDOWS\\system32\\dbr31004.ocx    0x10000000    0x0000E000  
C:\\WINDOWS\\system32\\DBR99008.OCX    0x00C30000    0x00006000  
  
注入到 explorer.exe 中的文件，C:\\WINDOWS\\system32\\dbr99008.ocx   基址：0x03800000，大小：0x00006000  
                            C:\\WINDOWS\\system32\\dbr31004.ocx   基址：0x03EB0000，大小：0x0000E000  
代码:

[PRE7]

三  
  
简单建立几个窗口，伪装成 ldjgame.exe，launch.exe 便于分析  
首先这是一个盗号木马，游戏有：鹿鼎记，SMDL(神魔大陆)，QQ 游戏，之前做的记录全都没了,现在就简单的分析几个线程吧  
并干掉 360 系列  
dbr31004.ocx 主要创建了 7 个线程：  
  
  
现在来看 dbr31004.ocx 文件的分析  
代码:

[PRE8]

简单写了点代码，大部分百度，不会编程的菜鸟真是伤不起啊，悲催！  
主要就是卸载注入到 explorer.exe 中的 dll 文件了，还有就是修改部分注册表，这个函数不是熟悉，也没敢乱写，哈哈！代码见附件！！！

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || KILL.rar  
 |