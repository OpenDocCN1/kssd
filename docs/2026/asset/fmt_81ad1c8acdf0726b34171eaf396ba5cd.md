
```
    /**

     * 直接hook sql达到获取撤回消息id的目的

     *

     * @param applicationContext

     * @param classLoader

     */

    private void hookDB(final Context applicationContext, final ClassLoader classLoader) {

        final Class<?> sQLiteDatabase = XposedHelpers.findClass("com.tencent.wcdb.database.SQLiteDatabase", classLoader);

        final Class<?> cancellationSignal = XposedHelpers.findClass("com.tencent.wcdb.support.CancellationSignal", classLoader);

        if (sQLiteDatabase == null) return;

        XposedHelpers.findAndHookConstructor("com.tencent.wcdb.database.SQLiteProgram",

                classLoader,

                sQLiteDatabase,

                String.class,

                Object[].class,

                cancellationSignal,

                new XC_MethodReplacement() {

                    @Override

                    protected Object replaceHookedMethod(MethodHookParam param) throws Throwable {

                        Object[] objArr = (Object[]) param.args[2];

                        String originalSql = param.args[1].toString();

                        //打印所有调用SQLiteProgram的sql

                        //LogUtils.e("hookDB", "sql -> " + param.args[1], "objArr:" + JSON.toJSONString(objArr));

                        if (objArr != null && originalSql.toUpperCase().startsWith("UPDATE MESSAGE")) {

                            for (Object obj : objArr) {

                                String sqlParam = obj.toString();//自己撤回10002 别人撤回10000

                                if (sqlParam.equals("10000")) {//别人撤回

                                    Object[] newObjArr = new Object[2];

                                    //param.args[1] = "UPDATE message SET type=? WHERE msgId=?";

                                    param.args[1] = "select * from message where type=? and msgId=?";

                                    param.args[2] = newObjArr;

                                    newObjArr[0] = 1;

                                    newObjArr[1] = objArr[objArr.length - 1];

                                    //param.args[1] = "UPDATE message SET content=(select (select content from message where msgId = ?)||X'0D'||X'0A'||X'0D'||X'0A'||(\"<sysmsg>wxInvoke卧槽，TA竟然要撤回上面的信息wxInvoke</sysmsg>\")),msgId=?,type=? WHERE msgId=?";

                                    LogUtils.e("hookDB", "originalSql->" + originalSql, "newSql->" + param.args[1], "sqlParam->" + JSON.toJSONString(newObjArr));

                                    WxChatInvokeMsg msg = new WxChatInvokeMsg();

                                    msg.setMsgId(newObjArr[1].toString());

                                    WxChatInvokeMsgDB.insertData(applicationContext, msg);

                                }

                            }

                        }

                        return XposedBridge.invokeOriginalMethod(param.method, param.thisObject, param.args);

                    }

                });



    }

```

看到上面我hook的是`SQLiteProgram` 会有朋友问为啥？

因为我`SQLiteDatabase`在里面找`insert`和`update`的方法发现里面最后走进入了`SQLiteProgram` 哈哈！ 偷个懒我就不具体截图找给大家看了，然后把撤回的消息id记录在本地，作为后面标记的依据

# 接下来开始处理撤回效果了
