
# [原创]【抛砖引玉，但是我是玉】一个可以通杀EAC-CR3保护的猜想

众所周知宇宙第一AntiCheat——EasyAntiCheat前段时间在`RustClient.exe`上部署了假`cr3`

消息来源：UC新闻网

[unknowncheats.me/forum/anti-cheat-bypass/580783-article-unleashing-secrets-easyanticheat-cr3-protection.html](https://bbs.kanxue.com/elink@058K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0f1^5x3o6M7^5x3#2)9J5k6r3q4J5N6r3W2U0L8r3g2Q4x3X3c8#2L8X3I4W2j5i4y4Z5K9h3&6Y4i4K6u0V1M7$3g2U0M7X3g2@1M7#2)9J5k6r3g2S2M7%4W2S2L8Y4c8A6j5$3S2W2j5i4c8Q4x3X3c8U0M7U0y4Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0W2K9s2c8E0L8l9`.`.)

[[News] Easy Anti-Cheat CR3 Trashing (unknowncheats.me)](https://bbs.kanxue.com/elink@893K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0f1^5y4K6V1@1y4W2)9J5k6r3g2S2M7%4W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1j5$3S2W2j5i4c8Q4x3X3c8U0M7U0y4Q4x3X3c8@1M7X3q4K6K9r3W2F1k6#2)9J5k6h3S2@1L8h3H3`.)

[Unleashing the Secrets: EasyAntiCheat’s CR3 Protection :: AVX's Blog (advancedvectorextensions.github.io)](https://bbs.kanxue.com/elink@42aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2k6S2L8X3y4W2k6s2k6W2j5%4c8G2M7X3g2^5N6r3g2F1M7$3W2G2L8Y4y4Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3k6h3q4K6P5h3q4F1N6r3W2U0K9r3g2S2N6q4)9J5k6r3y4J5x3#2)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1j5`.)

不知道具体原理的这里给大伙儿省个流：

1、偷指针，接管整个系统的异常处理

```
InternalData = HalpTimerGetInternalData( Timer );

Rax = ( *( __int64 ( __fastcall ** )( __int64 ) )( Timer + 0x70 ) )( InternalData );
```

2、给`EPROCESS`.`DirectoryTableBase`写非法数值，让这个值在`SwapContext`写入`cr3`时发生`#GP`异常

3、接管`#GP`异常，恢复真实`cr3`，从异常中恢复

大手子们针对EAC也是八仙过海各显神通

![](https://bbs.kanxue.com/upload/attach/202308/619065_XRKM94SPRBFZP4N.jpg)

有尝试暴力还原代码手动解密数据的：

[[Coding] eac new cr3 decryption routine (unknowncheats.me)](https://bbs.kanxue.com/elink@ac2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0f1&6x3K6p5I4z5g2)9J5k6r3g2S2j5#2)9J5k6r3y4J5x3#2)9J5k6r3c8W2j5%4u0&6M7s2c8A6L8$3&6Q4x3X3c8J5L8%4g2@1K9h3&6W2i4K6u0W2K9s2c8E0L8l9`.`.)

有插NMI的：

[[Coding] EAC Decrypted CR3 Method using NMIs & KPRCBs (unknowncheats.me)](https://bbs.kanxue.com/elink@0e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3E0F1L8%4N6F1j5$3S2W2j5i4c8K6i4K6u0W2L8h3g2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8X3q4F1N6r3W2Q4x3X3c8U0K9r3g2S2N6q4)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8U0f1&6y4e0p5K6x3#2)9J5k6r3g2S2j5#2)9J5k6r3c8W2j5%4u0&6M7s2c8W2k6q4)9J5k6r3y4J5x3#2)9J5k6r3#2W2N6r3S2G2k6q4)9J5k6s2g2K6K9h3&6Y4i4K6u0V1L8X3#2A6M7#2)9J5k6r3E0H3M7X3y4T1M7#2)9J5k6h3S2@1L8h3H3`.)

那么有没有更简单的无需更新的通杀方法呢？

拜读前辈的文章后我们得知，`cr3`对应的物理地址在`MmPfnDatabase`中的表项`MMPFN`.`ListEntry`，其实就是加密后的`EPROCESS`：

*[win10 1909逆向（通过任意物理帧判断是否是CR3和解密得到所属EPROCESS）-软件逆向-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-261778.htm)*

那么我们是不是可以遍历整个`MmPfnDatabase`，判断其中的`MMPFN`.`ListEntry`是否是游戏的`EPROCESS`，如果是，岂不是就可以从`MMPFN`反推出`CR3`对应的物理地址了？

半分钟随便乱写的伪代码（不保真）：

```
ULONG_PTR  ULONG_PTR = 0;

for(auto i = 0; i < MmPfnDatabaseSize; ++i)

{

  if(  DecryptEProcess(MmPfnDatabase[i]) == TargetGameEProcess)

  {

    RealDirectoryTableBase =  (ULONG_PTR)i << 12ull;

  }

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2023-8-16 17:04

被hzqst编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
