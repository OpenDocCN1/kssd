
```java
public class qqx5 {

	public static void qqx5Hook(final XC_LoadPackage.LoadPackageParam loadPackageParam) throws ClassNotFoundException{

		if (!loadPackageParam.packageName.contains("com.tencent.tmgp.MGCForAndroid")) return;

        XposedBridge.log("Loaded app: " + loadPackageParam.packageName);

        

        //该函数应该是一个日志类，此处hook该类可以查看关键信息

        XposedHelpers.findAndHookMethod("com.tencent.open.a.f", loadPackageParam.classLoader, "c", String.class, String.class, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	//XposedBridge.log("日志类*****");

            	String str_1 = param.args[0].toString();

            	if((!str_1.contains("HotUpdate")) && (!str_1.contains("GiftConfig"))){

            		XposedBridge.log("日志类:" + str_1 + ":" + param.args[1].toString());

            	}

            }

        });

        

        //另一个日志类

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.utils.z", loadPackageParam.classLoader, "b", String.class, String.class, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("日志类*****");

            	String str_1 = param.args[0].toString();

            	if((!str_1.contains("HotUpdate")) && (!str_1.contains("GiftConfig"))){

            		XposedBridge.log("日志类:" + str_1 + ":" + param.args[1].toString());

            	}

            }

        });

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.utils.RSAUtil", loadPackageParam.classLoader, "encrypt", int.class, String.class, int.class, int.class, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("RSAUtilencrypt*****");

            	XposedBridge.log("RSAUtilencrypt_param_0:" + param.args[0].toString());

            	XposedBridge.log("RSAUtilencrypt_param_1:" + param.args[1].toString());

            	XposedBridge.log("RSAUtilencrypt_param_2:" + param.args[2].toString());

            	XposedBridge.log("RSAUtilencrypt_param_3:" + param.args[3].toString());

            }

            @Override

            protected void afterHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("RSAUtilencrypt_return:" + param.getResult().toString());

            }

        });

      //发送数据包加密与解密

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "b", int.class, byte[].class,int.class, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("decrypt*****");

            	XposedBridge.log("decrypt_param_0:" + param.args[0].toString());

            	XposedBridge.log("decrypt_param_1:" + utils.bytesToHexFun1((byte[])param.args[1]));

            	XposedBridge.log("decrypt_param_2:" + param.args[2].toString());

            }

            @Override

            protected void afterHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("decrypt_return:" + utils.bytesToHexFun1((byte[])param.args[1]));

            }

        });

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "a", int.class, byte[].class,int.class, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("encrypt*****");

            	XposedBridge.log("encrypt_param_0:" + param.args[0].toString());

            	XposedBridge.log("encrypt_param_1:" + utils.bytesToHexFun1((byte[])param.args[1]));

            	XposedBridge.log("encrypt_param_2:" + param.args[2].toString());

            }

            @Override

            protected void afterHookedMethod(MethodHookParam param) throws Throwable {

            	XposedBridge.log("encrypt_return:" + utils.bytesToHexFun1((byte[])param.args[1]));

            }

        });

        

        Class<?> NetBufferClass = loadPackageParam.classLoader.loadClass("com.h3d.qqx5.framework.d.f");

       

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "a", Object.class, NetBufferClass, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	String str = param.args[0].getClass().getName();

            	XposedBridge.log("SaveStream_param_0:" + str);

            }

        });

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.j", loadPackageParam.classLoader, "b", Object.class, NetBufferClass, new XC_MethodHook(){

            @Override

            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

            	String str = param.args[0].getClass().getName();

            	XposedBridge.log("LoadStream_param_0:" + str);

            }

        });

        

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.h", loadPackageParam.classLoader, "c", new XC_MethodHook(){

            @Override

            protected void afterHookedMethod(MethodHookParam param) throws Throwable {

            	Method rpcMethod = param.getResult().getClass().getMethod("k");

            	ByteBuffer response = (ByteBuffer) rpcMethod.invoke(param.getResult());

            	byte[] bytes = response.array();

            	XposedBridge.log("m_body_buffer_return:" + utils.bytesToHexFun1(bytes));

            }

        });

        

        XposedHelpers.findAndHookMethod("com.h3d.qqx5.framework.d.h", loadPackageParam.classLoader, "d", new XC_MethodHook(){

            @Override

            protected void afterHookedMethod(MethodHookParam param) throws Throwable {

            	Method rpcMethod = param.getResult().getClass().getMethod("k");

            	ByteBuffer response = (ByteBuffer) rpcMethod.invoke(param.getResult());

            	byte[] bytes = response.array();

            	XposedBridge.log("m_header_buffer_return:" + utils.bytesToHexFun1(bytes));

            }

        });

	}



}
```

全部完毕。

这次app逆向中，学习到了很多知识，该app采用了java完成了序列化和反序列化，让我知道反射还可以这样玩，深入理解了基础连接认证，实际上安全隐患很大，因为其数据包并没有采用安全的密钥体系，中间人随意窃听。深入理解了其加密解密流程，整数的压缩与解压流程，感觉自己的功力又深了一层等等等等。。收货蛮大。

彩蛋环节：

在分析app的过程中，发现在帐号的认证过程中，也就是`CEventQueryVideoAccountInfo`数据包，`account`，`skey`，`openid`三者应该对应才可以成功登陆帐号，进而对帐号进行后续操作。实际发现，服务器对于`skey`与`openid`并没有校验，也就是说任意帐号即可登录炫舞梦工厂app。

已经和腾讯安全应急中心报告了该漏洞，并且已经完成了修复。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-3-18 07:58

被小堆编辑，原因：

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)

[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)
