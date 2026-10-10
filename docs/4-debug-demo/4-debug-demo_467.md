

# 周公解梦 V2.11

> 原文：[`www.pediy.com/kssd/pediy02/forum207.htm`](https://www.pediy.com/kssd/pediy02/forum207.htm)

 虽然这个软件很烂,可以说是一个垃圾,但我认为它是一个练习破解的好东东.锻炼你的综合破解能力.破解它,首先你得对它进行脱壳,然后改日期限制,改设置首页的问题.改.....总之要改的地方还很多.不知那位高手能出一篇详细的破解过程.在此先谢过了,欢迎大家一起参与讨论.!!!  

 周公解梦 2.11 破解   
  
这个程序（能称为软件吗？），正好昨晚做了梦想看看结果所以就用了看看，气人的是它竟然会自作主张将浏览器设为它的首页，不知有多少人深受其害？   
１、首先脱掉它的壳（脱壳不难）   
２、解决时间限制问题，由于它将时间过期限制标志写入文件设断如下：   
bpx WritePrivateProfileStringA   
  
:0044B280 50                      push eax   
:0044B281 E836B2FBFF              call 004064BC    // WritePrivateProfileStringA 这里将过期信息写入文件   
:0044B286 85C0                    test eax, eax   
:0044B288 7527                    jne 0044B2B1   
  
３、解决将浏览器设为它的首页,问题，浏览器首页设置在注册表   
HKCU\\software\\Microsoft\\Internet Explorer\\main\\Start Page 中设断如下：   
Bpx RegSetValueExa   
:0044BD74 50                      push eax   
:0044BD75 E832A5FBFF            call 004062AC  // RegSetValueExa 这里写入注册表将其设为首页   
:0044BD7A 85C0                    test eax, eax   
:0044BD7C 7424                    je 0044BDA2   
  
４、最后修改程序   
将 E8 36 B2 FB FF 改为 90 90 90 90 90   
  E8 32 A5 FB FF 改为 90 90 90 90 90   
如果愿意可将未注册信息一并改掉。程序简单破解也非常简单，初学者可以练练手。  

 我用的是 softice TRW2000 我没试，将下面的文件删除就不会再出过期信息了应该也能拦到了 bpx WritePrivateProfileStringA   
\\windows\\java\\packages\\data\\Giadir6c.dat   
原理就是不让它写过期标志   4dd466d0e1e0cc123afbe504da2462ca