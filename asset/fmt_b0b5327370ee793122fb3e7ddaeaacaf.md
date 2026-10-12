
# 利用 Flex 10.1 SDK 开发某软件注册机

> 原文：[`www.pediy.com/kssd/pediy12/136910.html`](https://www.pediy.com/kssd/pediy12/136910.html)

【软件名称】利用 Flex 10.1 SDK 开发某软件注册机。

【应用平台】WinXP SP4

【作者邮箱】chubing6143@sina.com

【使用工具】OllyDbg1.10，FlexLM 10.1SDK

【软件限制】FlexLM

好长时间不破解了，也好长时间没有到看雪来了。

最近因为需要帮人解决一个软件远程桌面调用的问题，该软件我以前用 Flexlm 7.2SDK 写的注册机，但是由于 FlexLM 低版本的 SDK 不支持远程桌面使用方式，即 License 文件中不支持 `TS_OK`，因此，需要利用高版本的 SDK 重新开发 NI 通用注册机。因此，我就又操起了这一行了，好长时间感觉没有投入破解了，还是怀念当初学习破解的过程，当然由于自己工作的原因，没有坚持下来，因此，连二把刀都不算。写这篇文章只是为了给我这样的初学者提供帮助。考虑到论坛的需要，就不公布是什么软件了。为了给初学者尽可能多的帮助，我将过程记录的尽可能详细，高手飘过。

### 一、安装 SDK

首先安装 `FLEXnetLicensing.exe`（10.1 的 SDK）。

### 二、生成 Key

利用 CrackZ 上下载的 Flexvkg3（`lmkg.exe`）自动生成 Vendor 为 `*****`（怎样找 Vendor，论坛上介绍的太多了，不叙述），Version 为 `10` 的 Key 如下：

```
/* Version 10 keys */

#define VENDOR_KEY1 0x0218a4**

#define VENDOR_KEY2 0xa52467**

#define VENDOR_KEY3 0x185a43**

#define VENDOR_KEY4 0xfa721f**

#define VENDOR_KEY5 0x011a5c**

#define TRL_KEY1 0x31f528**

#define TRL_KEY2 0x4cc898**

#define VENDOR_NAME "*****"
```

然后编辑 `C:\Program Files\flexlm\v10.1\machind\lm_code.h` 文件，将其中的 `VENDOR_KEY`、`VENDOR_NAME` 与 `TRL_KEY` 替换成前面生成的宏定义。

### 三、配置 Makefile 与生成 Seed

将 `C:\Program Files\flexlm\v10.1\i86_n3\makefile` 文件中 `VENDORNAME = demo` 替换为 `VENDORNAME = nilm`

然后运行 `cmd`，进入命令行模式下执行 `C:\Program Files\flexlm\v10.1\i86_n3\lmrand1 seed`

会随机生成 `LM_SEED1`、`LM_SEED2`、`LM_SEED3` 的值，并生成 `lmseed.dat` 文件，将该文件中的 `LM_SEED` 值粘贴到 `lm_code.h` 中去。

### 四、粘贴加密代码

考虑到该软件加密是从 7.1 升级过来的，因此在 `lm_code.h` 中粘贴下面的代码：

```
#define LM_SIGN_LEVEL 1

#define ENCRYPTION_SEED1 0x**1b372e（需要对软件跟踪进行查找，为了安全起见，我隐去了）

#define ENCRYPTION_SEED2 0x**394f25（需要对软件跟踪进行查找，为了安全起见，我隐去了）

#define ENCRYPTION_SEED3 0x12315688 //任意值了

#define ENCRYPTION_SEED4 0x12315688 //任意值了
```

并将 `LM_STRENGTH` 定义为 `LM_STRENGTH_LICENSE_KEY`

```
#define LM_STRENGTH LM_STRENGTH_LICENSE_KEY
```

### 五、构建加密工具

此时命令行模式下执行 `C:\Program Files\flexlm\v10.1\i86_n3\build.bat` 会自动生成 `lmcrypt.exe`，该文件就可以将 license 文件自动完成加密的过程。

然而为了写注册机，还得进行下面的工作。

### 六、构建 DLL 与依赖

命令行模式下执行 `C:\Program Files\flexlm\v10.1\i86_n3\build DLL`

会自动生成 `lmgr10.dll` 与 `lmgr10.lib`，这两个文件是非常有用的，写注册机时需要链接这两个文件，写注册机时需要 SDK 中的以及 build 生成的以下文件：

`lm_attr.h`、`lm_code2.h`、`lm_code.h`、`lm_trl.h`、`lmclient.h`、`lmhostid.h`、`lmprikey.h`、`lmseeds.h`、`lmgr10.lib`、`lmgr10.dll`。

### 七、修改头文件与编写解密代码

为了在 VC 工程中能够编译通过，首先将 `lmprikey.h` 中的：

```
lm_extern int *l_prikey_sign(void) = 0;
```

修改为：

```
lm_extern int (*l_prikey_sign)(void) = 0;
```

然后在 Project Setting 中添加 PC 的宏定义。

写解密的关键代码函数如下：

```
LM_CODE_GEN_INIT(&site_code);//:初始化 Flex 变量，主要是 lm_code.h 中的变量

if (lc_init((LM_HANDLE *)0, VENDOR_NAME, &site_code, &lm_job))
{
  lc_perror(lm_job, "lc_init failed");
  exit(-1);
}

if (lc_cryptstr(lm_job, fileCrypt, &ofile_str, /*&v*/&site_code, forceit, "", 
      &err))//laowang :关键函数将 License 文件字符串内容加密求得正确的 License 文件
{
  MessageBox(NULL,"lc_cryptstr 运行错误，请联系作者！",filePath,MB_OK);
}
```

到此就完成了，如果有我等初学者对此感兴趣的，我可以考虑将基本的注册机代码发上来。
