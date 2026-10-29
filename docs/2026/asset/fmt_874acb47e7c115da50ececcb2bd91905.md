
# [分享]ios objc 方法调用记录插件：iTracer v1.0

如果你想逆向某些app的调用流程或者系统app的一些功能的私有framework class api 调用流程，可以试试此工具

只需要配置需要挂接的类名和app名，就可以实时追踪相关功能的调用流程。支持批量 hook n多个类名

## 一、安装方法：

### 1. 配置需要挂接的应用，相关参数：

在mac下通过xcode打开 `iTracer.plist`，就改 `filter` 下 `bundles` 列表，设置需要挂接的app名，具体设置可以参考

`mobilesubstrate`的相关文档：57eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3K9r3!0F1k6h3c8W2N6Y4N6A6K9$3W2Q4x3X3g2F1k6i4c8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8V1#2G2j5X3W2D9k6g2y4#2j5Y4y4@1M7X3q4@1k6b7`.`.

例如要挂接`SpringBoard`（挂接这个一般比较危险，做好会白苹果的准备 = =！，真白了也没事：同时按开机键+home，之后再按开机+音量+，进入无插件模式，用itools吧对应的插件删了重启就行了）：

```
Filter = {

  Bundles = (com.apple.springboard);

};
```

### 2. 配置需要挂接的 class 的所有 api，例如：

```
Trace = {

  item0 = (UIView),

  item1 = (UIViewController),

  item2 = (UIApplication),

};
```

### 3. 配置是否开启打印api详细参数值功能（这个还有些不稳定，打印某些值会挂掉，不过通常情况下够用了，先凑活着用吧 = =）

`TraceArgs = YES/NO`

如果不巧打印的时候真挂了，就把这个功能禁用，仅仅 `trace` api的调用流程，还是很稳定的。。

### 4. 利用itool等工具，将 `iTracer.dylib` 和 `iTracer.plist` 放到系统 `mobilesubstrate` 插件目录：

`/Library/MobileSubstrate/DynamicLibraries`

### 5. 然后重启`SpringBoard`（可以通过`PPHelper`注销、或者ssh登上去 `killall -9 SpringBoard`）

## 二、下面是我 trace PPHelper的一些log，注：log 的实际输出在： Xcode-Organizer-Console 中：

```
Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _addSubview:positioned:relativeTo:]: <UIView: 0x95948d0; frame = (0 0; 0 0); alpha = 0.4; hidden = YES; userInteractionEnabled = NO; layer = <CALayer: 0x9594900>>: -1: null

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView superview]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView window]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView window]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _shouldTryPromoteDescendantToFirstResponder]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _isAncestorOfFirstResponder]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView description]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView frame]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView transform]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView clipsToBounds]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView layer]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView alpha]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView isHidden]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView isOpaque]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView autoresizingMask]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView autoresizesSubviews]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView isUserInteractionEnabled]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView tag]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView willMoveToSuperview:]: <SogouToolBarRootView: 0x9592560; frame = (0 0; 0 0); hidden = YES; layer = <CALayer: 0x9592660>>

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _unsubscribeToScrollNotificationsIfNecessary:]: null

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _makeSubtreePerformSelector:withObject:]: <type(:)>: null

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _makeSubtreePerformSelector:withObject:withObject:copySublayers:]: <type(:)>: null: null: 0

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _invalidateSubviewCache]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView actionForLayer:forKey:]: <CALayer: 0x9592660>: sublayers

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView description]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView frame]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView transform]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView clipsToBounds]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView layer]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView alpha]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView isHidden]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView isOpaque]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView autoresizingMask]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView autoresizesSubviews]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView isUserInteractionEnabled]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView tag]

Dec 11 10:50:46 unknown PPHelper[3090] <Warning>: [iTracer]: [UIView _subscribeToScrollNotificationsIfNecessary:]: <SogouToolBarRootView: 0x9592560; frame = (0 0; 0 0); hidden = YES; layer = <CALayer: 0x9592660>>
```

再比如我 trace iTunes-U的一些log：

```
Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore init]

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore activeAccount]

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore _reloadAccountsIfNeeded]

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore _setAccounts:]: (

            "<SSAccount: 0x1108d780>: (1096041240, xxxxxx@gmail.com)",

            "<SSAccount: 0x11091c40>: (1264698560, xxxxxx@gmail.com)"

        )

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore activeAccount]

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore _reloadAccountsIfNeeded]

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore activeAccount]

Dec 11 11:03:56 unknown iTunesU[3393] <Warning>: [iTracer]: [SSAccountStore _reloadAccountsIfNeeded]

Dec 11 11:03:57 unknown wifid[22] <Error>: WiFi:[376887837.256361]: Client itunesstored is background application

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManager _loadDownloadKindsUsingNetwork]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManager _copyDownloadKindsUsingNetwork]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManager _sendObserverConnection]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManagerOptions downloadKinds]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManager _newOptionsDictionary]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManagerOptions persistenceIdentifier]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManagerOptions shouldFilterExternalOriginatedDownloads]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManagerOptions prefetchedDownloadProperties]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManagerOptions prefetchedDownloadExternalProperties]

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManager _handleMessage:fromServerConnection:]: <type(^v)>: <type(^_xpc_connection_s=)>

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManagerOptions downloadKinds]

>

Dec 11 11:03:57 unknown iTunesU[3393] <Warning>: [iTracer]: [SSDownloadManager _handleMessage:fromServerConnection:]: <type(^v)>: <type(^_xpc_connection_s=)>

Dec 11 11:04:00 unknown iTunesU[3393] <Warning>: [iTracer]: [SSSoftwareUpdatesRequest handleFinishResponse:error:]: {

            0 = "238CA706-D2B3-4B99-9CDE-7589471CBF4E";

            2 =     (

                        {

                    "artist-name" = "Chongqing Rumtel Communication Technology Co.,Ltd";

                    "artwork-urls" =             (

                                        {

                            "box-height" = 57;

                            "box-width" = 57;

                            "needs-shine" = 0;
```
