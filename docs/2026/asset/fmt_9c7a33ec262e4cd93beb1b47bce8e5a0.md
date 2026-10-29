
```java
/**
 * hook 朋友圈
 */
@Override
public void hookWxMoments() {
    XposedHelpers.findAndHookMethod(
        "com.tencent.mm.plugin.sns.ui.bb",
        mClassLoader,
        "onCreate",
        new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);

                Field mat = XposedHelpers.findFieldIfExists(
                    param.thisObject.getClass(),
                    "mActivity"
                );

                final Activity mActivity = (Activity) mat.get(param.thisObject);

                Field odm = XposedHelpers.findFieldIfExists(
                    param.thisObject.getClass(),
                    "odm"
                );

                ListView mlv = (ListView) odm.get(param.thisObject);
                Class<?> mlvSuperClass = mlv.getClass().getSuperclass();

                XposedHelpers.findAndHookMethod(
                    mlvSuperClass,
                    "setAdapter",
                    ListAdapter.class,
                    new XC_MethodHook() {
                        @Override
                        protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                            super.beforeHookedMethod(param);

                            final ListAdapter adapter = (ListAdapter) param.args[0];

                            XposedHelpers.findAndHookMethod(
                                adapter.getClass(),
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
                                            // fl 第一个view是图片 第二个view是朋友圈内容
                                            final ViewGroup fl = (ViewGroup) view;

                                            view.setOnLongClickListener(new View.OnLongClickListener() {
                                                @Override
                                                public boolean onLongClick(View v) {
                                                    final List<WxContact> wxContacts =
                                                        WxContactDB.queryAll(mContext);

                                                    LogUtils.i(JSON.toJSONString(wxContacts));

                                                    Dialog show =
                                                        WxMomentContactDialog.instance(mActivity)
                                                            .build(
                                                                wxContacts,
                                                                new WxMomentContactDialog.CallBack() {
                                                                    @Override
                                                                    public void sure() {
                                                                        final StringBuffer sb =
                                                                            new StringBuffer();

                                                                        for (
                                                                            int i = 0;
                                                                            i < wxContacts.size();
                                                                            i++
                                                                        ) {
                                                                            WxContact contact =
                                                                                wxContacts.get(i);

                                                                            if (contact.isCheck()) {
                                                                                sb.append(
                                                                                    contact.getField_nickname()
                                                                                );
                                                                                sb.append(",");
                                                                            }
                                                                        }

                                                                        mActivity.runOnUiThread(
                                                                            new Runnable() {
                                                                                @Override
                                                                                public void run() {
                                                                                    String s =
                                                                                        sb.toString();

                                                                                    if (
                                                                                        fl != null
                                                                                            && fl.getChildCount()
                                                                                                > 1
                                                                                    ) {
                                                                                        LinearLayout msgLinear =
                                                                                            (LinearLayout)
                                                                                                fl.getChildAt(
                                                                                                    1
                                                                                                );

                                                                                        for (
                                                                                            int i = 0;
                                                                                            i
                                                                                                < msgLinear
                                                                                                    .getChildCount();
                                                                                            i++
                                                                                        ) {
                                                                                            View mc =
                                                                                                msgLinear
                                                                                                    .getChildAt(
                                                                                                        i
                                                                                                    );

                                                                                            String resourceName =
                                                                                                mContext
                                                                                                    .getResources()
                                                                                                    .getResourceName(
                                                                                                        mc.getId()
                                                                                                    );

                                                                                            if (
                                                                                                "com.tencent.mm:id/de_"
                                                                                                    .equals(
                                                                                                        resourceName
                                                                                                    )
                                                                                            ) {
                                                                                                ViewGroup vg =
                                                                                                    (ViewGroup)
                                                                                                        mc;

                                                                                                mc.setVisibility(
                                                                                                    View.VISIBLE
                                                                                                );

                                                                                                if (
                                                                                                    vg.getChildCount()
                                                                                                        > 0
                                                                                                ) {
                                                                                                    TextView likeView =
                                                                                                        (TextView)
                                                                                                            vg.getChildAt(
                                                                                                                0
                                                                                                            );

                                                                                                    likeView.append(
                                                                                                        s
                                                                                                    );
                                                                                                } else {
                                                                                                    TextView tv =
                                                                                                        new TextView(
                                                                                                            mContext
                                                                                                        );

                                                                                                    tv.setText(
                                                                                                        ""
                                                                                                    );
                                                                                                    tv.append(
                                                                                                        ","
                                                                                                    );
                                                                                                    tv.append(
                                                                                                        s
                                                                                                    );
                                                                                                    tv.setLayoutParams(
                                                                                                        new LinearLayout.LayoutParams(
                                                                                                            ViewGroup.LayoutParams.MATCH_PARENT,
                                                                                                            ViewGroup.LayoutParams.WRAP_CONTENT
                                                                                                        )
                                                                                                    );

                                                                                                    vg.addView(
                                                                                                        tv
                                                                                                    );
                                                                                                }
                                                                                            }
                                                                                        }
                                                                                    }
                                                                                }
                                                                            }
                                                                        );
                                                                    }
                                                                }
                                                            );

                                                    show.show();
                                                    show.getWindow().setLayout(
                                                        ViewUtil.dp2px(mActivity, 300f),
                                                        ViewUtil.dp2px(mActivity, 400f)
                                                    );

                                                    return false;
                                                }
                                            });
                                        }
                                    }
                                }
                            );
                        }
                    }
                );
            }
        }
    );
}
```
