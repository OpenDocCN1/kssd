
`static void LinkCode(ClassLinker * class_linker, ArtMethod * method, const OatFile::OatClass * oat_class, uint32_t class_def_method_index) REQUIRES_SHARED(Locks::mutator_lock_) {`
  `...`
  `const void * quick_code = method->GetEntryPointFromQuickCompiledCode();`
  `bool enter_interpreter = class_linker->ShouldUseInterpreterEntrypoint(method, quick_code);`
  `if (strlen(ArtMethod::GetTraceMethod())>0){`
  `std::ostringstream oss;`
  `oss<<"mikrom LinkCode method:"<<method->PrettyMethod().c_str();`
  `LOG(ERROR)<<oss.str();`
  `enter_interpreter=true;`
  `}`
  `...`
`}`

变量`enter_interpreter`决定了是否使用解释器执行，如果是quick快速模式执行，将无法执行到我们修改的函数。最后测试发现，函数第一次调用的时候，是走的解释执行，成功打印出trace记录，第二次调用时，又被优化到快速模式执行了。不过我们已经拿到想要的结果了，我就不继续优化了。关于`LinkCode`这里的修改如果不理解的，可以看看下面的参考文章，里面有详细的流程图了。我就不重复了。

> 参考文章：[将FART和Youpk结合来做一次针对函数抽取壳的全面提升](https://bbs.pediy.com/thread-260052.htm)

### 7、dex注入（未完成）

这个功能我原本是想内置一个IO重定向，或者是内置一个Sandhook，然后直接通过配置勾选就能自动注入使用了。然后我们修改的dex直接注入进去即可使用。不过目前还没想好怎么设计里面的一些交互部分。所以只是简单的做了下注入的步骤。后续也不一定有时间继续实现，先暂时搁置把。

### MikManager展示

![](https://bbs.kanxue.com/upload/attach/202201/659397_2YQMA39YAFF66RH.gif)

### 使用说明

操作起来非常简单，选择想要处理的应用，然后选择对应功能即可。记得勾选完后，回到应用栏目点保存，如果是要使用frida脚本，或者是要选择so，需要把文件放在sdcard中的对应应用的目录中。如果sdcard中没有对应目录，可以打开一下应用会自动生成的。

脱壳功能的保存结果在`/sdcard/Android/data/<PackageName>/dump`中查看

查看logcat输出日志统一搜索mikrom，jni调用的输出的格式如下

```
2022-01-30 19:43:30.198 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni CallNonvirtualVoidMethodV    void java.lang.ClassNotFoundException.<init>(java.lang.String, java.lang.Throwable)    arg1:android.widget.ViewStubarg2:0x6fcc6f50
2022-01-30 19:43:30.198 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni GetArrayLength
2022-01-30 19:43:30.198 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni GetStringUTFChars    android.widget.ViewStub
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni CallNonvirtualVoidMethodV    void java.lang.ClassNotFoundException.<init>(java.lang.String, java.lang.Throwable)    arg1:android.widget.ViewStubarg2:0x12c2a5e8
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni GetStringUTFChars    android.webkit.ViewStub
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni NewStringUTF    android.webkit.ViewStub
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni CallObjectMethodV    java.lang.Class java.lang.ClassLoader.loadClass(java.lang.String)    arg1:android.webkit.ViewStub
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni GetStringUTFChars    android.webkit.ViewStub
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni GetStringUTFChars    android.webkit.ViewStub
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni CallNonvirtualVoidMethodV    void java.lang.ClassNotFoundException.<init>(java.lang.String, java.lang.Throwable)    arg1:android.webkit.ViewStubarg2:0x6fcc6f50
2022-01-30 19:43:30.199 4105-4105/com.kanxue.crackme E/.kanxue.crackm: mikrom jni GetArrayLength
```

RegisterNative的输出格式如下

```
2022-01-30 19:48:43.219 4576-4576/com.kanxue.crackme E/.kanxue.crackm: mikrom RomPrint RegisterNative name:boolean com.kanxue.crackme.MainActivity.jnicheck(java.lang.String) native_ptr:0x75a5735904 method_idx:568
2022-01-30 19:48:43.219 4576-4576/com.kanxue.crackme E/.kanxue.crackm: mikrom RomPrint RegisterNative name:boolean com.kanxue.crackme.MainActivity.crypt2(java.lang.String) native_ptr:0x75a5739750 method_idx:2
```

ArtInvoke的输出格式如下

```
2022-01-30 19:50:16.247 4686-4686/com.kanxue.crackme E/.kanxue.crackm: mikrom [PerformCall] caller:boolean sun.misc.Unsafe.compareAndSwapInt(java.lang.Object, long, int, int)---called:boolean sun.misc.Unsafe.compareAndSwapInt(java.lang.Object, long, int, int)
2022-01-30 19:50:16.247 4686-4686/com.kanxue.crackme E/.kanxue.crackm: mikrom [PerformCall] caller:void android.graphics.Paint.nSetFlags(long, int)---called:void android.graphics.Paint.nSetFlags(long, int)
2022-01-30 19:50:16.247 4686-4686/com.kanxue.crackme E/.kanxue.crackm: mikrom [PerformCall] caller:int java.lang.String.hashCode()---called:int java.lang.String.hashCode()
2022-01-30 19:50:16.247 4686-4686/com.kanxue.crackme E/.kanxue.crackm: mikrom [PerformCall] caller:void android.graphics.Paint.nSetTextLocalesByMinikinLocaleListId(long, int)---called:void android.graphics.Paint.nSetTextLocalesByMinikinLocaleListId(long, int)
2022-01-30 19:50:16.247 4686-4686/com.kanxue.crackme E/.kanxue.crackm: mikrom [PerformCall] caller:void android.graphics.Paint.nSetDither(long, boolean)---called:void android.graphics.Paint.nSetDither(long, boolean)
2022-01-30 19:50:16.247 4686-4686/com.kanxue.crackme E/.kanxue.crackm: mikrom [PerformCall] caller:void android.graphics.Paint.nSetDither(long, boolean)---called:void android.graphics.Paint.nSetDither(long, boolean)
```

smali trace日志的输出结果如下
