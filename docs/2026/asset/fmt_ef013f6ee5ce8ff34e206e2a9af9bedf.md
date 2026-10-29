
# [原创]暴力破解某直播平台收费房

## 目录

[toc]

### 内容摘要

- 本贴仅限学习交流使用，切勿用于非法行径，否则后果自付
- 周末适合在家写写文章，前段时间手上拿到一款直播app，发现里面有很多房间都是收费的，需要充金币才能看，所以就有了如下文章

### 分析思路

- 工具

  1. `objection`
  2. `frida`

- 分析步骤

  1. 先放到`jadx`中发现该app是360的壳
  2. 直接脱，这里就不多介绍了，直接脚本就可以脱，具体可以看我上篇文章
  3. 查找关键点，直接打开app，跳到收费视频界面
  4. 知道关键点了，我们就需要快速定位到关键点，我们都知道弹窗一般是通过系统的`android.app.Dialog`类中的`show()`方法展示出来的，这里就hook该方法，然后调用栈，直接用`objection`安排上

  ```
  android hooking watch class_method android.app.Dialog.show --dump-args --dump-backtrace --dump-return
  ```

  5. 很快调用栈就轻而易举的定位到了
  6. 这里有三个点，我们先选择其中第2个点和第3个点去看代码分析，因为这两个点离逻辑层最近，先看第二个点的代码
  7. 发现最终都会调用`this.payLiveBlackBgView`这个类，然后看函数名字猜测这个是控制时间显示的方法，然后就是验证自己的猜想了
  8. 另外弹窗根据之前`objection`打印的`关键点3`也找到了对应的类
  9. 自此所有点都找到了，附上代码验证了

  ```js
  Java.perform(function(){
  var LiveLayoutViewerExtendActivity = Java.use("com.xxx.live.activity.room.LiveLayoutViewerExtendActivity");
  var PayLiveBlackBgView = Java.use("com.xxx.pay.appview.PayLiveBlackBgView")
  var LiveJoinPayDialog = Java.use("com.xxx.pay.dialog.LiveJoinPayDialog");
  // hook 收费弹窗 显示
  LiveJoinPayDialog.show.implementation = function(){
  console.log("LiveJoinPayDialog.show: ");
  }
  // 设置播放时间
  PayLiveBlackBgView.setProview_play_time.implementation = function(a1) {
  console.log("setProview_play_time ", a1);
  this.setProview_play_time(1000000000)
  }
  // 设置视频 图像显示
  PayLiveBlackBgView.setIs_only_play_voice.implementation = function(a1){
  console.log("setIs_only_play_voice: ", a1);
  return this.setIs_only_play_voice(0);
  }
  })
  ```

### 效果展示

- 至此简单的暴力破解已经结束，效果也还可以
- 附几张效果图有些房间比较yellow，找了个稍微不yellow点的
- PS 最后发现大部分都是录播视频，都是套路啊

### 协议破解

- 作为一个逆向的爱好者，往往通过暴力破解，感觉没啥意思，最终还是要破解协议，协议才是一个app的灵魂所在，直接安排上
- 说到协议，我们脑子第一个想到的就是抓包，`charles + postern`安排上，后面发现开了vpn后，app直接起不来了，然后又试了先让app起来后然后在开vpn，依然消息发不出去，看来app肯定对vpn做了检测啥的
- 既然不能抓包，此路不通换条路呗，那就从代码入手呗，直接回到我们的第一张图的`关键点1`,直接hook `AppHttpUtil`那个类，看调用栈，发现下面的关键函数，则直接安排上
- 然后我们在看调用`requestIndex`方法的上层，发现这个是请求的源头
- 既然找到关键点了，那我们去构造这个请求来一次主动调用，看看效果
- 这里我只写了两个消息的主动调用 房间列表，房间详情 其他接口都类似，就不举例了

```js
Java.perform(function() {
Java.openClassFile("/data/local/tmp/gson.dex").load();
var gson = Java.use("com.google.gson.Gson").$new();
var CommonInterface = Java.use("com.xxx.live.common.CommonInterface");
var AppRequestCallback = Java.use("com.xxx.xxx.http.AppRequestCallback");
var LiveTabHotView4 = Java.use("com.xxx.live.appview.main.LiveTabHotView$4");
var Json = Java.use("com.alibaba.fastjson.JSON");
var Index_indexActModel = Java.use("com.xxx.live.model.Index_indexActModel");
var LiveRoomModel = Java.use("com.xxx.live.model.LiveRoomModel");
var a = 1;
//对于 new class<class> 的构造 有点困惑
//CommonInterface.requestIndex(1, this.mSex, 0, this.mCity, new AppRequestCallback<Index_indexActModel>() {
//TODO :方法1 : CommonInterface.requestIndex(1, 0, 0, "热门", AppRequestCallback$new());
// 大厅房间列表获取
Java.choose("com.xxx.live.appview.main.LiveTabHotView$4", {
onMatch: function (instance) {
if(a == 1){
console.log("LiveTabHotView$4 .....", instance);
CommonInterface.requestIndex(1, 0, 0, "热门", instance);
a = 2;
}
}, onComplete: function () {
}
})
LiveTabHotView4.onSuccess.implementation = function(a){
var result = a.getDecryptedResult();
var resultModel = Json.parseObject(result, Index_indexActModel.class);
var roomList = Java.cast(resultModel, Index_indexActModel).getList();
console.log("size : ", roomList.size(), roomList.get(0))
for(var i = 0; i < roomList.size(); i++){
var LiveRoomModelInfo = Java.cast(roomList.get(i), LiveRoomModel);
console.log("roominfo: ", i, " ", gson.toJson(LiveRoomModelInfo));
}
return this.onSuccess(a);
}
var b = 1;
var LiveBusiness2 = Java.use("com.xxx.live.business.LiveBusiness$2");
// 下面的消息接口 也是大同小异
// 指定房间信息获取 (可以拿到 直播的 url)
Java.choose("com.xxx.live.business.LiveBusiness$2", {
onMatch: function (instance) {
if(b == 1){
console.log("LiveBusiness$2 .....", instance);
CommonInterface.requestRoomInfo(房间id, 0, "", instance);
b = 2;
}
}, onComplete: function () {
}
})
var App_get_videoActModel = Java.use("com.xxx.live.model.App_get_videoActModel")
LiveBusiness2.onSuccess.implementation = function(a){
var result = a.getDecryptedResult();
var resultModel = Json.parseObject(result, App_get_videoActModel.class);
var roomDetail = Java.cast(resultModel, App_get_videoActModel);
//这里只打印了 直播地址 (无用数据太多了) 拿到直播地址后 可以直接用 VLC 播放器打开 播放
// play_url 免费房 是用这个字段
// getPreview_play_url 收费房用这个字段
console.log("roomDetail: play_url: ", roomDetail.getPlay_url(), " getPreview_play_url: ", roomDetail.getPreview_play_url());
//console.log("roomDetail: ", gson.toJson(roomDetail));
return this.onSuccess(a);
}
```

- 接下来就要找加密算法了，然后实现`脱机`,这个过程中有遇到个小插曲，就是发现该app域名指向的地址是竟然本地的地址，很奇怪，记得当时我还问 `r0ysue`大佬呢，后来我发现了app中的秘密
- 就是该app竟然用了阿里的游戏盾
- 既然用了sdk的话，所以就想到去官网下载sdk玩下，后来发现貌似没有sdk，demo下载，而且大部分逻辑处理都在so层，功力不够，后面就放弃没去搞了，后来在 `r0ysue`的星球里看到有人在问，用adb端口转发，这个给了我启发，后来百度了下adb端口转发
- 既然这样的话，那我们是不是可以搞成，半脱机呢 后来证明是可以的，安排上
- 这里面涉及一些加密算法，都是用了标准的算法`aes/md5`,所以数据构造起来就很简单了
- 打开app查找本地端口，执行端口转发
- 然后用python脚本安排上
- 跑起来，效果还可以
- 这里只写了一个接口，其他类似,就不重复了，到此一个半脱机的脚步算写完了
- 吃个压压惊，写文章好累啊，写了快一天了

### 最后总结

- 该app唯一的难点就是用了阿里的游戏盾，其他一些加密算法都是标准算法，直接照抄源码即可
- 这游戏盾后面有时间的话，在去看看吧，毕竟大厂写的，弟弟表示很畏惧
- adb的端口转发功能是学到了，以前只会用在IDA调试中，现在发现原来也可以这么用
- 另外关于这个直播流，有几点疑问，就是我拿到直播流的地址后，随便用什么客户端都可以播放，难道后端是不校验前端的吗？如果要校验的话，是怎么去校验的呢？
- 本人水平很菜，有哪里写不对的话，欢迎大佬们指点批评啊
- 本贴仅限学习交流使用，切勿用于非法行径，否则后果自付
- 如有侵权，请联系作者删除
