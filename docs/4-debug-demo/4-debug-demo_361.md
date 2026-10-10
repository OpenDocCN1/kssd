

# 欧阳电子商务信使 V6.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1066.htm`](https://www.pediy.com/kssd/pediy03/forum1066.htm)

欧阳电子商务信使        (oymailman) V6.0   
http://www.skycn.com/down1.php?id=5158&url=http://202.107.104.34/soft/down/oymailman.exe   
  
通过网络向目标客户发送商业资讯电邮或电子杂志，可明送邮件也支持 BCC 高速暗送和多线程发送，收件人看不到别人的邮件地址。支持附件发送，列表发送中断后可保存断点以便续发。目标客户列表电邮数量不受限制，采用高速发送每小时可发送电邮达 100000 封。邮件信体支持文本文件和网页两种格式。只需填入您的网站或网页地址，软件自动完成超文本编码发送，让您的网站访问量以几何数字增长。您可以选择中文或西文字符编码，以满足不同的国内外客户阅读需要。一款不可多得的邮件高速群发软体。   
  
这个软件好象应该是 keyfile 保护的。我把在破解这个软件的过程详细的写出来了，希望出学者能看看，其实也很简单的，我搞了 3 天是因为自己经验不够，呵呵，下次不会了，   
  
工具：soft-ice 4.05 自己用的话就这一个工具就够了，如果要发布就还需要 w32dasm 以及 hiew 还有就是脱壳的 unpack 呵呵， 我手动不会，只好自动了   
  
开启软件，点击电子注册，输入 copyyour（自己喜欢）然后唤出 soft-ice 就是 ctrl+d 然后下断点 bpx hmemcpy(最常用的。我也只知道这一个:-( ) 然后就要辛苦的按 F12 了，一共是 58 次（59 次的时候会出现出错框）这是好来到下面这些地方，我慢慢地给大家说说，（等等，先喝一口百事可乐，不干喝可口可乐了:-( )   
  
:004B4299 3C01 cmp al, 01 =====>按 58 次 f12 后来到这里，按 F10 向下跟踪   
:004B429B 0F8528010000 jne 004B43C9   
:004B42A1 8B45FC mov eax, dword ptr \[ebp\-04\]   
:004B42A4 BA30444B00 mov edx, 004B4430 ========这里打 d eax 将会看见我输入的 copyyour   
:004B42A9 E8B6FDF4FF call 00404064 ========注意这里★ 软件给出一个假的注册码 oymailmanxxxxxx，你用假注册码输入后会跳出“请联系代理商，购买正版软件” 下 d edx 还将看到软件要检查 目录下面是否有 oymailman.key 这个文件 关键哟！！   
:004B42AE 0F8500010000 jne 004B43B4 =======没有 oymialman.key 就跳到后面的 call 也是弹出“请联系代理商，购买正版软件” 值得注意的是，软件每次启动都会进行检查，如果你没有这个 key 即使你改了跳，都会出现“请联系代理商，购买正版软件”而且软件不能启动！！   
:004B42B4 8D55F4 lea edx, dword ptr \[ebp\-0C\]   
:004B42B7 A1E8854B00 mov eax, dword ptr \[004B85E8\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B4243(C)   
|   
:004B42BC 8B00 mov eax, dword ptr \[eax\]   
:004B42BE E86994F9FF call 0044D72C   
:004B42C3 8B45F4 mov eax, dword ptr \[ebp\-0C\]   
:004B42C6 8D55F8 lea edx, dword ptr \[ebp\-08\]   
:004B42C9 E8C651F5FF call 00409494   
:004B42CE 8B55F8 mov edx, dword ptr \[ebp\-08\]   
:004B42D1 B8889B4B00 mov eax, 004B9B88   
:004B42D6 B954444B00 mov ecx, 004B4454   
:004B42DB E8C0FCF4FF call 00403FA0   
:004B42E0 A1889B4B00 mov eax, dword ptr \[004B9B88\]   
:004B42E5 E80251F5FF call 004093EC   
:004B42EA 84C0 test al, al   
:004B42EC 0F849C000000 je 004B438E =======比较是否是正确的注册码 可能在上面的一个 call 里面进行计算，我没有追了   
:004B42F2 6A00 push 00000000   
:004B42F4 668B0D64444B00 mov cx, word ptr \[004B4464\]   
:004B42FB B202 mov dl, 02   
:004B42FD B870444B00 mov eax, 004B4470   
:004B4302 E8EDE4F9FF call 004527F4 ======正确注册！！！！   
  
  
好了，代码分析到此结束，我们开始给这个软件动手术了！   
  
首先要制作一个 oymailman.key 怎样制作？？我很苯的，是在 dos 下面用一个文本文件改的 唉 c:\\rename 1.txt oymailman.key 就这样了   
  
然后放到 软件目录里面去了，再次重复刚才的分析过程   
  
:004B42AE 0F8500010000 jne 004B43B4 在这里执行 r fl z 呵呵，这是 kanker 大哥教我的 好象对 jnz jz 等等都有很好的疗效！！   
  
这样改了以后就一直 f10 或者直接 g 出来，注册成功了！！哈哈再次启动软件 ，软件的注册栏就变成灰色的了   
哈哈，   
  
这就是一个简单的 keyfile 保护吗？   
我也不清楚，呵呵。谢谢你的观看，这个软件的其他系列同理可解 要听 copyyour 的下次破程（破解过程）请给我一点掌声！呵呵，谢谢你！   
  
  
cracked by copyyour\[CCG\]   
cracked by copyyour\[BCG\]   
好想进入\[FCG\]不知道有什么条件？  4dd466d0e1e0cc123afbe504da2462ca