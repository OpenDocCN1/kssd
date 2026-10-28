

# 分享：发一个 Win7 下获得 TrustedInstaller 权限完全控制的补丁

我最近使用 Win7，发现即使拥有 Administrator 的密码，也不能完全控制电脑。

什么世道啊，还有一个 TrustedInstaller 比 Administrator 的权限都要高。

网上找了个补丁，说是能获得全部权限。只因为下载这个补丁，我费了很多周折。

所以发出来，希望大家以后再使用这个方法的时候可以节省时间。

![](http://bbs.win7c.com/attachments/month_0908/0908301314816c1567e47b69c9.jpg)

![](http://bbs.win7c.com/attachments/month_0908/090830131438ff2c8dc6b7f472.jpg)

![](http://bbs.win7c.com/attachments/month_0908/09083013142bace33cdf9876e7.jpg)

## 导入 `.reg`

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\runas]

@="获取权限"

"NoWorkingDirectory"=""

[HKEY_CLASSES_ROOT\*\shell\runas\command]

@="cmd.exe /c takeown /f \"%1\" && icacls \"%1\" /grant administrators:F"

"IsolatedCommand"="cmd.exe /c takeown /f \"%1\" && icacls \"%1\" /grant administrators:F"

[HKEY_CLASSES_ROOT\Directory\shell\runas]

@="获取权限"

"NoWorkingDirectory"=""

[HKEY_CLASSES_ROOT\Directory\shell\runas\command]

@="cmd.exe /c takeown /f \"%1\" /r /d y && icacls \"%1\" /grant administrators:F /t"

"IsolatedCommand"="cmd.exe /c takeown /f \"%1\" /r /d y && icacls \"%1\" /grant administrators:F /t"
```

## 删除 `.reg`

```reg
 i n d o w s   R e g i s t r y   E d i t o r   V e r s i o n   5 . 0 0

[ - H K E Y _ C L A S S E S _ R O O T \ * \ s h e l l \ r u n a s ]

[ - H K E Y _ C L A S S E S _ R O O T \ D i r e c t o r y \ s h e l l \ r u n a s ]
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

## 上传的附件

- [win7获得完全控制权软件.rar](javascript:void(0);)
  - （0.52kb，87 次下载）

