
![](https://bbs.kanxue.com/upload/attach/202006/776128_GZBDQ75KDY5SAYA.jpg)

## 寻找注入点 Hook

虽然最终没能成功实现免越狱 hook 修改游戏，但是我们还是拿它下手。

首先通过 `Class-dump` 将头文件 dump 下来，在其中寻找一个有代表性的函数进行 Hook，最终决定 hook `UnityAppController` 这个类中的 `init` 函数，在它调用的时候弹出弹框。

在 `your_projectDylib.xm` 中 Hook 代码如下：

```
#import <substrate.h>
#import <sys/sysctl.h>
#import <UIKit/UIKit.h>

%hook UnityAppController

// Hooking an instance method with an argument.
- (void)init {
    %orig;

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Hook成功" message:nil delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
    [alert show];
    //[alert release];
}

%end
```

运行程序将 `ipa` 装到未越狱手机上，游戏打开之后会有弹框出现。

![](https://bbs.kanxue.com/upload/attach/202006/776128_QAWE4XK35KTB82Z.jpg)

## 总结

本来准备写一篇免越狱 Hook 结合 Unity3D 游戏修改为一体的文章，但是在 Hook C 函数的过程中遇到了难题，目前还没有找到未越狱手机上 `inlinhook` 的好方案，只能将两者分开来写，之后找到好的方案之后再进行修改为大家分享。

下面附上本人 iOS Unity3D 入门看的一篇文章，本文大致参考了这位大佬的框架。[iOS上的Unity逆向与弓箭传说游戏修改](<https://bbs.kanxue.com/elink@ea3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6j5$3S2W2L8Y4S2A6j5h3!0&6N6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2W2M7X3!0Q4x3V1k6Q4x3U0y4Q4x3U0g2q4y4g2)9J5y4f1t1%4i4K6t1#2b7e0g2Q4x3U0g2q4y4g2)9J5y4e0R3#2i4K6t1#2b7U0M7>)。

**本文仅用来学习交流，禁止用来做恶意事情，否则后果自付！如有侵权，请联系作者删除**

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
