
# [原创][分享]Dex动态加载关键函数调用链

- 写在前面：最近实在是太忙了，本来说好上个周上完课之后就发篇帖子的，但是手头上一直有事，就一直没发，今天稍微整理了一下也不写具体细节了，就只是简单的分享几张我整理的图片吧，具体流程还是看源码吧。(由于论坛md上传图片原因，详细高清图见附件）

## Android 4.4下Dex动态加载

在Android4.4之前，Android是基于Dalvik虚拟机实现的，如果想要实现动态的dex的加载，正常来说就得借助`DexClassLoader`，`DexClassLoader`加载dex的具体流程，不再赘述，见附件图

![](upload/attach/202003/715334_AZMSBJKH38XJT86.png)

## Android 8.0下Dex动态加载

在Android4.4开始，Android引入Art虚拟机，大大加快了Android应用的运行速度，此时Dex的动态加载主要还是通过`DexClassLoader`进行加载，只是so层基于Art重写了，这里基于Android 8.0做了整理，注意！！！这个整理是基于禁用了`dex2oat`的流程，上图！

![](upload/attach/202003/715334_4YCF4WYAE2W3PU2.png)

在Android 8.0之后，Android又引入了`InMemoryDexClassLoader`机制，实现了直接从内存中加载dex的方式，这样也实现了Dex的动态加载，上图

![](upload/attach/202003/715334_TKB9D74PVVJ8EKF.png)

- 写在后面，以上所有图，都是上完@hanbingle师傅的课程后，自己看源码后整理的，如果有错误，欢迎指摘；最后在这里感谢@hanbingle师傅的指导和鼓励。

----------------------------更新实践部分-------------------

为了证实我的图中标注的脱壳点有效，拿了一个腾讯加固的例子

这里选取我之前总结的调用图中的`DexFileVerifier::Verify`和`DexFileVerifier`以及`OpenFile`函数与`DexFile::DexFile`函数作为脱壳点，

在函数中插入以下dump dex的代码

```c
DexFileVerifier(const DexFile* dex_file,
const uint8_t* begin,
size_t size,
const char* location,
bool verify_checksum)
: dex_file_(dex_file),
begin_(begin),
size_(size),
location_(location),
verify_checksum_(verify_checksum),
header_(&dex_file->GetHeader()),
ptr_(nullptr),
previous_item_(nullptr) {
// 以下是添加的代码
char dexfilepath[100]={0};
int pid=getpid();
sprintf(dexfilepath,"/sdcard/%d_%d_DexFileVerifier.dex",(int)(size),pid);
int fd=open(dexfilepath,O_CREAT|O_RDWR,0666);
if(fd>0){
write(fd,begin,size);
close(fd);
}
}
```

编译源码,刷机，

刷机成功后，安装并给予应用存储权限，然后运行app，并在`/sdcard`目录下使用`grep`命令过滤dex，最终得到脱壳后的文件如下

![](upload/attach/202003/715334_6R2K295Z6PZMXXU.jpg)

可以发现最终是可以脱下来的，有兴趣的可以测一测其他的加固，应当都是能脱下来的，测试的apk见

链接:`898K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4P5V1S2*7e0g2k6v1z5g2S2k6g2o6c8I4i4K6u0V1e0X3N6%4f1q4u0T1x3e0u0m8` 密码:`q7x4`

最后于 2020-3-27 10:10

被Simp1er编辑，原因：更新代码

上传的附件：

- [1.zip](javascript:void(0);)
  （2.83MB，117次下载）
