
# [原创]调试小技巧：绕过 apk 安装时的签名验证

小菜第一次发帖，不知道这篇文章有没有意义，大家多多包涵，多多指点。本文所说的小技巧的作用只有一个，就是能让你重打包某些有签名验证的 apk 用于调试，除此之外，确实没什么用处，大牛们可以直接飘过了。![](https://bbs.kanxue.com/view/img/face/1.gif)

## 1、

事情是这样的，最近在调试一个 apk，有签名验证，验证方法是将签名信息发往服务器验证的，所以不能进行重打包。更变态的是，签名信息的获取是在 `.so` 里调用 Java 层的 `getPackageInfo` 实现的。

最初的想法就是能 hook 住 `getPackageInfo`，替换掉里面的签名信息。试过用 `xposed` hook 函数 `getPackageInfo`，但是没有成功，咨询过高人说是“动态加载的 `.so` 是没法 hook 住的”，所以这种方法就作罢了。还想过 `.so` 注入，定制 rom 等方法，但因自己都没有做过，就没有尝试。

## 2、

其实自己一直有个想法，就是能不能把重打包之后的 apk 里的签名信息 `RSA` 文件换成原 apk 的签名信息，如果这样能安装成功的话，是不是就能突破这种签名验证了。

在网上看到了一篇文章[cf5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2L8X3c8S2L8q4)9J5k6h3&6W2N6q4)9J5c8U0x3J5x3g2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@206K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2L8X3c8S2L8q4)9J5k6h3&6W2N6q4)9J5c8U0x3J5x3g2)9J5k6h3S2@1L8h3H3`.)，说通过修改 `services.jar` 能“移除 apk 的签名验证机制”，于是就试验了一下，把重打包过的 apk 里的 `CERT.RSA` 文件替换为原先 apk 里的 `CERT.RSA`，然后按照作者的方法修改了 `services.odex`（网上有对 odex 的修改方法），结果还是没有安装成功，但是直接修改 odex 的方法还是很不错的。错误的原因可能还是因为修改的函数不对引起的。于是，根据安装时报出来的错误 `INSTALL_PARSE_FAILED_NO_CERTIFICATES`，还有 `Logcat` 里的堆栈信息，去源码里去找。最后定位到了出错的位置是在 `android4.0\libcore\luni\src\main\java\org\apache\harmony\security\utils\JarUtils.java` 中的 `verifySignature` 函数，在函数的最后：

```
if(!sig.verify(sigInfo.getEncryptedDigest())) {

               throw new SecurityException("Incorrect signature");

            }
```

抛出了个异常，罪魁祸首就是这个异常，函数的意思应该是验证 `CERT.RSA` 里的签名信息正确与否，所以这里应该直接跳过就 OK 了。

观察源码，最后这部分代码是编译到了 `core.jar` 里，也就是 `/system/framework/core.odex` 里。

将 `code.odex` 反编译出来，找到 `\core\org\apache\harmony\security\utils\JarUtils.smali`，在 `verifySignature` 函数中找到下面的位置，添加上红色的那句话。

```
invoke-virtual {v0, v1}, Ljava/security/Signature;->verify([B)Z
move-result v27
const v27, 0x1
if-nez v27, :cond_186
```

最后，重新编译成 odex，替换掉原先的 `core.odex`。

再按上面说的，将重打包过的 apk 里的 `CERT.RSA` 替换为原先 apk 里的 `CERT.RSA`。就可以正确安装，并且获取的签名信息还是原先的 apk 的签名信息，可以愉快的进行调试了。。。

### PS:

直接修改 odex 据说是有风险的，所以，大家小伙伴们修改之后要是开不了机，不要来找我。上面的测试只在 NEXUS 4 上测试通过。

另外，这只是自己想出来的方法，不知道大家都是用的什么方法，希望大家不吝赐教啊。

--------------------------------------

2014-08-07 14：10

多谢 `@SpaceDye` 提醒，我怎么就没想到写个 `xposed` 插件呢。![](https://bbs.kanxue.com/view/img/face/27.gif)

代码就几行，但能达到上面修改 `code.odex` 的效果。代码就直接贴出来了。

```
public void initZygote(StartupParam startupParam) throws Throwable {

    XposedHelpers.findAndHookMethod("java.security.Signature",null,"verify", byte[].class,new XC_MethodHook(){
        protected void afterHookedMethod(MethodHookParam param) throws Throwable {
            XposedBridge.log("disabled verifysignature......");
            param.setResult(Boolean.TRUE);
        }
    });
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`DisableVerifySignature.apk`](javascript:void(0);)

  （278.50kb，1240次下载）
