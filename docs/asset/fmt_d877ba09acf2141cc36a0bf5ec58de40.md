
1D284B5C `E9 0A000000` jmp 1D284B6B  
1D284B61 `037C24 18` add edi, dword ptr ss:[esp+18]  
1D284B65 `BF 2E9E4900` mov edi, 499E2E  
1D284B6A `C3` retn  
1D284B6B `5E` pop esi  
1D284B6C `5F` pop edi  
1D284B6D `EB 7B` jmp short 1D284BEA  

修改 `1D284B3D` 处代码，必须要跳转。运行，正常了。

下面去除时间限制，`CTRL+F2` 重新载入，到上面修改处继续跟踪，来到下面。

1D284C0B `E8 00D6FBFF` call 1D242210  
1D284C10 `E8 EBD6FBFF` call 1D242300  
1D284C15 `E9 12000000` jmp 1D284C2C  

改为

1D284C0B `E8 00D6FBFF` call 1D242210 ;全部改为 90  
1D284C10 `E8 EBD6FBFF` call 1D242300 ;全部改为 90  
1D284C15 `EB 24` jmp short 1D284C3B  

运行，时间提示没有了，保存，退出。

修改系统时间到过期时间以后，运行 `bds2006` 验证，试一下几个功能，OK，时间限制去除了。

## 七、后记

本来是不想写这篇文章的，第一次手工脱壳，水平真的是太菜，不想出来献丑了，权当是凑发帖数吧！文章里面基本没有自己的东西，`patch` 代码也是抄别人的，在这里要感谢论坛里的牛人们，无数牛人的文章使我受益良多。

`4dd466d0e1e0cc123afbe504da2462ca`
