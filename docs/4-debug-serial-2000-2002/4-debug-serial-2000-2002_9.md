

# animagic32

> 原文：[`www.pediy.com/kssd/pediy01/forum042.htm`](https://www.pediy.com/kssd/pediy01/forum042.htm)

animagic32   
  
介绍:此软件在<<软件世界>>光盘中,制作网页动画的软件   
开始用 bpx hmemcpy 中断然后按 F12 直到出现错误画面的前一次.   
找到 CMP \[484FC1\],00   
    jnz \*\*\*\*\*\*\*   
直接改此跳转会出现注册成功画面,但选择 About 菜单或退出重新启动后   
仍有未注册提示画面.   
追踪过程中,若修改上面跳转后,程序执行了 WritePrivateProFileStringA   
发现是将用户名,公司名,注册码写入到 animagic.ini   
综合上面两点,推测程序启动时要读此 ini 文件,比较后置\[00484FC1\]   
因此设 bpm 484FC1 后启动程序,找到   
call \*\*\*\*\*\*   
mov \[00484FC1\],al   
注册码的比较一定在 CALL 中,在 CALL 上设中断,重新启动程序被中断后,按 F8   
进入 CALL,里面还有一个 CALL,按 F8 进入(因为后面没两句就 ret 了)   
找到了计算过程(原程序用 W32dasm 存下的程序太大,此段没能拷贝过来)   
但其过程如下:   
举例说明:   
用户名:Killer   
将 ANIMAGIC16 和用户名的 ASCII 码相加   
ASCII(ANIMAFIC16)=2A0   
ASCII(转为大写字母(Killer))=263   
相加后等于 463   
463\*463=133E49   
133E49\*16=1A75A46   
1A75A46/7=3C7A9C   
3C7A9C\*463=94DDA54   
94DDA54-21=94DDA33   
  
用户名:Killer   
注册码:94DDA33   
  
另外此程序还有一个注册码:   
将 GIFOPT32,用户名和公司名的 ASCII 码相加   
ASCII(ANIMAFIC16)=2A0   
ASCII(转为大写字母(Killer))=263   
ASCII(转为大写字母(Home))=129   
相加后等于 58C   
58C\*58C=1EC490   
1EC490\*16=2A4E460   
2A4E460/7=60B2E9   
60B2E9\*58C=1860646C   
1860646C-21=1860644B   
  
用户名:Killer   
公司名:HOME   
注册码:94DDA33   
  
程序根据两个字符串 ANIMAGIC16 和 GIFOPT32 计算注册码   
但不知两个注册码有何区别.