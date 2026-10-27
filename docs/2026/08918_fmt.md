
# [原创]整理flexlm_v9.2_full_source不完全编译手册

下载`flexlm_v9.2_full_source`后，编译好久才把demo编译过去，考虑到可能会有同样辛苦的同志，所以把编译过程记录下来共享：

1) 解压后从dos窗口进入`v9.2`目录下运行环境变量`vc.bat`（在`vc`的安装目录`VC98\Bin`下）

2) 修改`pc.mak`中27行为`CERTDIR = certicom\lib\$(FLEXLM_MAKE_PLATFORM)\`

3) 注释第28行 `#CERTDIR = $(CERTICOM)`

4) 在`src\pc.mak`的490行下面添加一行

```
copy ..\flexlm-java-tng\c\i86_n3\lGetNativeHostId.obj .
```

5) 进入`utils`目录下面运行

```
lmrand1.exe -seed
```

生成3个随机种子，覆盖`h\lm_code.h`中对应宏定义

6) 运行`lmkg`，生成以`vonder name`为`demo`的`vonder key`值，覆盖`h\lm_code.h`中对应宏定义

7) `src\l_ctype.c`替换`?`为`*`（注意是注意其中是注释符号`/* */`匹配，不涉及注释匹配的不要修改）

8) `copy certicom\lib\i86_n3\include\*. * h /y`

9) `copy certicom\lib\i86_n3\lib\*. * i86_n3 /y`

10) `copy flexlm-java-tng\c\i86_n3\*. * src /y`

11) 编译设置环境变量

a) `cl makesetup.c`

b) `makesetup RELEASE NONDEBUG i86_n3 MT`

c) `setup.bat`

12) 编译SDK

```
nmake /fpc.mak release
```

看到`[Finished building kit! Don't forget to build lmtools utility.]`表示编译完全通过

13) 生成license

```
cd i86_n3
lmcrypt.exe counted.lic
lmcrypt.exe uncounted.lic
lmcrypt.exe expired.lic
```

14) 测试运行

`lmtools.exe`配置一个`license server`。然后运行`lmwin.exe`将所有的`feature`都`checkout`一次

## 附:vondor_key1-5与lm_seed1-3

```
#define VENDOR_KEY1 0x4054cac1
#define VENDOR_KEY2 0x4500538e
#define VENDOR_KEY3 0x542a6830
#define VENDOR_KEY4 0x3951f9f9
#define VENDOR_KEY5 0x0b165dca
```

```
/*
*         Vendor name.                  Leave "demo" if evaluating.  Otherwise,
*                                 set to your vendor daemon name.
*/
#define VENDOR_NAME "demo"
```

```
/*
*         Private SEEDs:                 Make up 3, 8-hex-char numbers, replace and
*                                guard securely.  You can also use the command
*                                'lmrand1 -seed' to make these numbers up
*/
#define LM_SEED1 0x82395728
#define LM_SEED2 0xac86853f
#define LM_SEED3 0x13765cfb
```

注：说是不完全编译是因为还有一些问题没有解决，`genlic32`一直运行不起来正在查找资料

参考不少朋友的文章，没有记下来，再次不提名感谢:)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
