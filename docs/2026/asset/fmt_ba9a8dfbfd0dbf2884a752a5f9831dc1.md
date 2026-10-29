
```java
/**

     * 微信聊天界面

     *

     * @param applicationContext

     * @param classLoader

     */



    private void hookWxChatUIMM(final Context applicationContext, final ClassLoader classLoader) {

        XposedHelpers.findAndHookMethod("com.tencent.mm.ui.base.MMPullDownView",

                classLoader,

                "onLayout",

                boolean.class,

                int.class,

                int.class,

                int.class,

                int.class,

                new XC_MethodHook() {

                    @Override

                    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

                        super.beforeHookedMethod(param);

                        ViewGroup mMPullDownView = (ViewGroup) param.thisObject;

//                        if (mMPullDownView.getVisibility() == View.GONE) return;

                        for (int i = 0; i < mMPullDownView.getChildCount(); i++) {

                            View childAt = mMPullDownView.getChildAt(i);

                            if (childAt instanceof ListView) {

                                final ListView listView = (ListView) childAt;

                                final ListAdapter adapter = listView.getAdapter();

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

                                                View view = (View) param.args[1];

                                                JSONObject itemData = null;

//                                                LogUtils.i(position, view.toString());

                                                if (position < adapter.getCount()) {

                                                    itemData = JSON.parseObject(JSON.toJSONString(adapter.getItem(position)), JSONObject.class);

                                                    int itemViewType = adapter.getItemViewType(position);

//                                                    LogUtils.i(itemViewType);

                                                    //经过以上代码可以知道    itemViewType == 1的时候打印的值是正常对话列表的值

                                                    if (itemData != null && (view != null && view.toString().contains("com.tencent.mm.ui.chatting.viewitems.p"))) {

//                                                        if (itemData != null && itemViewType == 1 && (view != null && view.toString().contains("com.tencent.mm.ui.chatting.viewitems.p"))) {

                                                        String field_msgId = itemData.getString("field_msgId");

                                                        WxChatInvokeMsg wxChatInvokeMsg = WxChatInvokeMsgDB.queryByMsgId(applicationContext, field_msgId);

                                                        ViewGroup itemView = (ViewGroup) view;

                                                        View itemViewChild = itemView.getChildAt(0);

                                                        Object tag = itemViewChild.getTag(R.id.wx_parent_has_invoke_msg);

                                                        TextView textView;

                                                        if (tag == null) {

                                                            textView = new TextView(applicationContext);

                                                            textView.setGravity(Gravity.CENTER);

                                                            textView.setLayoutParams(new LinearLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT));

                                                            itemViewChild.setTag(R.id.wx_parent_has_invoke_msg, textView);

                                                            textView.setId(R.id.wx_invoke_msg);

                                                            itemView.addView(textView);

                                                        } else {

                                                            textView = (TextView) itemViewChild.getTag(R.id.wx_parent_has_invoke_msg);

                                                        }

                                                        textView.setText("");

                                                        textView.setVisibility(View.GONE);

                                                        if (wxChatInvokeMsg != null) {

                                                            textView.setPadding(10, 5, 10, 5);

                                                            textView.setBackgroundDrawable(ShapeUtil.getCornerDrawable());

                                                            textView.setTextColor(Color.parseColor("#666666"));

                                                            View msgView = itemView.getChildAt(3);

                                                            RelativeLayout.LayoutParams lp = (RelativeLayout.LayoutParams) textView.getLayoutParams();

                                                            lp.addRule(RelativeLayout.CENTER_HORIZONTAL);

                                                            lp.addRule(RelativeLayout.BELOW, msgView.getId());

                                                            lp.bottomMargin = 50;

                                                            textView.setText("这小子想撤回这个消息");

                                                            textView.setVisibility(View.VISIBLE);

                                                            LogUtils.i(position, view, itemViewType, itemData.toJSONString());

                                                        }

                                                    }

                                                }

//



                                            }



                                        });

                                break;



                            }

                        }

                    }

                });

    }
```

记住 hook 这个 `listview` 的时候会遇到我文章片头说的问题，导致每个 item 的 `view` 对应错误，所以我加了过滤标识。

以上内容仅仅作为学习交流。代码很简单，乐趣在于找到这些代码位置的过程。

[垃圾代码已上传](https://bbs.kanxue.com/elink@75dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2K9h3&6Y4h3V1c8Q4x3V1k6i4P5p5c8W2L8h3)9`.)

[[原创]Xposed第四课(微信篇) 朋友圈点赞 (1)](https://bbs.pediy.com/thread-226951.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-2-2 10:52 被 admin 编辑，原因：图片本地化
