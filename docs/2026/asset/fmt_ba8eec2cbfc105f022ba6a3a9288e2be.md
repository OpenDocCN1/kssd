
`private static void hookPackageParser(Signature[] fakeSignature) {`
`try` `{`
`RposedBridge.hookAllMethods(`  
`RposedHelpers.findClass(``"android.content.pm.PackageParser"``, ClassLoader.getSystemClassLoader()),`  
`"generatePackageInfo"``, new RC_MethodHook() {`  
`@Override`  
`protected void afterHookedMethod(MethodHookParam param) {`  
`PackageInfo packageInfo`  
`=` `(PackageInfo) param.getResult();`  
`if` `(packageInfo` `=``=` `null)` `return``;`  
`if` `(packageInfo.packageName.equals(RuntimeToolKit.packageName)) {`  
`if` `(packageInfo.signatures !``=` `null && packageInfo.signatures.length >` `0``) {`  
`CLog.i(``"PackageParser signature info (method 1)"``);`  
`packageInfo.signatures[``0``]` `=` `fakeSignature[``0``];`  
`}`  
`if` `(Build.VERSION.SDK_INT >``=` `Build.VERSION_CODES.P) {`  
`if` `(packageInfo.signingInfo !``=` `null) {`  
`CLog.i(``"PackageParser signature info (method 2)"``);`  
`Signature[] signaturesArray` `=` `packageInfo.signingInfo.getApkContentsSigners();`  
`if` `(signaturesArray !``=` `null && signaturesArray.length >` `0``) {`  
`signaturesArray[``0``]` `=` `fakeSignature[``0``];`  
`}`  
`}`  
`}`  
`}`  
`}`  
`});`  
`} catch (Throwable e) {`  
`CLog.e(``"hook apkSign PackageParser -> generatePackageInfo "` `+` `e.getMessage());`  
`}`  
`}`

### 替换CREATOR

这个思路也是抄的lspatch ，这种思路可以通过检测CREATOR 变量Classloader的方式检测出来 。

```
public static void byPassSignatureForCREATOR(Signature[] fakeSignature) {
if (fakeSignature == null) {
return;
}
Parcelable.Creator<PackageInfo> originalCreator = PackageInfo.CREATOR;
Parcelable.Creator<PackageInfo> proxiedCreator = new Parcelable.Creator<PackageInfo>() {
@Override
public PackageInfo createFromParcel(Parcel source) {
PackageInfo packageInfo = originalCreator.createFromParcel(source);
if (packageInfo.packageName.equals(RuntimeToolKit.packageName)) {
if (packageInfo.signatures != null && packageInfo.signatures.length > 0) {
//CLog.i("CREATOR Replace signature info (method 1)");
packageInfo.signatures[0] = fakeSignature[0];
}
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
if (packageInfo.signingInfo != null) {
//CLog.i("CREATOR Replace signature info (method 2)");
Signature[] signaturesArray = packageInfo.signingInfo.getApkContentsSigners();
if (signaturesArray != null && signaturesArray.length > 0) {
signaturesArray[0] = fakeSignature[0];
}
}
}
}
return packageInfo;
}
@Override
public PackageInfo[] newArray(int size) {
return originalCreator.newArray(size);
}
};
RposedHelpers.setStaticObjectField(PackageInfo.class, "CREATOR", proxiedCreator);
try {
Map<?, ?> mCreators = (Map<?, ?>) RposedHelpers.getStaticObjectField(Parcel.class, "mCreators");
mCreators.clear();
} catch (NoSuchFieldError ignore) {
} catch (Throwable e) {
CLog.e("fail to clear Parcel.mCreators", e);
}
try {
Map<?, ?> sPairedCreators = (Map<?, ?>)
RposedHelpers.getStaticObjectField(Parcel.class,
"sPairedCreators");
sPairedCreators.clear();
} catch (NoSuchFieldError ignore) {
} catch (Throwable e) {
CLog.e("fail to clear Parcel.sPairedCreators", e);
}
}
```

### Binder transact 方法

有很多大厂会自己伪造IPC和服务端进行通讯，用这种方法可以进行签名的修改和替换，实现思路主要就是Hook binder通讯解析的方法 。判断如果传输类型数据是获取签名的话就进行替换 。主要思路就是Hook传输数据的“水管”，对水管里面的内容进行替换 。

不知道为什么lspatch新版本把这段代码删除了 ，只采用了替换CREATOR 和Hook服务端解析签名的方法 。

代码如下：

```
RposedHelpers.findAndHookMethod("android.os.BinderProxy",
context.getClassLoader(),
"transact",
int.class,
Parcel.class,
Parcel.class,
int.class,
new RC_MethodHook() {
@Override
protected void afterHookedMethod(MethodHookParam param) {
try {
Object object = param.thisObject;
int id = (int) param.args[0];
Parcel write = (Parcel) param.args[1];
Parcel out = (Parcel) param.args[2];
/* forward check */
if (write == null || out == null) {
return;
}
/* prevent recurise call (防止递归) */
if (id == IBinder.INTERFACE_TRANSACTION) {
/* IBinder协议事务代码：询问事务的收件人端以获取其规范接口描述符。 */
return;
}
String desc = (String) RposedHelpers.callMethod(object, "getInterfaceDescriptor");
if (desc == null || !desc.equals("android.content.pm.IPackageManager")) {
return;
}
if (id == TRANSACTION_getPackageInfo_ID) {
out.readException();
if (0 != out.readInt()) {
PackageInfo packageInfo = PackageInfo.CREATOR.createFromParcel(out);
if (packageInfo.packageName.equals(context.getApplicationInfo().packageName)) {
if (fakeSignature[0] == null) {
CLog.e(">>>>>>>>>> byPassSignature fakeSignature == null");
System.exit(0);
return;
}
/*CLog.i(
"org data size -> " + out.dataSize());*/

if (packageInfo.signatures != null && packageInfo.signatures.length > 0) {
packageInfo.signatures[0] = fakeSignature[0];
/*CLog.i(" byPassSignatureByLSPosed 1 !!");*/
}
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
if (packageInfo.signingInfo != null) {
Signature[] signaturesArray = packageInfo.signingInfo.getApkContentsSigners();
if (signaturesArray != null && signaturesArray.length > 0) {
signaturesArray[0] = fakeSignature[0];
/*CLog.i(
" byPassSignatureByLSPosed 2 !!");*/
}
}
}

/* reset pos */
out.setDataPosition(0);
out.setDataSize(0);
out.writeNoException();
out.writeInt(1);
packageInfo.writeToParcel(out, PARCELABLE_WRITE_RETURN_VALUE);
}
}
/* reset pos */
out.setDataPosition(0);
}
} catch (Throwable err) {
CLog.e(">>>>>>>>>> byPassSignatureByLSPosed error " + err.getMessage());
}
}
});
```

还有很多很好思路的代码 ，比如ISO线程去读取apk签名信息，防止SVC被IO重定向掉 。

当然检测签名的方法肯定不止这些，如果有更好的，我没写的，也可以在文章下面留言 。

### 模拟器检测：

Java层基础的获取api架构啥的，这块就不一一叙述了 。

#### Seccomp检测架构：

主要思路来自B哥 ，感谢B个 ，先安装seccomp架子 。
