
在 `unidbg` 里面，`resolveClass`是一个可变参数方法，它的第二个参数是可变参数，用于进一步描述我们所构造的这个`DvmClass`的父类和接口。

拿目前需要补充的 `android.app.Application` 为例，通过搜索资料可知，`Application` 继承与 `ContextWrapper`， 而 `ContextWrapper` 又继承与 `Context`，所以在 `unidbg` 里面需要实现一个 `Application`，那么可以使用以下嵌套代码：

```
DvmClass Context = vm.resolveClass("android/content/Context");
DvmClass ContextWrapper = vm.resolveClass("android/content/ContextWrapper", Context);
DvmClass Application = vm.resolveClass("android/app/Application", ContextWrapper);
```

补环境代码如下：

```
@Override
public DvmObject<?> getStaticObjectField(BaseVM vm, DvmClass dvmClass, String signature) {
    switch (signature) {
        case "com/jingdong/common/utils/BitmapkitUtils->a:Landroid/app/Application;": {
            DvmClass Context = vm.resolveClass("android/content/Context");
            DvmClass ContextWrapper = vm.resolveClass("android/content/ContextWrapper", Context);
            // 使用 android/app/Application 发现会报错: com.github.unidbg.arm.backend.BackendException
            // 百度了一下发现这个错误是找不到 methodID
            // 因此可以使用 android/app/Activity 代替, 因为它也可以获取 Application 对象
            DvmClass Application = vm.resolveClass("android/app/Activity", ContextWrapper);

            return Application.newObject(signature);
        }
    }

    return super.getStaticObjectField(vm, dvmClass, signature);
}
```

- `android/content/pm/ApplicationInfo->sourceDir:Ljava/lang/String;`

```
java.lang.UnsupportedOperationException: android/content/pm/ApplicationInfo->sourceDir:Ljava/lang/String;
    at com.github.unidbg.linux.android.dvm.AbstractJni.getObjectField(AbstractJni.java:171)
```

取 `ApplicationInfo` 也就是 `apk` 的存放位置 可以使用 `adb` 进手机查看一下

![图片描述](https://bbs.kanxue.com/upload/attach/202303/899374_X2G724KA3R2EC96.png)

所以 `app` 在这里： `/data/app/com.jingdong.app.mall-Icn20e0xHLzrVHPBwOirRA==/bask.apk`

```
@Override
public DvmObject<?> getObjectField(BaseVM vm, DvmObject<?> dvmObject, String signature) {
    switch (signature) {
        case "android/content/pm/ApplicationInfo->sourceDir:Ljava/lang/String;": {
            return new StringObject(vm, "/data/app/com.jingdong.app.mall-Icn20e0xHLzrVHPBwOirRA==/bask.apk");
        }
    }
    return super.getObjectField(vm, dvmObject, signature);
}
```

- `com/jingdong/common/utils/BitmapkitZip->unZip(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)[B`

```
java.lang.UnsupportedOperationException: com/jingdong/common/utils/BitmapkitZip->unZip(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)[B
    at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticObjectMethod(AbstractJni.java:432)
```

这个补的时候有一个参数，可以先输出一下参数，看看都是什么

```
arg0-- /data/app/com.jingdong.app.mall-Icn20e0xHLzrVHPBwOirRA==/bask.apk
arg1-- META-INF/
arg2-- .RSA
```

所以他找的好像是 `apk` 里面 `META-INF` 文件夹里面的 `xxx.RSA` ， 我们只要使用压缩文件打开 `apk` 就可以看到一个叫做 `JINGDONG.RSA` 的文件，补全一下即可！

![图片描述](https://bbs.kanxue.com/upload/attach/202303/899374_Z795XH96JASBJ9R.png)

```
@Override
public DvmObject<?> callStaticObjectMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {

    switch (signature) {
        case "com/jingdong/common/utils/BitmapkitZip->unZip(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)[B": {
            StringObject apkPath = varArg.getObjectArg(0);
            StringObject directory = varArg.getObjectArg(1);
            StringObject filename = varArg.getObjectArg(2);

            if (
                "/data/app/com.jingdong.app.mall-Icn20e0xHLzrVHPBwOirRA==/bask.apk".equals(apkPath.getValue())
                && "META-INF/".equals(directory.getValue())
                && ".RSA".equals(filename.getValue())
            ) {
                byte[] data = vm.unzip("META-INF/JINGDONG.RSA");
                return new ByteArray(vm, data);
            }

        }
    }

    return super.callStaticObjectMethod(vm, dvmClass, signature, varArg);
}
```

- `sun/security/pkcs/PKCS7-><init>([B)V`

```
java.lang.UnsupportedOperationException: sun/security/pkcs/PKCS7-><init>([B)V
    at com.github.unidbg.linux.android.dvm.AbstractJni.newObject(AbstractJni.java:741)
```

![图片描述](https://bbs.kanxue.com/upload/attach/202303/899374_SZ28SZQH74Z8JUK.png)

写完之后提示要捕捉异常，因此就捕捉一下，虽然我也不懂为什么

```
@Override
public DvmObject<?> newObject(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
    switch (signature) {
        case "sun/security/pkcs/PKCS7-><init>([B)V": {
            PKCS7 pkcs7 = null;

            try {
                byte[] bArray = (byte[]) varArg.getObjectArg(0).getValue();
                pkcs7 = new PKCS7(bArray);

            } catch (ParsingException e) {
                e.printStackTrace();

            }
            DvmClass pkcs7Class = vm.resolveClass("sun/security/pkcs/PKCS7");
            return pkcs7Class.newObject(pkcs7);

        }
    }

    return super.newObject(vm, dvmClass, signature, varArg);

}
```

- `sun/security/pkcs/PKCS7->getCertificates()[Ljava/security/cert/X509Certificate;`

```
java.lang.UnsupportedOperationException: sun/security/pkcs/PKCS7->getCertificates()[Ljava/security/cert/X509Certificate;
    at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:921)
```

```
@Override
public DvmObject<?> callObjectMethod(BaseVM vm, DvmObject<?> dvmObject, String signature, VarArg varArg) {
    switch (signature) {
        case "sun/security/pkcs/PKCS7->getCertificates()[Ljava/security/cert/X509Certificate;": {
            PKCS7 pkcs7 = (PKCS7) dvmObject.getValue();
            X509Certificate[] x509Certificates = pkcs7.getCertificates();
            return ProxyDvmObject.createObject(vm, x509Certificates);

        }
    }
    return super.callObjectMethod(vm, dvmObject, signature, varArg);
}
```

- `com/jingdong/common/utils/BitmapkitZip->objectToBytes(Ljava/lang/Object;)[B`

```
java.lang.UnsupportedOperationException: com/jingdong/common/utils/BitmapkitZip->objectToBytes(Ljava/lang/Object;)[B
    at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticObjectMethod(AbstractJni.java:432)
```

发现他找的是一个 `objectToBytes` 方法，但是找不到，所以我们直接去 `jadx` 里面搜索一下，便得到了结果

![图片描述](https://bbs.kanxue.com/upload/attach/202303/899374_XTRUMX2XUCF25RY.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202303/899374_W7E6G7X4TDUSG94.png)

可以直接复制出来使用，去除报错，导入一些包就可以了！

```
public static byte[] objectToBytes(Object obj) {
    try {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(obj);
        objectOutputStream.flush();
        byte[] byteArray = byteArrayOutputStream.toByteArray();
        objectOutputStream.close();
        byteArrayOutputStream.close();
        return byteArray;
    } catch (IOException e2) {
        return null;
    }
}
```

然后再继续补环境

```
case "com/jingdong/common/utils/BitmapkitZip->objectToBytes(Ljava/lang/Object;)[B": {
    return new ByteArray(vm, objectToBytes(varArg.getObjectArg(0).getValue()));
}
```

接着运行发现居然不报错了，因此我们就写一个获取 `sign` 的函数尝试调用一下！
