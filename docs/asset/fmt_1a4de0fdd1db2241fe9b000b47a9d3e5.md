
# 内存补丁与调试调试笔记

## 汇编指令修改记录

以下是针对 `nt` 模块中相关函数的汇编指令修改详情：

### 第 8 项：`nt!DbgkExitThread`
```
805c9610 399fbc000000    cmp     dword ptr [edi+0BCh],ebx
```
修改为：
```
399f74000000        {0x39,0x9f,0x74,0x00,0x00,0x00}
```

关联地址：
```
8  nt!DbgkExitThread+0x26            8063b42a
```

相关指令：
```
8063b441 f6804802000004  test    byte ptr [eax+248h],4
```
```
8063b448 7551            jne     nt!DbgkExitThread+0x71 (8063b49b)
```
```
8063b44a 8b89bc000000    mov     ecx,dword ptr [ecx+0BCh]
```
修改为：
```
8b8974000000      {0x8b,0x89,0x74,0x00,0x00,0x00}
```

### 第 9 项：`nt!KiDispatchException`
关联地址：
```
9  nt!KiDispatchException+0x18d          804fd94e
```

相关指令：
```
804fdacc 64a124010000    mov     eax,dword ptr fs:[00000124h]
```
```
804fdad2 8b4044          mov     eax,dword ptr [eax+44h]
```
```
804fdad5 39b8bc000000    cmp     dword ptr [eax+0BCh],edi
```
修改为：
```
39b874000000      {0x39,0xb8,0x74,0x00,0x00,0x00}
```

### 第 10 项：`nt!DbgkpCloseObject`
关联地址：
```
10 nt!DbgkpCloseObject+0x3e:
```

相关指令：
```
8063a7bd 8b5d08          mov     ebx,dword ptr [ebp+8]
```
```
8063a7c0 81c3bc000000    add     ebx,0BCh
```
修改为：
```
81c374000000        {0x81,0xc3,0x74,0x00,0x00,0x00}
```

## API 批量替换策略

然后是修改相关 API。我没有一个一个找特征码，我怕漏了，我直接计算了 `KernelBase` 和 `KernelSize` 比如 `0x804d8000`, `0x1f000000`, 我就在这区间里面直接替换。比如第 10 个 API `nt!DbgkpCloseObject` 我直接搜索区间里面所有的 `81c3bc000000` 替换为 `81c378000000`, 其他同理。

## 恢复 `KiAttachProcess`

最后是恢复 `KiAttachProcess`。我查看的时候这里没有特殊的保护，所以我在工具初始化的时候保存了原始地址，然后直接恢复了该地址。

## 工具下载

工具的下载地址（目前 `xp` 下能用）：

`bk_ssdtview.rar`

## 现状与求助

结果是：我已经能用 `windbg` 附加 `DNF`，但是还是收不到调试消息，我查看了 `debugport` 已经成功转移到了 `eprocess+0x78` 并且未被清，所有的 API 也已经使用了该地址，但是 `windbg` 还是显示挂起 30 秒那个提示，请各位给点提示！

随便请那位大哥给个邀请码，感谢！
