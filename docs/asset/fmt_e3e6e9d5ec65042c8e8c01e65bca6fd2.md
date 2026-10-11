
:0041A4F5 `90` `nop`

:0041A4F6 `90` `nop`

:0041A4F7 `90` `nop`

我们发现这里只有两行代码就是使 `eax` 为 `0`，而且还有很多地方要调用这个 `Call`，如果这里 `eax` 返回的是 `1` 的话，我们的目的就完成了，好在 `0041A4F3` 后面有好几个 `nop` 可以给我们用。

将上面代码简单修改如下就可以了：

```
:0041A4F0 33C0 xor eax, eax
:0041A4F2 40 inc eax
:0041A4F3 c3 ret
```

现在再重新启动 `ACDSee` 看看，是不是已经变成注册版了，还有一点不尽人意的地方就是打开 `About ACDSee...` 菜单时，里面的 “The software is license to:” 和 “license number:” 还是空的。要让它在这显示你的名字也很简单，用 `RegMon` 检测软件对注册表的读写，我们可以找到如下的地方，写个注册文件运行一下就可以了，至此 `ACDSee V3.1` 就彻底搞定了。

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\ACD Systems\ACDInTouch]
"UserName"="liangs"
"CompanyName"="ZTE"
"UserEMail"="liangs@wx88.net"

[HKEY_LOCAL_MACHINE\Software\ACD Systems\ACDSee]
"LicenseNumber"="ZTE389"

[HKEY_CURRENT_USER\Software\ACD Systems\ACDInTouch]
"UserName"="liangs"
```
