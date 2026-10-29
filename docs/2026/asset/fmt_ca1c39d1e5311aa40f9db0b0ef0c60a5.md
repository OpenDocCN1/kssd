
`2023-04-01 23:53:12.023 24017-24429/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z_btog==/lib/arm64/yyyy.so start-> 0x77a275b000 end-> 0x77a27ad000 size -> 335872  2023-04-01 23:53:12.140 24017-24247/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z_btog==/lib/arm64/kkkk.so start-> 0x778f68b000 end-> 0x779054c000 size -> 15470592  2023-04-01 23:53:12.172 24017-24424/? I/Zhenxi: linker load libnetd_client.so start-> 0x78ff86c000 end-> 0x78ff875000 size -> 36864  2023-04-01 23:53:12.362 24017-24429/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z_btog==/lib/arm64/libckeygeneratorV2.so start-> 0x7783a48000 end-> 0x7783aeb000 size -> 667648  2023-04-01 23:53:12.410 24017-24429/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z_btog==/lib/arm64/libxps_ws.so start-> 0x778159a000 end-> 0x77816ef000 size -> 1396736  2023-04-01 23:53:12.611 24017-24247/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z_btog==/lib/arm64/libDownloadProxy.so start-> 0x7778402000 end-> 0x7778d73000 size -> 9900032  2023-04-01 23:53:12.700 24017-24247/? I/Zhenxi: linker load libc.so start-> 0x7900557000 end-> 0x7900653000 size -> 1032192  2023-04-01 23:53:12.729 23903-23903/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z_btog==/lib/arm64/kkkk.so start-> 0x772d186000 end-> 0x772e047000 size -> 15470592 ...`

### Java 调用方法监听：

主要是Hook了`artmethod`的`invoke`方法，一切的java方法底层都会走这个方法，包括脱壳也是修改的这个方法获取被抽取的指令，然后对Dex进行重构 。

我想尝试在这个方法里面对参数进行打印，但是失败了，在源码里面很好修改，但是通过Hook的话很多函数拿不到 ，加上一些核心的方法被编译器inline了，很不好操作和处理，就没继续关注了 。如果你有想法和思路可以提issues 或者提交代码 。这个方法不建议开启，个人用的很少，主要打印量太大了，一秒几千条日志 。他会打印系统的一些Java方法 ，所以很卡顿 ，不过如果你想做监听和记录 ，分析一些隐藏Api很有用 。`callback`实现如下 。

```
HOOK_DEF(void*, invoke, void *thiz, void *self, uint32_t *args, uint32_t args_size, void *result, const char *shorty) {
    string basicString = invokePrintf_org_PrettyMethodSym(thiz, true);
    LOG(INFO) << "invoke method info -> " << basicString;
    if (isSave) {
        *invokeOs << basicString.append("\n");
    }
    return orig_invoke(thiz, self, args, args_size, result, shorty);
}
```

### nativeLib :

这个是一个我自己封装的一个`native`库，里面有很多常用的方法，都进行了封装 ，删除了一些改机和没用的模块，留了一些常用的模块 。

主要是下面几个比较常用的 ，也方便后续开发和维护 。

- `hook`模块
- 符号查找模块（支持非导出函数）
- 常用工具类模块
- `raw_syscall`模块
- `libpath`

比如常见的`hook`操作 。如何`inlinehook`少于四个字节的方法，如何插装`hook`和如何异常`hook` 。这些都不需要关注，导入头文件以后直接 。

```
HookUtils::Hooker(xxx,(void *) xxx,(void **) &new_xxx);
```

一键`hook`即可 。底层封装分三步实现，先`dobby hook` ，失败了则使用异常`hook`， 最后`dobby`插装 。代码如下

```
bool HookUtils::Hooker(void *dysym, void *newrep, void **org) {
    if (dysym == nullptr) {
        LOG(ERROR) << "dobby hook org == null ";
        return false;
    }
    if (hookedList == nullptr) {
        hookedList = new list<void *>();
    }

    //如果这个地址已经被Hook了 。也有可能返回失败 。dobby 会提示 already been hooked 。
    for (void *ptr : *hookedList) {
        if (ptr == dysym) {
            //如果保存了这个地址,说明之前hook成功过,我们也认为hook成功
            return true;
        }
    }

    bool ret = DobbyHook(dysym, reinterpret_cast<dobby_dummy_func_t>(newrep), reinterpret_cast<dobby_dummy_func_t *>(org)) == RT_SUCCESS;
    if (ret) {
        //LOG(ERROR) << "hook utils hook success !";
        //将地址添加到已经hook的列表,防止这个地址被多次hook
        hookedList->push_back(dysym);
        return true;
    }

    //如果dobby hook失败了,采用sandhook异常hook进行补救,
    LOG(ERROR) << "zhenxi runtime inlinehook start sandhook InlineHookImpl  ";
    ret = SandHook::Inline::InlineHookImpl(dysym, newrep, org);
    if (ret) {
        hookedList->push_back(dysym);
        return true;
    }

    LOG(ERROR) << ">>>>>>>>>>>>>>> sandhook inlinehook hook error,start dobby branch_trampoline hook ";
    //如果sandhook sign hook 也失败了,我们采用dobby附近插装去hook
    dobby_enable_near_branch_trampoline();
    //二次hook
    ret = DobbyHook(dysym, reinterpret_cast<dobby_dummy_func_t>(newrep), reinterpret_cast<dobby_dummy_func_t *>(org)) == RT_SUCCESS;
    //关闭附近插装
    dobby_disable_near_branch_trampoline();
    if (!ret) {
        LOG(ERROR) << "!!!!!!!!!!!!!!!  HookUtils hook error   ";
        return false;
    }
    hookedList->push_back(dysym);
    return ret;
}
```

其他的不一一概述了，感兴趣的可以去看代码 。项目主要采用`C++ 20`编译的，需要`NDK 23`以上版本支持 。

### 项目地址：

`e3dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4x3U0V1$3y4o6R3^5x3K6t1H3i4K6u0r3h3s2m8G2M7$3g2V1d9X3&6A6g2s2u0S2j5$3f1`.

### 结语：

可以基于我这个项目改改即可实现`Xposed Hook Naitve`的其他功能 。不过好像`Lsp`已经提供了`NaitiveHook`,但是具体的`Api` 还一直没看。

等什么时候需要的时候再说吧，这个版本为了兼容老版本`Xposed` ，用的构建版本很低 。后续可能仅支持`lsp`了 。

愿天下剑士人人会两袖青蛇， 人人可剑开天门 。

------《雪中悍刀行》

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-4-2 20:16

被珍惜Any编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#NDK分析](https://bbs.kanxue.com/forum-161-1-119.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)

上传的附件：

- [FunJni_v1.2_2023-04-02_release.apk](javascript:void(0);)

  （3.45MB，560次下载）
