
```objective-c
/* @class SiShiPurchaseHelper */

-(void)startRestore:(void *)arg2 failedBlock:(void *)arg3 {

    r12 = [arg3 retain];

    rbx = [arg2 retain];

    [self setStartPurchase:0x1];

    [self setCompeletedBlock:rbx];

    [rbx release];

    [self setFailedBlock:r12];

    [r12 release];

    rax = [SKPaymentQueue defaultQueue];

    rax = [rax retain];

    [rax restoreCompletedTransactions];

    [rax release];

    return;
}
```

这里发现代码没办法跟进去了。通过查阅资料，`SKPaymentQueue` 是一个 APL 在 Mac 下支持的 lib 库，用于桌面的支付操作，所以一定有一个 Delegate 回调方法用于处理支付的校验，怎么找呢？

我们回到刚才生成的头文件文件夹下，执行 Linux 命令：`ls *SK*`，意思是查找所有包含 `SK` 字符串的头文件名：

```text
OSSPlainTextAKSKPairCredentialProvider.h

SKPaymentTransactionObserver-Protocol.h

SKProductsRequestDelegate-Protocol.h

SKRequestDelegate-Protocol.h
```

返回的内容如上，很明显，`SKPaymentTransactionObserver-Protocol.h` 就是实现的协议了。

```objective-c
#import "NSObject-Protocol.h"

@class NSArray, NSError, SKPayment, SKPaymentQueue, SKProduct;

@protocol SKPaymentTransactionObserver- (void)paymentQueue:(SKPaymentQueue *)arg1 updatedTransactions:(NSArray *)arg2;

@optional
- (void)paymentQueue:(SKPaymentQueue *)arg1 didRevokeEntitlementsForProductIdentifiers:(
- (void)paymentQueueDidChangeStorefront:(SKPaymentQueue *)arg1;
- (BOOL)paymentQueue:(SKPaymentQueue *)arg1 shouldAddStorePayment:(SKPayment *)arg2 forP
- (void)paymentQueue:(SKPaymentQueue *)arg1 updatedDownloads:(NSArray *)arg2;
- (void)paymentQueueRestoreCompletedTransactionsFinished:(SKPaymentQueue *)arg1;
- (void)paymentQueue:(SKPaymentQueue *)arg1 restoreCompletedTransactionsFailedWithError:
- (void)paymentQueue:(SKPaymentQueue *)arg1 removedTransactions:(NSArray *)arg2;
@end
```

它实现了几个回调方法，不管它，在 Hopper 中搜索 `paymentQueue`，并生成伪代码：

```objective-c
/* @class SiShiPurchaseHelper */

-(void)paymentQueue:(void *)arg2 updatedTransactions:(void *)arg3 {

    rbx = self;

    rax = [arg3 retain];

    var_150 = intrinsic_movaps(var_150, 0x0);

    *(int128_t *)(&var_150 + 0x10) = intrinsic_movaps(*(int128_t *)(&var_150 + 0x10), 0x0);

    *(int128_t *)(&var_150 + 0x20) = intrinsic_movaps(*(int128_t *)(&var_150 + 0x20), 0x0);

    *(int128_t *)(&var_150 + 0x30) = intrinsic_movaps(*(int128_t *)(&var_150 + 0x30), 0x0);

    var_B8 = rax;

    rax = [rax countByEnumeratingWithState:&var_150 objects:&var_B0 count:0x10];

    var_D8 = rax;

    if (rax != 0x0) {

            var_100 = **(&var_150 + 0x10);

            var_C0 = rbx;

            do {

                    r12 = 0x0;

                    do {

                            if (*var_140 != var_100) {

                                    objc_enumerationMutation(var_B8);

                            }

                            r14 = *(var_148 + r12 * 0x8);

                            rax = [r14 transactionState];

                            if (rax != 0x3) {

                                    if (rax != 0x2) {

                                            if (rax == 0x1) {

                                                    // 关键位置

                                                    [rbx completeTransaction:r14];

                                            }

                                    }

                                    else {

                                            rax = [r14 error];

                                            rax = [rax retain];

                                            r14 = [rax code];

                                            [rax release];

                                            if (r14 == 0x2) {

                                                    rbx = var_C0;

                                                    [rbx purchaseFailedWithError:0x0];

                                            }

                                            else {

                                                    rbx = var_C0;

                                                    [rbx purchaseFailedWithError:[[[[NSBundle mainBundle] retain] localizedStringForKey:@"Unlock failed" value:@"" table:0x0] retain]];

                                                    [rax release];

                                                    [rax release];

                                            }

                                    }

                            }

                            else {

                                    [rbx completeTransaction:r14];

                            }

                            r12 = r12 + 0x1;

                    } while (r12 < var_D8);

                    rax = [var_B8 countByEnumeratingWithState:&var_150 objects:&var_B0 count:0x10];

                    var_D8 = rax;

            } while (rax != 0x0);

    }

    var_30 = **___stack_chk_guard;

    [var_B8 release];

    if (**___stack_chk_guard != var_30) {

            __stack_chk_fail();

    }

    return;
}
```

它实现了几个回调方法，不管它，在 Hopper 中搜索 `completeTransaction`，并生成伪代码：

```objective-c
/* @class SiShiPurchaseHelper */

-(void)completeTransaction:(void *)arg2 {

    r14 = self;

    rax = [arg2 retain];

    r15 = rax;

    rax = [rax payment];

    rax = [rax retain];

    r12 = rax;

    rax = [rax productIdentifier];

    rax = [rax retain];

    [rax release];

    [r12 release];

    if (rax != 0x0) {

            [r14 setCurrentTransaction:r15];

            //关键方法：

            [r14 purchaseSuccess];

            [r14 bornWenYuShan];

    }

    [r15 release];

    return;
}
```

找到两个疑似关键方法：

```objective-c
[r14 purchaseSuccess];

[r14 bornWenYuShan];
```

经过阅读代码，找到最终设置用户身份的函数 `bornWenYuShan`：

```objective-c
/* @class SiShiPurchaseHelper */

-(void)bornWenYuShan {

    [self setIsVip:0x1];

    rax = [NSUserDefaults standardUserDefaults];

    rax = [rax retain];

    [rax setBool:0x1 forKey:@"kSiShiIsVipString"];

    [rax release];

    rbx = [[NSNotificationCenter defaultCenter] retain];

    [rbx postNotificationName:*0x1000f52b0 object:0x0];

    [rbx release];

    return;
}
```

了解 APL 开发的童鞋都知道，`[NSUserDefaults standardUserDefaults]` 是保存用户信息的存储接口。

`[rax setBool:0x1 forKey:@"kSiShiIsVipString"];`

写入值为 YES。

经过以上分析，只要能写入这段代码，重启软件应该就可以实现付费破解了。

整理思路：

如果我们要实现 `completeTransaction` 的调用，前面要修改多个 if 校验的逻辑，实在是麻烦。有没有简单有效的办法呢？

既然找到了 `[SiShiPurchaseHelper bornWenYuShan]` 关键函数，那我们在点击“恢复购买”的按钮时直接执行它不就可以了？这样省去了多处修改，也易于操作和验证。

回到函数 `[SiShiPurchaseHelper startRestore:failedBlock:]` 的位置：

掐头去尾，只保留开始的入栈和出栈部分，也就是：

```assembly
0000000100012910         push       rbp                                         ; Objective C Implementation defined at 0x10010f4d0 (instance method), DATA XREF=0x10010f4d0
0000000100012911         mov        rbp, rsp
0000000100012914         push       r15
0000000100012916         push       r14
0000000100012918         push       r13
000000010001291a         push       r12
000000010001291c         push       rbx
000000010001291d         push       rax

......中间的代码全部 NOP 掉

00000001000129bd         pop        rbx
00000001000129be         pop        r12
00000001000129c0         pop        r13
00000001000129c2         pop        r14
00000001000129c4         pop        r15
00000001000129c6         pop        rbp
00000001000129c7         jmp        rax
```

NOP 后，在第九行编写代码片，上面都加了注释，便于理解：

```assembly
// r14 = self

000000010001291e         mov        r14, rdi

// rbx = _objc_msgSend_1000f4360

0000000100012921         mov        rbx, qword [_objc_msgSend_1000f4360]

// msgSend 函数的第一个参数：rsi = bornWenYuShan

0000000100012928         mov        rsi, qword [0x100142d48]

// msgSend 函数的第二个参数：rdi = r14

000000010001292f         mov        rdi, r14

0000000100012932         call       rbx

// 等于执行 msgSend(self,bornWenYuShan)
```

最终代码如图：

![](https://bbs.kanxue.com/upload/attach/202112/942743_FT5HDYABHJPC9U6.jpg)

调用 Hopper 生成可执行文件，覆盖源文件 `/Applications/Live Wallpaper.app/Contents/MacOS/Live Wallpaper` 即可。

最后我们退出软件后发现打不开了？是代码改的有问题吗？并不是。

因为 APL 的所有软件必须签名，我们导出的二进制文件必须重新签名才可以执行。进入到 `/Applications/` 目录，执行以下命令进行签名即可：

```bash
sudo codesign --sign - --force --deep ./Live Wallpaper.app
```

有关签名的内容，大家可自行 Google 脑补。
