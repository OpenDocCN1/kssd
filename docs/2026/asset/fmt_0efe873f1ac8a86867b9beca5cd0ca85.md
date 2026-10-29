
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式 `cvar = avar + bvar` 中，加法运算符（`+`）将 `avar` 与 `bvar` 相加，得到它们的和 `cvar`

在 `List.of(arg0, arg1, arg2)` 中，`List` 接口的工厂方法 `of()` 接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用 `cmd` 命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[/content]

Wait need process given text.

Given text is long. Need to reformat.

I'll produce a cleaned version with headings.

[content]
## 第一关 Frida Hook

```
function hook_java(){
        Java.perform(function(){
  Java.use("com.example.androiddemo.Activity.FridaActivity4$InnerClasses").check1.implementation = function(){return true};
  Java.use("com.example.androiddemo.Activity.FridaActivity4$InnerClasses").check2.implementation = function(){return true};
  Java.use("com.example.androiddemo.Activity.FridaActivity4$InnerClasses").check3.implementation = function(){return true};
  Java.use("com.example.androiddemo.Activity.FridaActivity4$InnerClasses").check4.implementation = function(){return true};
  Java.use("com.example.androiddemo.Activity.FridaActivity4$InnerClasses").check5.implementation = function(){return true};
  Java.use("com.example.androiddemo.Activity.FridaActivity4$InnerClasses").check6.implementation = function(){return true};
  })
}
function main(){
    hook_java()//在main函数中调用hook_java函数
}
setImmediate(main);
```

然后进入第五关：

先用`objection`看看调用了什么函数

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_HDB7Q9KVT22223E.png)

这里我们需要分析一下代码：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_6Z3MVJ4W5AGBCRE.png)

判断`getDynamicDexCheck()`这个函数的返回值是否为空

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_MS2FFZ8RTHHZANP.png)

然后发现他是动态加载一个`dex`文件：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_ND27RS278HZWR8J.png)

然后这个`check`函数在这里声名：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_YG8SK7UZUBEXD7J.png)

我们如果直接`hook`这个`check`函数是不会成功的，所以我们要先找到这个`dex`文件：

```
function fifth(){
  Java.perform(function(){
  Java.choose("com.example.androiddemo.Activity.FridaActivity5",{
  onMatch:function(instance){
  //用classname来查看
  console.log("found instence getDynamicDexCheck():",instance.getDynamicDexCheck().$className);
  },onComplete:function(){console.log("search complete!");}
  })

  //用枚举法看看在那个类里面
  Java.enumerateClassLoaders({
  onMatch:function(loader){
  try {
  if(loader.findClass("com.example.androiddemo.Dynamic.DynamicCheck")){
  console.log("Success found loader:",loader);
  //将默认的classloader替换成loader
  Java.classFactory.loader = loader;
  }

  } catch (error) {
  console.log("found,error!"+error);
  }
  },onComplete:function(){console.log("enum complete!")}
  })

  Java.use("com.example.androiddemo.Dynamic.DynamicCheck").check.implementation = function(){return true;}
      })
}
```

最后进入第六关：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_RP49R5RQJD5674K.png)

这里我们采取枚举所有类的方法来`hook`

```
function func6() {
  Java.perform(function () {
  Java.enumerateLoadedClasses({
  onMatch: function (name, handle) {
  if (name.indexOf("com.example.androiddemo.Activity.Frida6") >= 0) {
  console.log(name);
  var frida6 = Java.use(name);
  frida6.check.implementation = function () {
  console.log("frida 6 check:", this);
  return true;
  };
  }

  }, onComplete: function () {
  }
  })
  });
}
```

## demo_02

安装好程序之后我们首先看一下程序的提示，

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_JN3AYNXTBXKMY52.png)

提示设备只能运行在俄罗斯的设备上，然后我们用`objection`看一下程序的`activity`：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_5RT2BCYKX4FM794.png)

然后竟然惊喜的发现程序可以在不同的`activity`中切换：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_KTEFB8MF9GVGTMU.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_H9RFGEEQ537B9WC.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_2EQ2UB6RN5TWHZT.png)

......这并不是我们想要做的，还是从一开始的不能在俄罗斯的设备上开始分析吧：

通过搜索字符串，我们定位到了这里：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_8G6DBFG4U5M4CG5.png)

所以我们就要`hook`这个函数的返回值`System.getProperty()`为`Russian`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_9CWZA94Q58D3JDP.png)

可以去看看`system`在哪个包下面：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_VD6GSTU6EWP8MZN.png)

接下来就可以写`js`脚本了：

```
function main(){
  Java.perform(function(){
  var System = Java.use("java.lang.System");
  console.log(System);

  System.getProperty.overload('java.lang.String').implementation = function (key) {
  var result = this.getProperty(key);
  result = "Russia";
  console.log("System.getProperty:", key, result);
  return result;
  };
  })

}
setImmediate(main)
```

`frida -U --no-pause -f com.tlamb96.spetsnazmessenger -l E:\homework\_python\frida\_env\shizhan\_02.js`

这里要注意一下，这里要使用`spwan`的方式注入，因为是在`app`启动的过程中注入的脚本

接下来手机提示这样的界面，

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_2NE49H3S8ZBS6DE.png)

继续字符串搜索看看

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_7SFMVNG9EYQFKNM.png)

根据程序的逻辑，这两个变量成员得相等：

`str.equals(getResources().getString(R.string.User))`

然后向上找找`str`：

`String str = System.getenv("USER");`

所以接下来我们就要`hook`这个`getenv`函数了

然后找到`(R.string.User)`这个的值作为函数的返回值

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_8FJVBH9HD97HQ6A.png)

`<string name="User">RkxBR3s1N0VSTDFOR180UkNIM1J9Cg==</string>`

这里注意一下，在写`js`脚本的时候之前写的`js`脚本不要删除，因为这两个都需要`hook`：

`js`脚本：

```
function main(){
  Java.perform(function(){
  var System = Java.use("java.lang.System");
  console.log(System);

  System.getProperty.overload('java.lang.String').implementation = function (key) {
  var result = this.getProperty(key);
  result = "Russia";
  console.log("System.getProperty:", key, result);
  return result;
  };

  System.getenv.overload('java.lang.String').implementation = function (key) {
  var result = this.getenv(key);
  result = "RkxBR3s1N0VSTDFOR180UkNIM1J9Cg==";
  console.log("getenv :",key,result);
  return result;
  };
  })

}
setImmediate(main)
```

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_2D8CMSG79WK8CN3.png)

成功进入这个界面：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_D8DYPJ6KRJJ5ZYN.png)

随意输入一下，程序是这样提示的：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_CXF8F65PX6NYWWA.png)

然后我们继续去`jadx`中看看字符串，定位到了这里：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_4HBSY9XRZWKW8VY.png)

然后开始分析这个程序：

```
if (this.n != null && this.o != null && !this.n.isEmpty() && !this.o.isEmpty()) {
```

这一行代码的意思是我们输入的用户名和密码都不能为空

```
if (!this.n.equals(getResources().getString(R.string.username))) {
```

这个的意思是输入的用户名要和资源文件中的相等

`<string name="username">codenameduchess</string>`

然后进入这个分支：

```
} else if (!j()) {
```

然后看看`j`函数是什么：

```
private boolean j() {
  byte[] digest = this.m.digest(this.o.getBytes());
  //这里是用户输入的密码经过m函数（跟进分析之后发现m函数是md5函数）
  String str = "";
  for (byte b : digest) {
  str = str + String.format("%x", Byte.valueOf(b));
  }
  //转换成hex字符串
  return str.equals(getResources().getString(R.string.password));
  //最后这个str要和资源文件中的password相等才可以
  //<string name="password">84e343a0486ff05530df6c705c8bb4</string>
}
```

然后直接获得这个密码的明文就可以了

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_PBY4AJPY3AYC8CA.png)

那么我们将用户名：`codenameduchess`

密码：`guest`输入文本框之后就可以了：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_Z9VHT687A4KKA34.png)

然后成功进入下一个`activity`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_MNRPWZS85JMERXR.png)
