
# 记录一次编译内核修改内核源码监控系统调用

最近看了很多很好的贴子啊

SVC的TraceHook沙箱的实现&无痕Hook实现思路 https://bbs.pediy.com/thread-273160.htm

Android通用svc跟踪以及hook方案——Frida-Seccomp

https://bbs.pediy.com/thread-271815.htm

又菜又想搞svc，就只能做最简单无脑的了

## 一， 给小白介绍一下什么是系统调用(syscall)

简单的来说呢就是 `libc.so` 里面的一些系统函数比如 `read()`, `write()` 最简单的读写函数，这些函数实际上底层都是由 `syscall()` 交给 linux 内核去实现的

![](upload/attach/202206/928079_W9SBAFDY8PJU357.png)

![](upload/attach/202206/928079_QH8F9CQM6W6VZTP.png)

他的第一个参数是系统调用号，后面的参数是根据调用的函数来写的

现在用 android studio 开发的话使用系统调用和使用正常的函数一样也不需要去记这些系统调用号都有一个常量表可以很方便的去实现功能

![图片描述](upload/attach/202206/928079_72ZDR6PMXKN58Y6.png)

![图片描述](upload/attach/202206/928079_P6RY7NVFVSR3EJ5.png)

在调用 `syscall` 时通过会从用户态切换到内核态

![图片描述](upload/attach/202206/928079_4D76F37WPV3T7RU.png)

正常使用 hook 框架很难去 hook 到内核态下的代码 常规手段可以通过找到 `svc` 指令地址 在他调用 `svc` 之前去 `inlinehook` 他效率以及一些别的问题 导致局限性非常大

## 二， 编译内核步骤

1. 去谷歌的官方文档查看自己机型的内核源码的目录

我的是 Pixel 3a XL (bonito) 也就是 `bonito-kernel`

这里在谷歌官方下载的话太慢了我选择去中科大下载

命令：

```
git clone http://mirrors.ustc.edu.cn/aosp/kernel/msm.git
//下载完成后cd 到sms 目录 在链接手机在终端输入
cat /proc/version
Linux version 4.9.165-g583404d31612-ab5723792
//这一串583404d31612 这个是我们需要的
git branch -r --contains 583404d31612
//获取到我们需要的版本号
```

![图片描述](upload/attach/202206/928079_3G67M9T93RA3494.png)

检出了那么一大堆 随意挑选一个中意的去检出即可

![图片描述](upload/attach/202206/928079_BHDYBQMDSHHTHK3.png)

```
git checkout -b android-msm-bonito-4.9-android10 origin/android-msm-bonito-4.9-android10
```

下载完毕

![图片描述](upload/attach/202206/928079_9KGSZCQQ3ER4WZW.png)

接下来就是编译一下内核看看有没有问题了

因为编译内核有一大堆命令就搞个 sh 脚本来

```
#吧之前编译的内核清理掉 在msm下面操作啊 别再aosp 要不然直接清空了
make clean
# 设置编译平台为64位arm
export ARCH=arm64
export SUBARCH=arm64
# 配置arm64的交叉编译工具路径
export PATH=/home/用户名/bin/aosp1000r2/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin:$PATH
# 配置arm32的交叉编译工具路径
export PATH=/home/用户名/bin/aosp1000r2/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin:$PATH
# 设置64位交叉编译工具前缀，上述64位工具路径中的公共前缀，以CROSS_COMPILEgcc为准
export CROSS_COMPILE=aarch64-linux-android-
# 设置32位交叉编译工具前缀，上述32位工具路径中的公共前缀，以CROSS_COMPILEgcc为准
export CROSS_COMPILE_ARM32=arm-linux-androideabi-
# 生成编译配置文件，O=out指定输出目录
make O=out marlin_defconfig
# 执行内核编译 根据自己设置的cup核心
make -j18 O=out ARCH=arm64
# 复制内核到marlin-kernel目录
cp -f out/arch/arm64/boot/Image.lz4-dtb ../device/google/marlin-kernel/;
```

写好 sh 脚本后 改一下给个 777 权限 运行就好了

```
chmod 777 tutu.sh
./tutu.sh
//然后直接在aops目录 编译完成后刷机即可
make -j16
```

接下来就是修改内核源码了

![图片描述](upload/attach/202206/928079_VQYRVG685WNY48V.png)

可以看到这些函数实际上真正底层是在内核实现的

![图片描述](upload/attach/202206/928079_UD3AUGBG2642GNM.png)

这里只搞一个做演示哈 这里是 `oepn` 内核的源码 我们可以在这里动一些手脚

```
long do_sys_open(int dfd, const char __user *filename, int flags, umode_t mode)
{
struct open_flags op;
int fd = build_open_flags(flags, mode, &op);
struct filename *tmp;
//add
const struct cred *cred = current_cred();
kuid_t isuid = cred->uid;
int pid = current->pid;
int isuid = uid.val;
//过滤系统app
if(isuid > 10000) {
char bufname[256]={0};
strncpy_from_user(bufname, filename, 255);
/*bufname 就是监控到的一些调用syscall的一些读文件操作当然也可以做io重定向去判断某些特定的值写一个系统app去监听这里过滤替换的一些操作这里为了简单学习实现先这样了*/
printk("do_sys_open : %s uid:%d pid:%d\n",
bufname, isuid, pid);
}
//add end
if (fd)
return fd;
tmp = getname(filename);
if (IS_ERR(tmp))
return PTR_ERR(tmp);
//得到一个文件描述符
fd = get_unused_fd_flags(flags);
if (fd >= 0) {
//得到一个struct file结构
struct file *f = do_filp_open(dfd, tmp, &op);
if (IS_ERR(f)) {
put_unused_fd(fd);
fd = PTR_ERR(f);
} else {
fsnotify_open(f);
//把文件描述符和struct file结构关联起来
fd_install(fd, f);
}
}
putname(tmp);
return fd;
}
```

修改完毕后直接使用之前写的 sh 脚本在 msm 编译内核即可

```
./tutu.sh
```

大佬的 Linux kernel 笔记
