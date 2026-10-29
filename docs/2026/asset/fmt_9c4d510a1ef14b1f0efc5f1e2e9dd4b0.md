
```
#import <UIKit/UIKit.h>



//! Project version number for ShootMoonHacker.

FOUNDATION_EXPORT double ShootMoonHackerVersionNumber;



//! Project version string for ShootMoonHacker.

FOUNDATION_EXPORT const unsigned char ShootMoonHackerVersionString[];



// In this header, you should import all the public headers of your framework using statements like #import <ShootMoonHacker/PublicHeader.h>

#import <HackerLoader.h>

#import <NSObject+Hacker.h>

```

我们在文件的最后追加我们之前创建的类和分类。

`import`时记得使用尖括号，而不是双引号。

### NSObject+Hacker.h

我们在分类头文件中声明一个方法，叫`hack`，提供给`HackerLoader`类调用，执行破解代码。

```
#import <Foundation/Foundation.h>



@interface NSObject (Hacker)



- (void)hack;



@end

```

### NSObject+Hacker.m

现在我们开始改写 `tenPointLineY` 方法。

我们可以使用Objective-C运行时的方法交换来达到我们的目的。

此处我事先封装了交换方法的代码，参数是原始方法名、原始类名、新方法、新类、是否为类方法。

虽然我们目前只需要交换 `tenPointLineY` 这一个方法，但是考虑到以后可能需要交换多个方法，为了避免出现大量重复的代码，所以封装一下，同时也能让代码看起来整洁一点。

```
- (void)exchangeMethod:(NSString *)methodName ofClass:(NSString *)className toMethod:(SEL)method ofClass:(Class)class isInstanceMethod:(BOOL)isInstanceMethod {

    Class c = NSClassFromString(className);

    SEL sel = NSSelectorFromString(methodName);

    Method om = isInstanceMethod ? class_getInstanceMethod(c, sel) : class_getClassMethod(c, sel);

    Method nm = isInstanceMethod ? class_getInstanceMethod(class, method) : class_getClassMethod(class, method);

    method_exchangeImplementations(om, nm);

}
```

**注**：使用方法交换需要导入`<objc/runtime.h>`头文件。

```
#import <objc/runtime.h>
```

接下来编写我们自己的 `tenPointLineY` 方法，名字就叫 `my_tenPointLineY`。

```
- (float)my_tenPointLineY {

    NSLog(@">>>>> my_tenPointLineY");

    float ret = [self my_tenPointLineY];

    NSLog(@">>>>> my_tenPointLineY ret: %f", ret);

    ret = 200;

    NSLog(@">>>>> change tenPointLineY ret to: %f", ret);

    return ret;

}
```

`[self my_tenPointLineY]` 这一句，其实调用的是原始的 `tenPointLineY` 方法，因为在交换方法后，原始的 `tenPointLineY` 方法名对应的是 `my_tenPointLineY` 方法的代码段，而 `my_tenPointLineY` 方法名对应的是 `tenPointLineY` 方法的代码段。

具体可以搜索 `Method Swizzling` 来了解方法交换的细节。

我们不知道把10分线的Y坐标改为多少才是我们想要的效果，先改个200试试看。

添加`NSLog`打印日志是为了确定代码是否按我们预期的那样执行了。

写完 `my_tenPointLineY`，开始交换方法，也就是实现我们最开始声明的`hack`方法。

```
- (void)hack {

    NSString *className = @"MainEnemy";

    Class selfClass = [self class];



    // TenPointLineY

    [self exchangeMethod:@"tenPointLineY" ofClass:className

                toMethod:@selector(my_tenPointLineY) ofClass:selfClass

        isInstanceMethod:YES];

}
```

原始的 `tenPointLineY` 是属于 `MainEnemy` 类的，`my_tenPointLineY` 是属于 `NSObject` 类的，是实例方法。

### HackerLoader.m

我们希望在动态库加载时自动执行破解代码的话，需要这么写。

```
static void __attribute__((constructor)) entry(void) {

    NSLog(@">>>>> Code Injected <<<<<");

    NSObject *obj = [[NSObject alloc] init];

    [obj hack];

}
```

不要忘记导入头文件 `NSObject+Hacker.h`。

```
#import "NSObject+Hacker.h"
```

方法 `entry` 的名字不是固定的，可以是你想要的任何符合命名规则的名字。

关键的是 `__attribute__((constructor))` ，当动态库被加载的时候，会自动执行拥有此属性的方法。

## 编译

在编译之前，有些工程配置需要修改。

### 修改运行所需的最低系统版本

如下图，`iOS Deployment Target` 改为 `iOS 11.0`。

![](https://bbs.kanxue.com/upload/attach/201803/320395_GB46239AESWDJ6Q.png)

### 修改签名

如下图，**签名** 改为 **None**。

![](https://bbs.kanxue.com/upload/attach/201803/320395_H3FJYJF4P7ZUSAF.png)

### 修改编译目标

如下图，**编译目标** 改为 **Generic iOS Device**。

![](https://bbs.kanxue.com/upload/attach/201803/320395_BZWJE8M389JD5Z2.png)

配置修改完成后，按下`Command+B`编译。

编译成功后，就能得到`ShootMoonHacker.framework`。

## 动态库注入

展开`Products`目录，右键`ShootMoonHacker.framework`，选择`Show in Finder`。

![](https://bbs.kanxue.com/upload/attach/201803/320395_HCDKRMAG4EJUR2R.png)

新开一个`Terminal`窗口，输入 `cd`空格，然后把 `ShootMoonHacker.framework` 目录拖到`Terminal`窗口上，如下图。

![](https://bbs.kanxue.com/upload/attach/201803/320395_5KFXAD2WSGF55VM.png)

松开后，效果如下图。

![](https://bbs.kanxue.com/upload/attach/201803/320395_BXUHUVVWZU6BXG3.png)

按下回车键后，我们需要将 `ShootMoonHacker` 文件传到iPhone上，命令如下。

```
$ scp -P 2222 ShootMoonHacker root@localhost:~/
root@localhost's password: # 输入密码
```

我猜测 `tenPointLineY` 这个方法会在开始游戏时调用，所以先将游戏退到初始界面，然后回到通过电脑ssh登录到iPhone上的`Terminal`窗口，进入到`bfinject`目录，输入如下命令注入我们的动态库。

```
# 注入动态库
$ bash bfinject -P shootthemoon -l ~/ShootMoonHacker

# 输出结果
[+] Liberios detected
[+] Injecting into '/var/containers/Bundle/Application/24546F40-53DA-4C49-81C0-DED0C13DB814/shootthemoon.app/shootthemoon'
[+] Getting Team ID from target application...
[+] Thinning dylib into non-fat arm64 image
[+] Signing injectable .dylib with Team ID A6DVR4V967 and platform entitlements...
[bfinject4realz] Calling task_for_pid() for PID 3593.
[bfinject4realz] Calling thread_create() on PID 3593
[bfinject4realz] Looking for ROP gadget... found at 0x1826574e0
[bfinject4realz] Fake stack frame at 0x10b3c8000
[bfinject4realz] Calling _pthread_set_self() at 0x182897804...
[bfinject4realz] Returned from '_pthread_set_self'
[bfinject4realz] Calling dlopen() at 0x182657460...
[bfinject4realz] Returned from 'dlopen'
[bfinject4realz] Success! Library was loaded at 0x1c0151b40
[+] So long and thanks for all the fish.
```

**注意**：

1、`bfinject`命令必须在`bfinject`目录下执行，在其它目录下执行会出错。

2、如果注入过程中出现闪退，按`Ctrl+C`退出，重启`App`，再执行一遍注入命令即可。

注入成功后，我们开始游戏看一下效果。

![](https://bbs.kanxue.com/upload/attach/201803/320395_XS2PR9XR5ZKJDXG.png)

就像游戏中的肥蜜蜂一样，我皱起了眉头。

怎么没有任何效果，是不是动态库没有执行？

## 查看日志

让我们在电脑上打开`Console`，选择iPhone，重启游戏再执行一遍注入。

![](https://bbs.kanxue.com/upload/attach/201803/320395_WP79TAK6BF4NZ58.png)

由于日志太多，我们在搜索栏输入`>>>`，按下回车，可以在列表中看到我们注入成功的代码。右键选择我们动态库输出的那行日志，选择`Show Process 'shootthemoon'`，这样列表就会隐藏其它和我们`App`无关的日志了。

![](https://bbs.kanxue.com/upload/attach/201803/320395_C8T9MNV4PWUGYJW.png)

日志告诉我们，注入成功了，但是10分线的位置为什么没有改变呢？开始游戏的时候似乎也没有调用我们的代码。难道10分线的位置是启动游戏时候调用的？抱着试试看的想法，我们在游戏启动的瞬间执行注入命令，不过这么做会导致游戏卡住然后闪退，只能换一种方式了。

## 换个姿势插入动态库

还记得我们的 `shootthemoon.app` 目录吗？把我们的动态库文件 `ShootMoonHacker` 复制到这个目录下。

接下来我们要使用 `macho_edit` 工具来修改 `shootthemoon` 可执行文件的动态库列表。

还记得我们备份出来的那个 `shootthemoon` 文件吗？我们对它下手。

执行如下命令。

```
$ macho_edit shootthemoon
# 输出内容
Thin mach-o binary:
    arm64 arch (offset 0x0)

1 Fat mach-o configuration
2 Load command edit
3 Exit

Select an option: 2
```

选择 `2 Load command edit`，编辑列表。

```
1 List load commands
2 Remove load command
3 Insert load command
4 Move load command
5 Remove code signature
6 Cancel

Select an option: 1
```

选择 `1 List load commands`，看看原始的列表有哪些内容。
