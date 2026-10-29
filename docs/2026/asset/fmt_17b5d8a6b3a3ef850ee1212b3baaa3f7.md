
进入`/path/to/xxx`目录，找到`xxx.json`。

好的，又开始报错了，咱继续补环境

- `java/lang/StringBuffer-><init>()V`

```
java.lang.UnsupportedOperationException: java/lang/StringBuffer-><init>()V
at com.github.unidbg.linux.android.dvm.AbstractJni.newObjectV(AbstractJni.java:791)
```

```
@Override
public DvmObject<?> newObjectV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
switch (signature) {
case "java/lang/StringBuffer-><init>()V": {
StringBuffer stringBuffer = new StringBuffer();
return vm.resolveClass("java/lang/StringBuffer").newObject(stringBuffer);
}
}
return super.newObjectV(vm, dvmClass, signature, vaList);
}
```

- `java/lang/StringBuffer->append(Ljava/lang/String;)Ljava/lang/StringBuffer;`

```
java.lang.UnsupportedOperationException: java/lang/StringBuffer->append(Ljava/lang/String;)Ljava/lang/StringBuffer;
at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethodV(AbstractJni.java:416)
```

```
@Override
public DvmObject<?> callObjectMethodV(BaseVM vm, DvmObject<?> dvmObject, String signature, VaList vaList) {
switch (signature) {
case "java/lang/StringBuffer->append(Ljava/lang/String;)Ljava/lang/StringBuffer;": {
StringBuffer stringBuffer = (java.lang.StringBuffer) dvmObject.getValue();
stringBuffer.append(vaList.getObjectArg(0).getValue());
DvmClass StringBuffer = vm.resolveClass("java/lang/StringBuffer");
return StringBuffer.newObject(stringBuffer);
}
}
return super.callObjectMethodV(vm, dvmObject, signature, vaList);
}
```

- `java/lang/Integer->toString()Ljava/lang/String;`

```
java.lang.UnsupportedOperationException: java/lang/Integer->toString()Ljava/lang/String;
at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethodV(AbstractJni.java:416)
```

```
case "java/lang/Integer->toString()Ljava/lang/String;": {
Integer value = (Integer) dvmObject.getValue();
return vm.resolveClass("java/lang/String").newObject(value.toString());
}
```

- `java/lang/StringBuffer->toString()Ljava/lang/String;`

```
java.lang.UnsupportedOperationException: java/lang/StringBuffer->toString()Ljava/lang/String;
at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethodV(AbstractJni.java:416)
```

```
case "java/lang/StringBuffer->toString()Ljava/lang/String;": {
StringBuffer value = (StringBuffer) dvmObject.getValue();
return vm.resolveClass("java/lang/String").newObject(value.toString());
}
```

最后发现环境补完了，可以直接得到 `sign` 了！

## jdgs
