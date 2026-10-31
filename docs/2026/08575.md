
# [原创]检测虚拟化第二弹——cr access

天字号杀手第0x114514名：利用某些坑爹 hypervisor 处理 `cr access` 时的缺陷。

## 原理

真机环境下，根据文档：

> “`CR0`寄存器如果 `PG=1`、`PE=0`，此时由于不在保护模式下不能启用分页机制，因此处理器会产生一个一般保护异常，即这种标志组合无效；”

我们设置 `CR0.PG=1`、`CR0.PE=0`，然后 `writecr0`，可以得到一个 `#GP` 异常。

如果是某些不完善的 hypervisor（比如 hyperplatform、ksm 等），由于缺乏对 `CR0`寄存器合法性的检查，会让这种组合的 `CR0` 成功写入 `GuestCr0`，从而导致 guest vmentry 失败。

## 注意事项

1. 因为触发了 vmentry failure，容易把 guest os 搞蓝屏。

## 备注

ps：当然，利用其他 `cr` 寄存器特性，和真机触发异常、虚拟机不异常的情况，可以做到不蓝屏检测。

pss：大数字厂的 `hvm64` 居然也能被这种方法检测到，实属给力嗷。

## 代码

没有，so that big hands can not copy-paste。

## 截图

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-5-24 17:45 被 hzqst 编辑，原因：
