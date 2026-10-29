
``` 
void Initialize() {

  std::vector<std::string> sonames;

  const char* android_root_env = getenv("ANDROID_ROOT");

  std::string root_dir = android_root_env != nullptr ? android_root_env : "/system";

  std::string public_native_libraries_system_config =

          root_dir + kPublicNativeLibrariesSystemConfigPathFromRoot;



  LOG_ALWAYS_FATAL_IF(!ReadConfig(public_native_libraries_system_config, &sonames),

                      "Error reading public native library list from \"%s\": %s",

                      public_native_libraries_system_config.c_str(), strerror(errno));



  // 省略无关代码



  // This file is optional, quietly ignore if the file does not exist.

  ReadConfig(kPublicNativeLibrariesVendorConfig, &sonames);



  // android_init_namespaces() expects all the public libraries

  // to be loaded so that they can be found by soname alone.

  //

  // TODO(dimitry): this is a bit misleading since we do not know

  // if the vendor public library is going to be opened from /vendor/lib

  // we might as well end up loading them from /system/lib

  // For now we rely on CTS test to catch things like this but

  // it should probably be addressed in the future.

  for (const auto& soname : sonames) {

    dlopen(soname.c_str(), RTLD_NOW | RTLD_NODELETE);

  }



  public_libraries_ = base::Join(sonames, ':');

}

```

`public_native_libraries_system_config=/system/etc/public.libraries.txt`，而`ReadConfig()`方法很简单，读取传进来的文件路径，按行分割，忽略空行和以`#`开头的行，然后把这行`push_back()`到传进来的`vector`里。

所以这个函数做了这几件事：

1. 读取`/system/etc/public.libraries.txt`和`/vendor/etc/public.libraries.txt`
2. **挨个`dlopen()`这两个txt文件里提到的所有so库**

# 2. 注入实现

有了上面的分析基础，我们这样做就可以让我们自己的so库被zygote进程`dlopen()`了：

1. 把自己写的so库扔到`/system/lib/`下面（64位是`/system/lib64/`）
2. 把这个so库的文件名追加到`/system/etc/public.libraries.txt`里

到这，已经可以在zygote里加载自己的库了，但还有一个问题：zygote只打开了这个库，并没有调用任何函数，而我们常用的`JNI_OnLoad()`函数在这是不会被调用的，怎么才能让zygote执行自己的代码呢？各位估计已经知道了，写一个用`__attribute__((constructor))`修饰的函数，这个函数会被登记在`.init.array`里，会在so被加载时调用，我们就完成了注入逻辑；接下来你就可以在zygote里做自己想要做的事了，玩的开心 :)

最后夹带点私货，此方案最先发表在[我的博客](<https://bbs.kanxue.com/elink@4d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3q4F1P5h3W2W2i4K6u0W2N6r3!0H3i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3x3o6y4Q4x3V1k6S2i4K6u0V1L8X3g2%4i4K6u0V1P5s2m8G2M7$3g2V1i4K6u0V1M7%4c8&6L8r3g2Q4x3X3c8X3M7X3q4E0k6i4N6G2M7X3E0Q4x3V1j5>)上，顺便结合SandHook写了个简单的Xposed实现，感兴趣的可以去围观下~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-6-5 09:14 被残页编辑，原因：尝试修复代码块高亮
