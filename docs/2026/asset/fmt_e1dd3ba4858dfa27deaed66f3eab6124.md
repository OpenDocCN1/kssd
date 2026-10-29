
# [原创]某东算法分析

## 1. 首先 `charles` 抓包发现每个请求 `Url` 后都接了一个 `sign` 的参数且每次都不一样。也没有其他的一些别的特别参数,那么关键问题就是分析 `sign` 参数的生成了

![](https://bbs.kanxue.com/upload/attach/202203/884888_CHV7UV9PMZWSJZX.jpg)

## 2. `jadx` 反编译,寻找 `sign` 的生成的位置

> 直接搜索 `sign` 参数匹配的出来的结果太多了,一时间不好区分哪个是真的。于是使用 `getSign` 为前缀搜索

![](https://bbs.kanxue.com/upload/attach/202203/884888_6YXS5QPFMW49SHU.jpg)![](https://bbs.kanxue.com/upload/attach/202203/884888_H8FTXZTNFBVZRKG.jpg)> 运气不错搜索到一个 `native` 方法 前缀也是 `getSign` 的方法名,打开这个类。找下引用位置进一步确认下是否为计算 `sign` 的类

![](https://bbs.kanxue.com/upload/attach/202203/884888_5PRDZ6JXCZAT8CR.jpg)

> 最终在 `BaseApplication` 里面找到这个类引用，可以确定这个就是计算 `sign` 的类

## 3. 开始分析 `sign` 方法的入参

`frida` hook 这个方法得到结果如下:

```
function main(){

     Java.perform(function(){

       var BitmapkitUtils =  Java.use("com.jingdong.common.utils.BitmapkitUtils")

       BitmapkitUtils.getSignFromJni.overload('android.content.Context', 'java.lang.String', 'java.lang.String', 'java.lang.String', 'java.lang.String', 'java.lang.String').implementation = function( context,  str,  str2,  str3,  str4,  str5){

             console.log("签名入参为: ","str 		   =>",str,"str2=>",str2,"str3=>",str3,"str4=>",str4,"str5=>",str5)

            var result = this.getSignFromJni(context,str,str2,str3,str4,str5)

             console.log('签名sign为 ',result)

            return result

      }

     })

 }
```

![](https://bbs.kanxue.com/upload/attach/202203/884888_S9DTDUBZP968UHB.jpg)

> 根据 `hook` 结果稍加分析大概知道各个参数代表的意义:

> `context`：上下文对象

> `str`：`url` 路径

> `str2`：请求 `body` 信息

> `str3`：`55a9c688729bb118` 为 `KEY` 16位长度,且每次请求都固定

> `str4`：`android` 为平台

> `str5`：版本号

> 查看源码大概得出 `str3` 的为 `installationId` 生成规则是 `UUID` 随机 `16` 位长度,在 `app` 第一次安装的时候创建保存在缓存中.

## 4. 使用 `unidbg` 进行黑盒调用

> 1. 第一步当然是补环境了:

![](https://bbs.kanxue.com/upload/attach/202203/884888_PVCNDQFNTKRY7S2.jpg)

> 需要补充返回一个 `application` 对象,偷个懒想从 `AbstractJni` 这里 `copy` 一份

![](https://bbs.kanxue.com/upload/attach/202203/884888_8X3UAPSYT6KD2Y6.jpg)

> 重新运行一下又报错了,这个错是找不到对应的 `methodId`、于是想着把 `Application` 换成 `Activity` 也可以,毕竟 `Activity` 也是可以获取 `Application` 对象的

```
  @Override

     public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {

         if ("com/jingdong/common/utils/BitmapkitUtils->a:Landroid/app/Application;".equals(signature)) {

             //返回appliation

             return vm.resolveClass("android/app/Activity",

                    vm.resolveClass("android/content/ContextWrapper",

                             vm.resolveClass("android/content/Context"))).newObject(null);

        }

         return super.getStaticObjectField(vm, dvmClass, signature);

     }
```

> 重新运行就没问题了,接下来继续报错补环境,返回 `apk` 的路径。很简单

```
 @Override

 public DvmObject<?> getObjectField(BaseVM vm, DvmObject<?> dvmObject, String signature) {

     //sourceDir 代表当前apk目录

     if("android/content/pm/ApplicationInfo->sourceDir:Ljava/lang/String;".equals(signature)){

         StringObject stringObject = new StringObject(vm, APK_PATH);

         return stringObject;

     }

     return super.getObjectField(vm, dvmObject, signature);

 }
```

![](https://bbs.kanxue.com/upload/attach/202203/884888_2G5SCVWWURYY4VB.jpg)

> 图上这个返回刚开始不知道传具体什么参数,于是打算看看源码。结果没有反编译出来具体的函数实现。于是用 `frida` hook 了这个方法拿到图下的返回

> 第一个参数为：`app` 安装目录 第二个参数为 `META-INF` 目录 第三个是 `RSA`

![](https://bbs.kanxue.com/upload/attach/202203/884888_BV3P5E9HQHEWBKY.jpg)> 既然这样就直接返回 `RSA` 公钥了

```
@Override

     public DvmObject<?> callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {

         if ("com/jingdong/common/utils/BitmapkitZip->unZip(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)[B".equals(signature)) {

             byte[] unzip = vm.unzip("META-INF/xxxx.RSA");

             System.out.println("unzip " + new String(unzip));

             return new ByteArray(vm, unzip);

         }

         return super.callStaticObjectMethod(vm, dvmClass, signature, varArg);

     }
```

> 继续补充环境

```
 @Override

 public DvmObject<?> newObject(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {

     if ("sun/security/pkcs/PKCS7-><init>([B)V".equals(signature)) {

         DvmObject<?> objectArg = varArg.getObjectArg(0);

         try {

             PKCS7 pkcs7 = new PKCS7((byte[]) objectArg.getValue());

             return vm.resolveClass("sun/security/pkcs/PKCS7").newObject(pkcs7);

         } catch (ParsingException e) {

             e.printStackTrace();

         }

 

     }

     return super.newObject(vm, dvmClass, signature, varArg);

 }
```

> 继续补充环境,需补充如下

![](https://bbs.kanxue.com/upload/attach/202203/884888_WSSB4BXMRKF46FF.jpg)

```
 @Override

     public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {

         if ("sun/security/pkcs/PKCS7->getCertificates()[Ljava/security/cert/X509Certificate;".equals(signature)) {

             PKCS7 pkcs7 = (PKCS7) dvmObject.getValue();

             X509Certificate[] certificates = pkcs7.getCertificates();

             DvmObject<?> object = ProxyDvmObject.createObject(vm, certificates);

             return object;

 

 

         }

         return super.callObjectMethod(vm, dvmObject, signature, varArg);

     }
```

> 继续补充环境,从反编译的源码中 `copy` 一份 `objectToBytes` 方法即可然后构造 `ByteArray` 返回即可

```
 if("com/jingdong/common/utils/BitmapkitZip->objectToBytes(Ljava/lang/Object;)[B".equals(signature)){

     DvmObject<?> objectArg = varArg.getObjectArg(0);

     byte[] bytes = objectToBytes(objectArg.getValue());

     return new ByteArray(vm,bytes);

 }
```

## 5. 开始黑盒调用计算 `sign` 的方法

```
 List<Object> params = new ArrayList<>();

        params.add(dalvikVM.getJNIEnv());

        params.add(0);

        DvmClass context = dalvikVM.resolveClass("android/content/Context");

        params.add(dalvikVM.addLocalObject(context.newObject(null)));

        params.add(dalvikVM.addLocalObject(new StringObject(dalvikVM, "personinfoBusiness")));

        params.add(dalvikVM.addLocalObject(new StringObject(dalvikVM, "{\"callCJH\":\"1\",\"callNPS\":\"1\",\"closeJX\":\"0\",\"headTaskRefresh\":\"1\",\"locationArea\":\"0_0_0_0\",\"menuStaticSource\":\"0\",\"menuTimeStamp\":\"1631586010000\"}")));

        params.add(dalvikVM.addLocalObject(new StringObject(dalvikVM, INSTALL_ID)));

        params.add(dalvikVM.addLocalObject(new StringObject(dalvikVM, PLAT_FROM)));

        params.add(dalvikVM.addLocalObject(new StringObject(dalvikVM, VERSION)));

        Number numbers = moduleModule.callFunction(androidEmulator, 0x028B4+1, params.toArray())[0];
```

> 运行报错，还缺少环境

![](https://bbs.kanxue.com/upload/attach/202203/884888_DVV4XHMJHDEQ58T.jpg)

```
 @Override

 public DvmObject<?> newObjectV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {

     if("java/lang/StringBuffer-><init>()V".equals(signature)){

         StringBuffer stringBuffer = new StringBuffer();

         return vm.resolveClass("java/lang/StringBuffer").newObject(stringBuffer);

     }

     return super.newObjectV(vm, dvmClass, signature, vaList);

 }
```

> 重新继续补充环境 这部分环境补充比较简单 直接上最后运行结果

![](https://bbs.kanxue.com/upload/attach/202203/884888_KXRWUX24KPKFCYC.jpg)

> 黑盒调用没问题了,接下来开始分析 `sign` 具体是怎么生成的。全部代码如下:
