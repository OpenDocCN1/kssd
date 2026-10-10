

# Winzip6.0

> 原文：[`www.pediy.com/kssd/pediy03/forum560.htm`](https://www.pediy.com/kssd/pediy03/forum560.htm)

winzip6.0 去窗小记   
  
\------cracked by kobe   
  
winzip6.0 这个软件是老早就破解了的，注册机制很简单，想了解的朋友请翻阅看雪的教程或其他的 CRACK 站点，我就不再重新破解它，并写破解过程了！破解之后它总是 show 出一个窗口大意是：该版本较旧，请下载更新的版本，云云。。   
  
当初想破解完全一点，可没那份心，今天心情不好，SO，看见哪个窗口，烦，于是   
  
就屏蔽掉 show 窗口的 call 就 ok 了！这次的破解很简单，只用 soft-ice 和 hexworkshop 就完成了，入门级别，给初学者看吧。   
  
断点用 bpx createwindowex，别的什么 bpx messagebox，bpx createwindow..拦不下！   
  
拦下后 bc 掉断点，按 5 下 F12 就 game over 了！于是从 4 下 f12 后开始追了！按 F10 一路   
  
step 下去，按呀按。。。。又 GAMEOVER 了，，，噫，刚才 CALL 的地址忘看了，好在带过之后也能别单步 STEP 拦下，看上面的地址:0257:00438c30 call XXXXXXXXX;(code on 看机器码:e8cb9efcff)   
  
yeah!!!!!下次再重新执行，动态改为 5 个 NOP，果然成功了！   
  
于是后面的是就发生了，用 hexworkshop 找 e8cb9efcff 改为 9090909090，存盘.   
  
至此那个讨厌的窗口就不会出现了！很 EASY，才化了 5 分钟就 kill 掉它了！   
  
  
\--------------------------------------------------------------------------------   
  
欢迎转载请不要改动信息(C)kobe 工作室   
  
http://kobesworld.yeah.net   
  
warmwaters@21cn.com  4dd466d0e1e0cc123afbe504da2462ca