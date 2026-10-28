
(6) 感染后的标志为 `"WhBoy"` + 被感染程序名 + `".exe"`

![](https://bbs.kanxue.com/upload/attach/201802/559788_MWG6G4ATCT5SMX9.jpg)

![](https://bbs.kanxue.com/upload/attach/201802/559788_AG8NEU4Y4TQFB9G.jpg)

# 0x15 感染后程序的运行

(1) <感染后程序> 的结构,

![](https://bbs.kanxue.com/upload/attach/201802/559788_GGRGCQUUUU5QZNF.jpg)

![](https://bbs.kanxue.com/upload/attach/201802/559788_4K2HYFNT2TVFMHH.jpg)

(2) 判断是否被感染, 感染则跳转

![](https://bbs.kanxue.com/upload/attach/201802/559788_623BN3E5KC5KRXQ.jpg)

(3) 创建 自身进程名 + `".exe"`文件

![](https://bbs.kanxue.com/upload/attach/201802/559788_6ZJ3XGCTC8CY8V3.jpg)

![](https://bbs.kanxue.com/upload/attach/201802/559788_WWFXFVVDNRBGM7E.jpg)

(4) 从自身中提取 <源程序> 并写到 `"BeInfected.exe.exe"`

![](https://bbs.kanxue.com/upload/attach/201802/559788_28KRTKKGV5372GW.jpg)

![](https://bbs.kanxue.com/upload/attach/201802/559788_BQ3DMWPJTVP8YP8.jpg)

(5) 在`Temp`目录 写批处理 并 运行批处理

![](https://bbs.kanxue.com/upload/attach/201802/559788_YC2KFRM649787AE.jpg)

批处理内容:

```bat
//循环删除自身, 直至删除成功
:try1
del "C:\Documents and Settings\Administrator\桌面\2222\BeInfected.exe"
if exist "C:\Documents and Settings\Administrator\桌面\2222\BeInfected.exe" goto try1
//将 BeInfected.exe.exe 修改为 BeInfected.exe, 并运行 BeInfected.exe.
ren "C:\Documents and Settings\Administrator\桌面\2222\BeInfected.exe.exe" "BeInfected.exe"
if exist "C:\Documents and Settings\Administrator\桌面\2222\BeInfected.exe.exe" goto try2
"C:\Documents and Settings\Administrator\桌面\2222\BeInfected.exe"
:try2
del %0
```

执行生成的批处理

![](https://bbs.kanxue.com/upload/attach/201802/559788_D2ABGYXK49AF6XK.jpg)

(6) <感染后程序> 就被还原成 <源程序> 了, 并运行 <源程序>.

![](https://bbs.kanxue.com/upload/attach/201802/559788_UXRM5T2BRA5F7E9.jpg)

(1) 有什么错误请大神们指出.

(2) 并感谢15PB老师们的帮助和教导.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
