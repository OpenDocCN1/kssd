
然而脱下来的壳依旧只有Shell代码，即便我们通过延迟脱壳(即首先获取DexFile的begin和size，之后通过命令行调用脱壳函数)依旧如此

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_9C9UU48UR24HNYR.webp)

所以尝试别的办法，首先我们知道整体加固的原理是替换ClassLoader来加载原Dex，而Shell Dex尾部恰好存在着大量数据，可以猜测这就是被加密的Dex。由于Shell加载完成之后会将程序控制权归还给原程序，所以我们可以Hook `performLaunchActivity`，在Activity启动之前获取系统ClassLoader进而得到其加载的Dex文件进行Dump。经过测试，最后一个覆盖的Dex文件为原Dex

```javascript
function dump_memory(base, size) {
  Java.perform(function () {
    var currentApplication = Java.use("android.app.ActivityThread").currentApplication();
    var dir = "/data/local/tmp";
    var file_path = dir + "/mydump.so";
    var file_handle = new File(file_path, "w+");
    if (file_handle && file_handle != null) {
      Memory.protect(ptr(base), size, 'rwx');
      var libso_buffer = ptr(base).readByteArray(size);
      file_handle.write(libso_buffer);
      file_handle.flush();
      file_handle.close();
      console.log("[dump]:", file_path);
    }
  });
}
var size = 0
var dump_addr = 0

function hook_libart() {
  Java.perform(function () {
    var loadMethod = Module.findExportByName("libart.so", "_ZN3art11ClassLinker10LoadMethodERKNS_7DexFileERKNS_13ClassAccessor6MethodENS_6HandleINS_6mirror5ClassEEEPNS_9ArtMethodE")
    var dumped = 0
    Interceptor.attach(loadMethod, {
      onEnter: function (args) {
        if (dumped == 0) {
          dump_addr = ptr(args[1].add(8).readPointer())
          size = args[1].add(16).readInt()
          dump_memory(dump_addr, size)
          dumped = 1
        }
      },
      onLeave: function (retval) {}
    })
  })
}

function main() {
  hook_libart()
}
setImmediate(main)
```

```javascript
function dump_memory(base, size) {
  Java.perform(function () {
    var currentApplication = Java.use("android.app.ActivityThread").currentApplication();
    var dir = "/data/local/tmp";
    var file_path = dir + "/mydump.so";
    var file_handle = new File(file_path, "w+");
    if (file_handle && file_handle != null) {
      Memory.protect(ptr(base), size, 'rwx');
      var libso_buffer = ptr(base).readByteArray(size);
      file_handle.write(libso_buffer);
      file_handle.flush();
      file_handle.close();
      console.log("[dump]:", file_path);
    }
  });
}

function getDexFile() {
  Hook_Invoke()
  Java.perform(function () {
    var ActivityThread = Java.use("android.app.ActivityThread")
    ActivityThread["performLaunchActivity"].implementation = function (args) {
      var env = Java.vm.getEnv()
      var classloader = this.mInitialApplication.value.getClassLoader()
      var BaseDexClassLoader = Java.use("dalvik.system.BaseDexClassLoader")
      var elementsClass = Java.use("dalvik.system.DexPathList$Element")
      classloader = Java.cast(classloader, BaseDexClassLoader)
      var pathList = classloader.pathList.value
      var elements = pathList.dexElements.value
      console.log(elements)
      //console.log(elements.value)
      for (var i in elements) {
        var element;
        try {
          element = Java.cast(elements[i], elementsClass);
        } catch (e) {
          console.log(e)
        }
        //之后需要获取DexFile
        var dexFile = element.dexFile.value
        //getMethod(dexFile, classloader)
        var mCookie = dexFile.mCookie.value
        //$h获取句柄
        var length = env.getArrayLength(mCookie.$h, 0)
        //console.log(length)
        var Array = env.getLongArrayElements(mCookie.$h, 0)
        //第一个不是DexFile
        for (var i = 1; i < length; ++i) {
          var DexFilePtr = Memory.readPointer(ptr(Array).add(i * 8))
          var DexFileBegin = ptr(DexFilePtr).add(Process.pointerSize).readPointer()
          var DexFileSize = ptr(DexFilePtr).add(Process.pointerSize * 2).readU32()
          console.log(hexdump(DexFileBegin))
          console.log("Size => " + DexFileSize.toString(16))
          dump_memory(DexFileBegin,DexFileSize)
          //dex_begin = DexFileBegin
          //dex_size = DexFileSize
        }
      }
      return this.performLaunchActivity(arguments[0], arguments[1])
    }
  })
}
```

反编译结果如下，可以发现onCreate的方法内容未被填充

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_3RMDT6AGNUAJNS8.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202309/942805_VJUV48GEYTY87MA.webp)

所以尝试延迟Dump的时机，通过Hook `ArtMethod::Invoke`来Dump，最终代码如下

```javascript
function Hook_Invoke() {
  var InvokeFunc = Module.findExportByName("libart.so", "_ZN3art9ArtMethod6InvokeEPNS_6ThreadEPjjPNS_6JValueEPKc")
  Interceptor.attach(InvokeFunc, {
    onEnter: function (args) {
      console.log(args[1])
      // args[0].add(8).readInt().toString(16) == 0
      console.log("Method Idx -> " + args[0].add(8).readInt().toString(16) + " " + args[0].add(12).readInt().toString(16))
      dump_memory(dex_begin, dex_size)
      //dump_memory(dex_begin, dex_size)
    },
    onLeave: function (retval) {}
  })
}
var dex_begin = 0
var dex_size = 0

function dump_memory(base, size) {
  Java.perform(function () {
    var currentApplication = Java.use("android.app.ActivityThread").currentApplication();
    var dir = "/data/local/tmp";
    var file_path = dir + "/mydump.so";
    var file_handle = new File(file_path, "w+");
    if (file_handle && file_handle != null) {
      Memory.protect(ptr(base), size, 'rwx');
      var libso_buffer = ptr(base).readByteArray(size);
      file_handle.write(libso_buffer);
      file_handle.flush();
      file_handle.close();
      console.log("[dump]:", file_path);
    }
  });
}

function getDexFile() {

  Hook_Invoke()
  Java.perform(function () {
    var ActivityThread = Java.use("android.app.ActivityThread")
    ActivityThread["performLaunchActivity"].implementation = function (args) {
      var env = Java.vm.getEnv()
      var classloader = this.mInitialApplication.value.getClassLoader()
      var BaseDexClassLoader = Java.use("dalvik.system.BaseDexClassLoader")
      var elementsClass = Java.use("dalvik.system.DexPathList$Element")
      classloader = Java.cast(classloader, BaseDexClassLoader)
      var pathList = classloader.pathList.value
      var elements = pathList.dexElements.value
      console.log(elements)
      //console.log(elements.value)
      for (var i in elements) {
        var element;
        try {
          element = Java.cast(elements[i], elementsClass);
        } catch (e) {
          console.log(e)
        }
        //之后需要获取DexFile
        var dexFile = element.dexFile.value
        //getMethod(dexFile, classloader)
        var mCookie = dexFile.mCookie.value
        //$h获取句柄
        var length = env.getArrayLength(mCookie.$h, 0)
        //console.log(length)
        var Array = env.getLongArrayElements(mCookie.$h, 0)
        //第一个不是DexFile
        for (var i = 1; i < length; ++i) {
          var DexFilePtr = Memory.readPointer(ptr(Array).add(i * 8))
          var DexFileBegin = ptr(DexFilePtr).add(Process.pointerSize).readPointer()
          var DexFileSize = ptr(DexFilePtr).add(Process.pointerSize * 2).readU32()
          console.log(hexdump(DexFileBegin))
          console.log("Size => " + DexFileSize.toString(16))
          dex_begin = DexFileBegin
          dex_size = DexFileSize
        }
      }
      return this.performLaunchActivity(arguments[0], arguments[1])
    }
  })
}
```

反编译效果如下
