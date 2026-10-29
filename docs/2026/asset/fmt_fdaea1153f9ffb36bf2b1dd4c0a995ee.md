
```text
. have the jailbreak untether leave System partition read-only

. 重启

. dd if=/dev/rdisk0s1s1 of=backup bs=512k count=1

. use kextloader to run a decrypted and pwned iBEC

. once in iBEC, upload dtre/rdsk/krnl and bootx

. ssh into the ramdisk

. nvram boot-ramdisk="/a/b/c/d/e/f/g/h/i/j/k/l/m/disk.dmg"

. dd of=/dev/rdisk0s1s1 if=ramdiskF.dmg bs=512k count=1

.重启
```

当设备重新启动时，您会注意到启动标志闪烁（即漏洞利用代码重新启动IBoot），然后进入恢复控制台。我们现在可以用`irecovery`[12]连接它：

```text
irecovery -s
```

要想退出这种模式，我们要这样做：

- in the pwned Recovery, upload a pwned iBEC and jump to it
- once in iBEC, upload `dtre/rdsk/krnl` and `bootx`
- `ssh` into the ramdisk
- `dd of=/dev/rdisk0s1s1 if=backup bs=512k count=1`
- `nvram -d boot-ramdisk`
- `reboot`

这个bug的实际应用是引导任何未签名的内核。考虑创建一个第三分区并触发那里的漏洞：

```text
. dd of=/dev/rdisk0s1s3 if=ramdiskG.dmg bs=512k count=1

. nvram boot-partition=2

. nvram boot-ramdisk="/a/b/c/d/e/f/g/h/i/j/k/l/m/disk.dmg"
```

当然，payload需要这样修改：

- 移到`sp`
- 禁用解释器
- 应用那个补丁, 从0分区加载内核, 忽略`ramdisk`
- 解压 `nettoyeur`
- 静默安装
- 运行 `nettoyeur`
- 跳转到`iBoot`起始点然后再次运行

## 3. 结论

这一特定漏洞及其利用方法的旅程到此结束。它已经在iOS 8上修复了，然而，回想起来我觉得它很有趣，因为发动攻击的环境非常恶劣。此外，触发它导致所有地狱挣脱和狙击的出路肯定是有趣的。最后，能从中学到一些教训。

对这种漏洞利用不起作用的缓解措施：

- 堆栈金丝雀不起作用，因为在传统意义上，我们并没有溢出堆栈；我们溢出了整个 heap。`ReadExtent` 将永远不会返回，`memalign` 的堆栈没有溢出，而是使用一个精确的写入来重写返回地址。
- 任务调度程序检查根本没有帮助。一旦触发，就不会有`task_yield()`，所以这些检查永远不会发生。
- `iBoot`中的`W^X`会让漏洞利用更加困难，但作用不大, 因为我们可以直接置成`ROP`。

对这种漏洞利用起作用的缓解措施：

- 数据和堆之间未映射的保护页会阻止这种攻击，因为我们不能用小堆栈帧跳过它们。再好一点，应该为每个任务设置防护页。
- 硬切割递归将有帮助。也就是说，一个限制递归的检查`ResolvePathToCatalogEntry()`。这就是苹果如何在iOS 8中修补它的方法。
- 堆随机化 或者`IASLR`将阻止漏洞利用。

## 4. 参考

```text
[1] 344K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8h3u0W2k6r3c8W2k6r3N6#2M7Y4g2K6i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8S2N6r3g2Q4x3X3c8K6M7r3q4U0k6g2)9J5c8U0t1H3x3e0c8Q4x3V1j5H3x3W2)9J5c8X3q4J5k6g2)9J5k6s2N6W2i4K6u0V1M7$3S2G2L8%4c8A6L8X3N6Q4x3X3c8G2N6i4u0K6k6h3I4$3k6i4y4Q4x3X3c8A6L8W2)9J5k6s2c8Z5k6g2)9J5k6r3k6G2L8%4c8Q4x3X3c8%4K9i4c8Z5i4K6u0V1M7%4c8S2j5$3E0Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5c8R3`.`.
[2] e0cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3&6X3k6i4u0W2L8X3y4W2i4K6u0W2K9r3W2@1j5W2)9J5k6h3!0J5k6#2)9J5c8X3S2A6N6r3u0K6k6h3y4U0L8$3&6X3x3U0l9I4x3$3E0#2L8q4)9J5c8X3#2S2N6r3g2J5K9h3q4D9M7#2)9J5c8V1b7J5g2o6q4Q4x3U0f1J5x3q4)9J5k6q4)9J5y4e0t1H3d9X3!0K6K9s2g2S2i4K6t1#2x3U0m8Q4x3U0N6H3x3s2y4A6P5r3&6A6L8X3A6S2i4K6t1%4i4K6t1#2x3U0m8t1K9h3I4D9i4K6t1#2x3U0m8Q4x3X3c8Q4x3U0f1J5x3q4y4t1b7i4c8@1k6i4u0W2k6q4)9J5y4e0t1H3c8s2u0W2j5h3#2K6i4K6u0W2M7r3c8X3
[3] 93fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5k6i4u0#2j5W2)9J5c8X3W2G2M7#2)9J5k6r3E0W2P5r3g2U0i4K6u0V1N6i4c8A6L8s2x3`.
[4] 58fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1K9r3g2A6M7r3S2G2L8X3g2%4K9h3E0A6i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2C8K9g2)9J5c8V1W2F1L8Y4y4T1M7Y4g2U0K9#2c8S2L8%4x3`.*11B554a*(iPhone3,1)
[5] d1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6f1j5i4u0B7j5h3&6Q4x3U0N6K6i4K6g2X3M7%4c8J5L8$3&6Y4L8s2W2Q4y4h3k6U0L8$3&6F1k6h3y4@1k6h3c8Q4y4h3k6U0L8$3#2H3L8$3&6W2L8Y4c8K6i4K6g2X3j5h3I4Y4L8%4u0A6N6r3S2E0
[6] f3dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5k6i4u0#2j5W2)9J5c8X3W2V1j5i4y4@1N6h3k6X3i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4c8S2M7X3A6S2L8W2)9J5k6i4m8&6
[7] 3e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6t1c8W2x3`.*Plus
[8] f83K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6n7i4K6u0V1N6s2u0W2k6b7`.`.
[9] bc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4P5s2c8W2L8Y4b7`.*(file\_systems)
[10] f1eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7r3g2F1M7$3!0#2M7X3y4W2i4K6u0W2j5i4m8H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6@1j5i4u0T1j5h3I4D9M7#2)9J5c8V1u0G2L8%4c8j5i4K6u0r3b7X3!0G2N6q4S2Q4x3X3b7^5x3g2)9J5k6i4c8S2M7W2)9J5k6h3N6\*7
[11] d00K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1K9r3g2A6M7r3S2G2L8X3g2%4K9h3E0A6i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2C8K9g2)9J5c8V1N6u0c8q4)9#2k6V1E0W2P5b7`.`.
[12] 4b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5k6i4u0#2j5W2)9J5c8X3W2J5k6h3y4G2N6X3g2J5P5b7`.`.
```

## 5. 源码
