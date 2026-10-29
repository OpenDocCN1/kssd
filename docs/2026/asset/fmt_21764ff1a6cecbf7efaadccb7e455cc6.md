
```
/**
 * hook 朋友圈
 */
@Override
public void hookWxMoments() {
XposedHelpers.findAndHookMethod("com.tencent.mm.plugin.sns.ui.bb",
mClassLoader,
"onCreate",
new XC_MethodHook() {
@Override
protected void afterHookedMethod(MethodHookParam param) throws Throwable {
super.afterHookedMethod(param);
Field mat = XposedHelpers.findFieldIfExists(param.thisObject.getClass(), "mActivity");
final Activity mActivity = (Activity) mat.get(param.thisObject);
Field odm = XposedHelpers.findFieldIfExists(param.thisObject.getClass(), "odm");
ListView mlv = (ListView) odm.get(param.thisObject);
Class<?> mlvSuperClass = mlv.getClass().getSuperclass();
XposedHelpers.findAndHookMethod(mlvSuperClass,
"setAdapter",
ListAdapter.class,
new XC_MethodHook() {
@Override
protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
super.beforeHookedMethod(param);
final ListAdapter adapter = (ListAdapter) param.args[0];
XposedHelpers.findAndHookMethod(adapter.getClass(),
"getView",
int.class,
View.class,
ViewGroup.class,
new XC_MethodHook() {
@Override
protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
super.beforeHookedMethod(param);
final View view = (View) param.args[1];
if (view != null) {
//fl 第一个view是图片 第二个view是朋友圈内容
final ViewGroup fl = (ViewGroup) view;
view.setOnLongClickListener(new View.OnLongClickListener() {
@Override
public boolean onLongClick(View v) {
final List<WxContact> wxContacts = WxContactDB.queryAll(mContext);
LogUtils.i(JSON.toJSONString(wxContacts));
Dialog show = WxMomentContactDialog.instance(mActivity).build(wxContacts, new WxMomentContactDialog.CallBack() {
@Override
public void sure() {
final StringBuffer sb = new StringBuffer();
for (int i = 0; i < wxContacts.size(); i++) {
WxContact contact = wxContacts.get(i);
if (contact.isCheck()) {
sb.append(contact.getField_nickname());
sb.append(",");
}
}
mActivity.runOnUiThread(new Runnable() {
@Override
public void run() {
String s = sb.toString();
if (fl != null && fl.getChildCount() > 1) {
LinearLayout msgLinear = (LinearLayout) fl.getChildAt(1);
for (int i = 0; i < msgLinear.getChildCount(); i++) {
View mc = msgLinear.getChildAt(i);
String resourceName = mContext.getResources().getResourceName(mc.getId());
if ("com.tencent.mm:id/de_".equals(resourceName)) {
ViewGroup vg = (ViewGroup) mc;
mc.setVisibility(View.VISIBLE);
if (vg.getChildCount() > 0) {
TextView likeView = (TextView) vg.getChildAt(0);
likeView.append(s);
} else {
TextView tv = new TextView(mContext);
tv.setText("");
tv.append(",");
tv.append(s);
tv.setLayoutParams(new LinearLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT));
vg.addView(tv);
}
}
}
}
}
});
}
});
show.show();
show.getWindow().setLayout(ViewUtil.dp2px(mActivity, 300f), ViewUtil.dp2px(mActivity, 400f));
return false;
}
});
}
}
});
}
});
}
});
}
```

效果圖如下

| 效果一 | 效果二 | 效果三 |
| --- | --- | --- |
| ![](upload/attach/201902/3_WQ2ZF4P3WHSDWUF.png) | ![](upload/attach/201902/3_9REWRMTWRFZNKCW.png) | ![](upload/attach/201902/3_GMP73UY4UYFK54K.png) |

有些不完美的地方，联系人需要点击下通讯录才开始保存数据 这个阶段先做到联系人与点赞关联，下次把点赞的联系人与朋友圈消息关联关系一起保存起来。就可以做到下次在进去朋友圈可以看到上次点赞的。后面还可以进行取消以及其他操作

[垃圾代码已上传](<elink@086K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2K9h3&6Y4h3V1c8Q4x3V1k6i4P5p5c8W2L8h3)9`.)>

这次我把我用到的微信版本和genymotion的arm-transfer放进去了 所以体积变大了 哈哈

下一篇 [\[原创\]Xposed第五课(微信篇) 聊天机器人](https://bbs.pediy.com/thread-228441.htm)

上一篇 [\[原创\]Xposed第四课(微信篇) 朋友圈点赞 (1)](https://bbs.pediy.com/thread-226951.htm)

前面做了个小玩意，朋友圈点赞，缺点是只能在当前界面临时显示，并且不能选择好友，返回后在进去就没有了，今天准备将联系人与朋友圈点赞功能关联起来。

首先分析布局结构

![](https://bbs.kanxue.com/upload/attach/201902/3_2GRQJ46Y4P2NEVG.png)

根据关键名称进行查找

| `BizContactEntranceView` | `AlphabetScrollBar` |
| --- | --- |
|  |  |

定位到了`com.tencent.mm.ui.contact.AddressUI`

有人想，你光找到这个有个软用，数据库你不找到怎么去保存到你自己的app里面。 大家接着往下看，可能我代码有自己的想法，皮一下很开心O(∩\_∩)O

先上代码

```
@Override

    public void hookContact() {

        XposedHelpers.findAndHookMethod("com.tencent.mm.ui.contact.AddressUI.a",

                mClassLoader,

                "coR",

                new XC_MethodHook() {

                    @Override

                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {

                        super.afterHookedMethod(param);

                        Field oiy = XposedHelpers.findField(param.thisObject.getClass(), "oiy");

                        oiy.setAccessible(true);

                        LogUtils.i(oiy);

                        ListView oiyLv = (ListView) oiy.get(param.thisObject);

                        Class<?> mlvSuperClass = oiyLv.getClass().getSuperclass();

                        LogUtils.i(oiyLv);

                        XposedHelpers.findAndHookMethod(mlvSuperClass,

                                "setAdapter",

                                ListAdapter.class,

                                new XC_MethodHook() {

                                    @Override

                                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

                                        super.beforeHookedMethod(param);

                                        final ListAdapter adapter = (ListAdapter) param.args[0];

                                        LogUtils.i(adapter.toString());

                                        XposedHelpers.findAndHookMethod(adapter.getClass(),

                                                "getView",

                                                int.class,

                                                View.class,

                                                ViewGroup.class,

                                                new XC_MethodHook() {

                                                    @Override

                                                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

                                                        super.beforeHookedMethod(param);

                                                        int position = (int) param.args[0];

                                                        final View view = (View) param.args[1];

                                                        final ViewGroup viewGroup = (ViewGroup) param.args[2];

                                                        LogUtils.i(position, view, viewGroup, JSON.toJSONString(adapter.getItem(position)), adapter.getItem(position).toString());

                                                    }

                                                });

                                    }

                                });

                    }

                });

    }
```

可以得到以下信息
