

# iOS 逆向：微信抢红包 + 微信运动步数修改 + UI 界面新增控件

@[TOC]（iOS 逆向：微信抢红包 + 微信运动步数修改 + UI 界面新增控件）

## 开篇

捣鼓几天搞出来的微信修改版，支持自动抢红包和修改微信运动步数，并且在设置里面可以开关功能。

## 需求与最终效果

![](upload/attach/202006/895501_6M25WTANRZN7A8H.jpg)

![](upload/attach/202006/895501_K5K2VDYQ3D98TJZ.jpg)

## 环境要求与即将使用的工具

| 环境       | 版本                                        |
| :--------- | :------------------------------------------ |
| 操作系统   | MacOSX Catalina 10.15.4（版本太新了不太好用，很多工具用不了，我后面打算降级） |
| 手机系统   | Iphone7 IOS11                               |
| mac 上面的 [theos](elink@66eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3g2G2M7#2)9J5c8Y4c8Z5k6h3!0K6) | 最新版                                      |
| xcode      | 11.5                                        |
| MonkeyDev  | -                                           |

### 工具介绍

`MonkeyDev` 集成在 `xcode` 上面，可以快速开发 hook 的代码，链接到 Mach-O 文件，支持修改 ipa 后的免越狱安装。配合 `lldb` 的调试效率高。

## 实现过程

### 页面 UI 新增控件

新建一个 `MonkeyDev` 项目，我的是 `WeChatDemo`。

先把砸壳后的微信 ipa 拖拽进工程中的 `TargetApp` 目录。

真机调试执行 run 编译运行至手机，成功的话会看到手机多出一个微信，并且可以利用 `xcode` 来调试微信了！

首先完成设置页面 UI 新增“自动抢红包”和微信步数输入框的控件功能。

打开微信设置页面，`xcode` 打开 Debug View Hierarychy 查看层级。

![](upload/attach/202006/895501_RDCXPZ6SGGB5T68.jpg)

查找表格布局的数据源，发现 Data Source 是 `WCTableViewManager` 这个类，所以我们的新增控件功能要在这个类注入方法。

打开 class-dump 好之后的头文件 `WCTableViewManager.h`。

开发过 iOS 的都知道，表格的实现需要实现 `UITableViewDelegate`、`UITableViewDataSource` 协议，用下面三个方法来控制 cell：

```objectivec
// 表格每一行的遍历
- (id)tableView:(id)arg1 cellForRowAtIndexPath:(NSIndexPath*)arg2;
// 每组有多少行
- (long long)tableView:(id)arg1 numberOfRowsInSection:(long long)arg2;
// 表格有多少组
- (long long)numberOfSectionsInTableView:(id)arg1;
```

注入这三个方法：

我新增了两个控件，所以 `numberOfSections` 加 2：

```objectivec
// 表格有多少组
- (long long)numberOfSectionsInTableView:(id)arg1 {
    NSLog(@"num2===%@", arg1);
    if ([[[[arg1 nextResponder] nextResponder] nextResponder] isKindOfClass:%c(MoreViewController)]) {
        NSLog(@"这是设置页面");
        return %orig + 2;
    }
    return %orig;
}
```

我的每组是一行，所以第 4 个 section 和第 5 个 section 返回一行：

```objectivec
// 每组有多少行
- (long long)tableView:(id)arg1 numberOfRowsInSection:(long long)arg2 {
    NSLog(@"num===%lld", arg2);
    // 红包设置的行
    if ([[[[arg1 nextResponder] nextResponder] nextResponder] isKindOfClass:%c(MoreViewController)]) {
        NSLog(@"红包section的行数");
        if (arg2 == 4) {
            return 1;
        } else if (arg2 == 5) {
            // 微信运动的行数
            return 1;
        }
    }
    return %orig;
}
```

以下是具体每一个 cell 的实现：

```objectivec
// 表格每一行的遍历
- (id)tableView:(id)arg1 cellForRowAtIndexPath:(NSIndexPath*)arg2 {
    NSLog(@"indexPath===%@", arg2);
    if ([[[[arg1 nextResponder] nextResponder] nextResponder] isKindOfClass:%c(MoreViewController)]) {
        if (arg2.section == 4) {
            NSLog(@"红包的行cell");
            UITableViewCell *cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:@"mycell"];
            cell.textLabel.text = @"自动抢红包";
            cell.backgroundColor = [UIColor whiteColor];
            UISwitch *sw = [[UISwitch alloc] init];
            sw.on = [Comm confIsRedEnvelopeSwitchEnable];
            [sw addTarget:self action:@selector(redEnvelopeSwitchChange:) forControlEvents:UIControlEventValueChanged];
            cell.accessoryView = sw;
            return cell;
        } else if (arg2.section == 5) {
            [MyMoreViewController setTableViewObject:arg1];
            return [MyMoreViewController createWCSportTextField];
        }
    }
    return %orig;
}
```

其中：

- `sw.on = [Comm confIsRedEnvelopeSwitchEnable];` 是我新增的类方法，获取配置文件中的是否开启自动红包的配置。
- `[MyMoreViewController createWCSportTextField];` 是指创建微信步数控件，具体代码这里我不列出来了。

### 抢红包功能实现

网络上有很多分析拆红包代码流程，这里简述流程。

首先需要找到微信消息接收入口，就是 `CMessageMgr` 这个类的 `onNewSyncAddMessage` 方法，普通消息、表情、红包等等大部分消息都走这个方法。

然后判断消息类型（`m_uiMessageType`），为 49 时确定为微信红包消息。

调用下面这个方法告诉微信服务器将要拆红包的请求：`[redEnvelopesLogicMgr ReceiverQueryRedEnvelopesRequest:mutableDict]`。

`mutableDict` 里面的字典数据就是我们要拼装的数据。

`ReceiverQueryRedEnvelopesRequest` 调用成功后，微信会回调触发 `[WCRedEnvelopesLogicMgr OnWCToHongbaoCommonResponse]` 这个方法，这个方法能获取到 `timingIdentifier` 这个参数，然后我们再调用 `[redEnvelopesLogicMgr OpenRedEnvelopesRequest:redParameter.params]`。

最终实现收获红包的调用。`redParameter.params` 就是请求的参数。

下面代码实现：

```objectivec
%hook CMessageMgr
// 收到微信消息
- (void)onNewSyncAddMessage:(id)arg1 {
    %orig;
    [MyRedEnvelopesProcc onNewSyncAddMessageProcc:arg1];
}
%end
```

`[MyRedEnvelopesProcc onNewSyncAddMessageProcc:arg1]` 是我写的类方法。

```objectivec
// 红包处理类
@implementation MyRedEnvelopesProcc

+ (void)onNewSyncAddMessageProcc:(id)arg1 {
    CMessageWrap *wrap = arg1;
    NSLog(@"红包==========%@\n%@", arg1, [arg1 class]);
    NSLog(@"类型==========%d\n", wrap.m_uiMessageType);

    if (wrap.m_uiMessageType != 49) {
        NSLog(@"不是红包消息");
        return;
    }

    if (![Comm confIsRedEnvelopeSwitchEnable]) {
        NSLog(@"不启用抢红包功能");
        return;
    }

    // 收到红包消息
    NSString *nsFromUsr = [wrap m_nsFromUsr];

    // 只有是红包消息类型才会有 m_oWCPayInfoItem
    WCPayInfoItem *payInfoItem = [wrap m_oWCPayInfoItem];
    NSLog(@"payInfoItem==========%@\n", payInfoItem);

    if (payInfoItem == nil) {
        return;
    }

    NSString *m_c2cNativeUrl = [payInfoItem m_c2cNativeUrl];
    if (m_c2cNativeUrl == nil) {
        NSLog(@"m_c2cNativeUrl是nil !!!!!!!!!");
        return;
    }

    NSInteger length = [@"wxpay://c2cbizmessagehandler/hongbao/receivehongbao?" length];
    NSString *subString = [m_c2cNativeUrl substringFromIndex:length];
    NSDictionary *dict = [objc_getClass("WCBizUtil") dictionaryWithDecodedComponets:subString separator:@"&"];
    NSMutableDictionary *mutableDict = [NSMutableDictionary dictionary];
    [mutableDict setObject:@"1" forKey:@"msgType"];

    NSString *sendId = dict[@"sendid"];
    [mutableDict safeSetObject:sendId forKey:@"sendId"];

    NSString *channelId = dict[@"channelid"];
    [mutableDict safeSetObject:channelId forKey:@"channelId"];

    CContactMgr *service = [[objc_getClass("MMServiceCenter") defaultCenter] getService:[objc_getClass("CContactMgr") class]];
    CContact *contact = [service getSelfContact];
    NSString *displayName = [contact getContactDisplayName];
    [mutableDict safeSetObject:displayName forKey:@"nickName"];

    NSString *headerImg = [contact m_nsHeadImgUrl];
    [mutableDict safeSetObject:headerImg forKey:@"headImg"];

    id nativeUrl = [payInfoItem m_c2cNativeUrl];
    [mutableDict safeSetObject:nativeUrl forKey:@"nativeUrl"];

    // 之前获取 m_nsUsrName 的方式只有在聊天窗口才有能获得变量，不人性的做法，改
    /*
    MMMsgLogicManager *logicManager = [[objc_getClass("MMServiceCenter") defaultCenter] getService:[objc_getClass("MMMsgLogicManager") class]];
    BaseMsgContentLogicController *logicController = [logicManager GetCurrentLogicController];
    id m_contact = [logicController m_contact];
    id sessionUserName = [m_contact m_nsUsrName];
    // wxid_wps5gzsp30an32
    */
    NSString *sessionUserName = [wrap m_nsFromUsr];
    [mutableDict safeSetObject:sessionUserName forKey:@"sessionUserName"];

    if ([nsFromUsr hasSuffix:@"@chatroom"]) {
        // 群红包
        [mutableDict safeSetObject:@"0" forKey:@"inWay"]; // 0:群聊，1：单聊
    } else {
        // 个人红包
        [mutableDict safeSetObject:@"1" forKey:@"inWay"]; // 0:群聊，1：单聊
    }

    [mutableDict safeSetObject:@"0" forKey:@"agreeDuty"];

    if (sendId.length > 0) {
        SPRedParameter *redParameter = [[SPRedParameter alloc] init];
        redParameter.params = mutableDict;
        [[SPRedManager sharedInstance] addParams:redParameter];
    }

    NSLog(@"SPRedManager------mutableDict=%@", mutableDict);
    WCRedEnvelopesLogicMgr *redEnvelopesLogicMgr = [[objc_getClass("MMServiceCenter") defaultCenter] getService:[objc_getClass("WCRedEnvelopesLogicMgr") class]];
    [redEnvelopesLogicMgr ReceiverQueryRedEnvelopesRequest:mutableDict];
}
@end
```

主要功能是调用下面这个方法告诉微信将要拆红包的请求：`[redEnvelopesLogicMgr ReceiverQueryRedEnvelopesRequest:mutableDict]`。

然后再注入这个方法：

```objectivec
%hook WCRedEnvelopesLogicMgr
// [redEnvelopesLogicMgr ReceiverQueryRedEnvelopesRequest:mutableDict] 请求后的响应，拆红包请求的回调
- (void)OnWCToHongbaoCommonResponse:(id)hongBaoRes Request:(id)hongBaoReq {
    %orig;
    [MyRedEnvelopesProcc OnWCToHongbaoCommonResponseProcc:hongBaoRes Request:hongBaoReq];
}
%end
```

`[MyRedEnvelopesProcc OnWCToHongbaoCommonResponseProcc:hongBaoRes Request:hongBaoReq]` 是我自己写的方法，最终实现抢红包的调用。
