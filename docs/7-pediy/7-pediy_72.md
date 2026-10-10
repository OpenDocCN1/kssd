

# regsnap2.8(build 638)

> 原文：[`www.pediy.com/kssd/pediy03/forum610.htm`](https://www.pediy.com/kssd/pediy03/forum610.htm)

小弟初学破解，请多指教   
前些天买了张电脑报二○○○年合订本配套光盘，里面有个注册表监视软件 Regsnap 2.8 (build 638)   
小弟水平太菜，找不到注册码，暴力破解时发现其对本身加密，只要改动就会拒绝执行，用 trw2000 调试时发现   
只要把 1->\[004249f0\] 就可以解除时间限制。于是用 Rpp 做了个内存补丁，其脚本文件如下：   
  
O=Load\_regsnap.exe:   
  
F=regsnap.exe: ; 要补丁的程序   
  
P=4249f0/00/01:    
  
p=40463b/74/75:   
  
p=411401/74,5b/90,90:   
  
$ ; 结束   
  
用 rpp.exe 生成文件 Load\_regsnap.exe 后将其拷到 regsnap 目录下执行，一切 ok  4dd466d0e1e0cc123afbe504da2462ca