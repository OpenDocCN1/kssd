
# [翻译]OWASP iOS crackme 的教程：使用Frida来解决

在这篇文章中，我将解释如何使用Frida解决OWASP移动安全测试指南（MSTG）1级的Crackme。 非常感谢[Bernhard Mueller](https://bbs.kanxue.com/elink@d9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3L8i4g2W2L8r3I4W2M7X3u0W2M7X3&6V1N6l9`.`.)创建这些crackme，并[鼓励人们写教程阐述如何使用开源工具来攻破他们](https://bbs.kanxue.com/elink@8aeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3L8i4g2W2L8r3I4W2M7X3u0W2M7X3&6V1N6q4)9J5c8Y4y4@1j5i4c8#2M7#2)9J5c8U0R3#2y4K6p5%4z5e0f1K6y4o6x3$3z5e0b7^5z5o6R3&6y4H3`.`.)。 最后，我将再次使用OWASP iOS crackme教程，并展示如何使用NowSecure上的一些工具打破crackme。

## OWASP iOS crackme 指导的概述

首先，我下载这个app并重新签名，这样就能把它安装到我的测试机上。 由于本教程是关于动态分析的，所以我得使用一个真实机器，还得安装[Frida](https://bbs.kanxue.com/elink@064K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3W2F1M7%4c8S2L8r3I4S2N6r3W2G2L8W2)9J5c8R3`.`.)。安装了app后，运行起来，观察UI，然后用我的工具来解决这个谜题。 如果没有工具来揭开这个app的面具，几乎不可能猜到答案并解决问题。 这才是关键，动态分析不仅是得以一窥app奥秘的的强大工具，也是逆向app和执行app安全测试的强大工具。 我将展示一下如何使用UI完全自动化的解决方案来处理这个crakeme，其中有一些构建在Frida之上的非常强大的工具。

继续！

## 下载&给app重签名

你可以在[这拿到crakeme.ipa](https://bbs.kanxue.com/elink@cf9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6g2@1q4e0f1q4)9J5c8X3!0%4j5i4y4H3i4K6u0V1L8i4y4@1k6#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6a6e0g2c8s2i4K6u0V1c8X3W2D9k6i4y4Q4x3V1j5H3x3W2)9#2k6V1y4J5j5h3y4C8L8h3g2K6i4K6u0r3x3o6u0Q4y4h3k6A6e0#2y4Q4x3V1k6x3k6i4k6W2L8q4)9#2k6U0l9I4i4K6u0r3)

你可以在这儿follow[OWASP的crakeme工具](https://bbs.kanxue.com/elink@61eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6g2@1q4e0f1q4)9J5c8X3!0%4j5i4y4H3i4K6u0V1L8i4y4@1k6#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6a6e0g2c8s2i4K6u0V1c8X3W2D9k6i4y4Q4x3V1j5H3x3W2)9#2k6V1y4J5j5h3y4C8L8h3g2K6i4K6u0r3e0r3W2K6N6q4)9#2k6X3!0X3i4K6g2X3b7%4u0S2j5$3E0E0k6i4y4Q4x3X3g2E0k6q4)9J5x3%4g2K6k6i4u0Q4x3X3c8U0L8$3&6@1k6h3&6@1i4K6u0V1K9h3!0K6)，或者使用Cydia Impactor，iSign等等。

我会使用nowsecure/applesign工具，这是一套非常实用的苹果相关的签名相关的工具。

```
$ applesign ./UnCrackable_Level1.ipa -B -c -d --identity xxx --mobileprovision ~/.isign/Manual_iOS_Team_Provisioning_Profile.mobileprovision --output /tmp/resigned.ipa

Unzipping /Users/user/apple/ipa/crackmes/UnCrackable_Level1.ipa

Payload found

Main IPA executable is not encrypted

Embedding new mobileprovision

{"keychain-access-groups":["xxx.*"],"get-task-allow":true,"application-identifier":"xxx.*","com.apple.developer.team-identifier":"xxx"}

Cloning entitlements

Updated binary entitlements/Users/user/apple/ipa/crackmes/UnCrackable_Level1.ipa.eaf60e81-9162-4437-a954-438301932004/Payload/UnCrackable Level 1.app/UnCrackable Level 1.entitlements

Signing libraries and frameworks

Executable found at /Users/user/apple/ipa/crackmes/UnCrackable_Level1.ipa.eaf60e81-9162-4437-a954-438301932004/Payload/UnCrackable Level 1.app/UnCrackable Level 1

Resolving signing order using layered list

Signed /Users/user/apple/ipa/crackmes/UnCrackable_Level1.ipa.eaf60e81-9162-4437-a954-438301932004/Payload/UnCrackable Level 1.app/UnCrackable Level 1

Verifying /Users/user/apple/ipa/crackmes/UnCrackable_Level1.ipa.eaf60e81-9162-4437-a954-438301932004/Payload/UnCrackable Level 1.app/UnCrackable Level 1

Zipifying into /tmp/resigned.ipa ...

Cleaning up /Users/user/apple/ipa/crackmes/UnCrackable_Level1.ipa.eaf60e81-9162-4437-a954-438301932004

Target is now signed: /tmp/resigned.ipa
```

使用Applesign有一大优点，它能自动启用重签名的app的get-task-allow权限，这对于附加调试器是很有用的。

现在我有一个重新签名版本的.ipa可以安装在设备上 - 我将使用libimobiledevice工具。

```
$ ideviceinstaller -i /tmp/resigned.ipa

WARNING: could not locate iTunesMetadata.plist in archive!

WARNING: could not locate Payload/UnCrackable Level 1.app/SC_Info/UnCrackable Level 1.sinf in archive!

Copying './resigned.ipa' to device... DONE.

Installing 'sg.vp.UnCrackable1'

Install: CreatingStagingDirectory (5%)

Install: ExtractingPackage (15%)

Install: InspectingPackage (20%)

Install: TakingInstallLock (20%)

Install: PreflightingApplication (30%)

Install: InstallingEmbeddedProfile (30%)

Install: VerifyingApplication (40%)

Install: CreatingContainer (50%)

Install: InstallingApplication (60%)

Install: PostflightingApplication (70%)

Install: SandboxingApplication (80%)

Install: GeneratingApplicationMap (90%)

Install: Complete
```

所以现在我可以在设备上运行这个app，如果你也跟着做到这儿了，应该在屏幕上看到类似的内容：

![](https://bbs.kanxue.com/upload/attach/201705/746111_o1eimw5e5qyw4b5.png)

我需要使用“在隐藏标签中发现一个秘密！”的提示来寻找解开crackme所需要的文本。

如果我不能输入正确的数值，那我会得到下面的结果：

![](https://bbs.kanxue.com/upload/attach/201705/PUUSZZ5PD6JPM3J.jpg)

## 提示

**前方警惕** - 过了这一步我得警告一下，再继续就得风险自负。

现在我要动用提示和一些聪明才智来解决这个问题。 我可以尝试的一件事是dump应用程序的视图层次结构。

来自于[苹果的档案](https://bbs.kanxue.com/elink@ab5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8Y4g2A6K9$3W2@1i4K6u0r3N6h3W2%4K9h3&6V1L8%4N6Q4x3@1k6D9j5h3&6Y4N6h3q4Y4k6g2)9K6c8r3!0T1K9X3x3`.)**:

*Only one window at a time can be the key window, and you can use a window’s keyWindow property to determine its status.（一次只能有一个窗口是关键窗口，您可以使用窗口的keyWindow属性来确定其状态。）*

所以我可以在应用程序运行时使用Frida询问当前的UIWindow及其视图层次结构。 随着Frida及其python命令行界面（CLI）工具的安装，我将执行以下操作：

```
$ frida -U UnCrackable1

     ____

    / _  |   Frida 9.1.27 - A world-class dynamic instrumentation framework

   | (_| |

    > _  |   Commands:

   /_/ |_|       help      -> Displays the help system

   . . . .       object?   -> Display information about 'object'

   . . . .       exit/quit -> Exit

   . . . .

   . . . .   More info at 28fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5K9h3c8S2i4K6u0W2M7X3g2Q4x3V1k6V1L8$3y4K6i4K6u0r3K9r3!0E0k6g2)9J5c8R3`.`.

[USB::iPhone::UnCrackable1]->
```

### Frida的 Objective-C API

您可以在这里了解更多关于Frida的Obj-C API。 使用这些API，我可以弄清楚如何绑定到UIWindow，获取它的keyWindow，然后，通过足够的研究，找到将dump整个视图层次结构的recursiveDescription函数来查看。

## 终极解决
