
# [原创] 给 PC WX 加个按钮

# PC Wechat 界面加按钮

作者：御风

日期：2020年4月18日

描述：此方法可快速在微信界面上添加按钮，非 Duilib

版本：不限

[TOC]

## 1. 实现效果

![](upload/attach/202004/654096_4EDPYBP434968G5.png)

![](upload/attach/202004/654096_6X7ZB2J9489B7Z7.png)

![](upload/attach/202004/654096_4QW5FHVKRUEXM8G.png)

![](upload/attach/202004/654096_H8CY56CGE9F6NGH.png)

## 2. DLL 注入

DLL 注入的方式可以保证插件与 `Wechat.exe` 为同一进程，也无需为多开管理烦恼

具体的 DLL 注入方法这里不多讲了，教程很多

## 3. 具体实现

### 3.1. 插件功能窗口

DLL 注入成功后，创建插件功能窗口

窗口创建后先隐藏，任务栏不可视，无边框风格

![](upload/attach/202004/654096_45K3PJXMP8G9WD6.png)

然后循环调用 `GetModuleHandle("WechatWin.dll")` 检测 `WechatWin.dll` 是否已加载成功，可以设置一个超时时间

当检测到 `WechatWin.dll` 加载完成后，可以 Hook 登录成功的事件

在登录成功的时候，等待微信聊天界面（“微信”, `WeChatMainWndForPC`）

可以根据自身进程 `pid` 去获取自身窗口句柄，保证窗口句柄不会取错

当微信聊天界面加载完成后：

1、对微信聊天界面窗口 `SetWindowLong` 接管消息

2、调用 `SetParent` 把插件功能窗口嵌入到微信聊天界面窗口

`SetWindowLong` 可以监听到微信聊天界面窗口的点击事件，我们只需要判断一下坐标：

![](upload/attach/202004/654096_WABWKVSKJ54V3NH.png)

当监听到 `WM_LBUTTONDOWN` 消息时，就可以根据点击坐标得到点击的导航按钮索引

当监听到 `WM_SIZE` 消息时，更改插件窗口大小

### 3.2. 插件导航按钮

如果你有闲心，可以在微信聊天界面窗口的重绘窗口消息中用 GDI 绘制按钮，再根据鼠标移动消息加上热点效果

像我这么懒的，选择再嵌入一个按钮窗口：

```
窗口类名：WeChatPluginBtnForPC
窗口标题：无
```

这个按钮窗口上，放一个图片控件，用来展示鼠标点击和热点效果

窗口结构如下：

![](upload/attach/202004/654096_YSGD5W9SAQNGPMA.png)
