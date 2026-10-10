

# 超级解霸 2000

> 原文：[`www.pediy.com/kssd/pediy02/forum208.htm`](https://www.pediy.com/kssd/pediy02/forum208.htm)

 \*豪杰超级解霸 2000 最新补丁破解   
  
setup.exe-->6,700,544 bytes (选自数码时代)   
  
    此补丁修补超级解霸 2000 的一些问题和增加多项功能，包含了新的卸载程序、新的可视音乐插件——超级跳舞机、网络视频 MPC 播放器、快速 Http 下载器、支持在 Internet 中直接播放 MPC、MP3 和 MPG 文件、与 IE 进一步融合在一起。同时，修正了在多数笔记本中（如东芝、宏基等等品牌）播放 VCD 出现非法操作的问题、Direct 方式播放 CD、WIN97 中播放 MP3 无法测试文件类型等等问题。   
  
    我是在《豪杰超级解霸》（吉林.03 测试版）基础上升级的   
  
梁先生的比较核心没变,很好解,我只把结果贴上好了,有兴趣的话自己试试吧:)   
  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Setup   
  
次数 RunTime(二进制值)------->RunTime 值为 0 的话退出   
  
STHSVCD.exe ->747,520 bytes   
  
017f:00415bc2 jnz 00415be6 ->jmp 00415be6   
  
查找:85C07522E817B2FFFF   
改为:85C0eb22E817B2FFFF   
  
Mmxado.exe ->380,928 bytes   
  
017f:0040e867 jnz 0040e89e ->jmp 0040e89e   
  
查找:cc35E8B2E7FFFF(跟到的是 7535E8B2E7FFFF why? )   
改为:eb35E8B2E7FFFF   
  
时间 InstallTime(二进制值)   
  
STHSVCD.exe   
  
017f:00415b97 jbe 00415bbb->jmp 00415bbb   
  
查找:3BCA7622   
改为:3BCAeb22   
  
Mmxado.exe   
  
017f:0040e85a ja 0040e869->nop   
  
查找:89442410770D   
改为:894424109090   
  
没有经过安全测试,有什么副作用未知,自己看着办吧:(   
  
完活:)   
  
  
  
\*Paint Shop Pro 6.02 无限期使用   
  
psp602ev.exe-------->14,929,800 bytes(选自数码时代)   
  
psp.exe   
  
hkey\_local\_machine\\software\\classes\\clsid\\   
{ADBB2D80-5DFD-11D2-A00C-00104B65AAF5}\\ms   
ipid6au=dword:x(x 值需要小于 30,超过 30 的话过期)   
  
  
017f:00771cbd add esp,18   
017f:00771cc0 cmp esi,1e(比较是不是 30,小于等于跳转)   
017f:00771cc3 jle 00771d0c ->jmp 00771d0c   
.   
.   
.   
  
017f:00771d6e add esp,04   
017f:00771d71 cmp esi,eax   
017f:00771d73 jle 00771e03 ->sorry,不记得了   
.   
.   
.   
017f:00771eb7 add esp,04   
017f:00771eba cmp ecx,eax   
017f:00771ebc jle 00771edc ->jmp 00771edc   
  
  
psp.exe   
  
查找:83 FE 1E 7E 47   
改为:83 FE 1E eb 47   
  
查找:3B F0 0F 8E 8A 00 00 00   
改为:3B F0 e9 8b 00 00 00 90   
  
查找:3B C8 7E 1E   
改为:3B C8 eb 1E   
  
anim.exe   
  
hkey\_local\_machine\\software\\classes\\clsid   
{64c53720-b69e-11d2-8728-00104b141a76}\\ms   
ipid6au=dword:x(x 值需要小于 30,超过 30 的话过期)   
  
017f:005085f5 add esp,18   
017f:005085f8 cmp esi,1e(比较是不是 30,小于等于跳转)   
017f:005085fb jle 00508643 ->jmp 00508643   
.   
.   
.   
017f:005086a9 add esp,04   
017f:005086ac cmp esi,eax   
017f:005086ae jle 00508706 ->jmp 00508706   
  
anim.exe   
  
查找:83 fe 1e 7e 46   
改为:83 fe 1e eb 46   
  
查找:3b f0 7e 56   
改为:3b f0 eb 56   
  
无任何副作用,只不过进去时需点一下 start,最新 psp7.0 beta3 华军有下载   
  
说明:两个软件大同小异,所以写一起了,由于时间限制的软件比较好解,我偷懒不写流程了,比较核心都写在上面了,改的时候注意文件要对应,不同的话也找不到.   
  
cracked by 电神魔鬼   
thanks for your supports   4dd466d0e1e0cc123afbe504da2462ca