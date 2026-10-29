
![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_8QC7JHTTNMEDRTC.png)

然后随意输入之后发现没有什么提示，只能看代码了：

我们发现这里有程序界面中的字符串

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_5BFQQ4QTYN4UNFT.png)

我们可以猜测，接下来的操作肯定是要输入字符串，所以我们可以hook这个`com.tlamb96.kgbmessenger.b.a`函数，看看他的函数调用栈，可能就会有发现：

根据刚刚界面的参数我们可以知道，我们接下来是要hook这个构造函数的：

`public a(int i, String str, String str2, boolean z)`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_VBRUP8GW4CYSCZR.png)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `var a` `=` `Java.use("com.tlamb96.kgbmessenger.b.a")`;  `a.$init.implementation` `=` `function(i,str,str2,z){`  `this.$init(i,str,str2,z);`  `console.log("a,$init",i,str,str2,z);`  `console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));`  `}`  `})` |

调用栈取自r0ysue大佬的知识星球

`console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_T7RDKWSFYZ7Q523.png)

发现是由这个函数调用来的：

`com.tlamb96.kgbmessenger.MessengerActivity.onSendMessage`

从gadx中看看：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_P5Z95EE2QGBTQVN.png)

在127行发现确实是由这个函数调用的

然后我们发现程序的逻辑就是让这个a函数的返回值为

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_TFXV7MZXNXUVSZU.png)

然后我们用objection看一下，程序在执行过程中确实调用了这个函数：

`com.tlamb96.kgbmessenger.MessengerActivity.a`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_DCHDHEPK5MQSFU3.png)

这个a函数不是静态

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_G72RTCB5DU7F8KT.png)

所以我一开始想用主动调用的方法来着，但是通过主动调用hook这个函数之后，即使没有输入他也会hook成功，所以达不到我们想要的效果：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `Java.choose("com.tlamb96.kgbmessenger.MessengerActivity",{`  `onMatch:function(instance){`  `console.log("found instance",instance);`  `instance.a` `=` `function(str){`  `result` `=` `"V@]EAASB\u0012WZF\u0012e,a$7(&am2(3.\u0003"`;`  `return` `result;`  `}`  `},onComplete:function(){console.log("search,completed!")}`  `})` |

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_4TVHFMXK9CPNA3E.png)

发现这样是不可以的，所以还是采用`java.use`的方法来hook这个函数

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `Java.use("com.tlamb96.kgbmessenger.MessengerActivity").a.implementation` `=` `function(x){`  `var result` `=` `this.a(x);`  `console.log("a:x,result:",x,result);`  `if(x=="kanxue"){`  `result=` `Java.use("java.lang.String").$new("V@]EAASB\u0012WZF\u0012e,a$7(&am2(3.\u0003");`  `}`  `return` `result;`  `}` |

如果我们输入`kanxue`的话就会注入脚本

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_EWZ8XKSTSE8BUQX.png)

程序就会正确运行

这里我们采用了类似于爆破的方法，但是这样程序到最后是不会得到flag的

我们在程序的末尾发现了flag是由i和j函数产生的

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_TFF7Z25G72228Y3.png)

然后这个i函数里面的两个参数

`char[] charArray = this.q.substring(19).toCharArray();`

`char[] charArray2 = this.s.substring(7, 13).toCharArray();`

`q`和`s`是由我们的程序的输入框中的值得到的

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_WCXSZKZ9BGJGB7N.png)

所以我们还是要按照解密顺序一步步的来，不能直接返回两个字符串的值

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_UVZ583SU2KEA5AA.png)

这里我们用`android stdio`将`java`算法变成`dex`文件来解密

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_RQMV3E66ASRA3UE.png)

`build/intermediates/javac/debug/classes/com/example/myapplication`这个目录下是编译得`class`文件,然后将`.class`文件转换成`.jar`文件在转换成`.dex`文件，然后将`dex`文件放在`data/local/tmp`文件下，然后给他777权限，最后在js脚本中加载`dex`文件

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_6YBMRQTQ94KD357.png)

在js脚本刚开始的地方

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_3WQHNHQ7ZGER6FW.png)

然后输入执行后的结果

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_FBMC2KYTZGCG5EX.png)

继续去`jadx`中看看

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_6JUUQFC438CNWRS.png)

所以接下来就要分析这个b函数了

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_CBKRSRT4TDCJ7JM.png)

如果还是按照之前的方法就不好弄了，所以直接编写`py`脚本解密就可以了：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_SC6VVR4PHMKZFAY.png)

输入程序的运行结果之后得到`flag`：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_9ZZ85NQXF2UM4R4.png)

# so层的案例

当程序执行到这一步的时候，如果点击好吧，就会退出程序：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_FQAY2NPA8SXXJNZ.png)

然后再`jadx`中定位到关键代码

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_T2BZ8JRWWB89AVE.png)

我们可以发现

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_TX3Z2KJ23B3CTCT.png)

`sn`是我输入的字符串，然后转成字节，转成`string`类型

然后这一行代码中传入了：

|  |  |
| --- | --- |
| 1 | `((MyApp) RegActivity.this.getApplication()).saveSN(sn);` |

`saveSN`这个关键函数

我们跟进去分析一下，他是`native`层的函数

我们再从程序的入口点分析分析：

我们刚刚分析的注册的函数是从这里跳过来的：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_N4Q93J3H5B6P3AG.png)

如果我们再`doRegister()` 函数中点击注册按钮，就会执行这里

|  |  |
| --- | --- |
| 1 | `intent.setComponent(new ComponentName(BuildConfig.APPLICATION_ID,` `"com.gdufs.xman.RegActivity"`));` |

然后我们再向上回溯：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_NX7ASJ97P5UF8CV.png)

在这个`oncreate`函数中的`onclick`方法中：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_D74MZ7MSJBUMNU2.png)

根据`MyApp.m`的值判断是否需要注册

然后我们分析一下这个`MyAPP`方法，发现他的逻辑也在`so`层：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_3YBCC74FRQRFWAD.png)

所以我们要分析的就是这几个`so`层的方法了，在`ida`中打开看看：

我们在`export`中并没有发现`savesn`方法：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_ZDVYAUG5RZ8UYET.png)

所以它采用的是动态注册的方法，直接去`jni`中看看：

首先要导入`jni.h`头文件，不要忘了这一点：

这里展示一下我修复之后的`jni`函数：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_X54EGN3CNKX7CA8.png)

然后我们发现确实注册了这几个函数：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_44XME983UBEF5WD.png)

我们首先来分析第一个函数`"initSN"`

也就是那个`n1`函数：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_XSZUKTZZWWCEPCK.png)

首先打开这个文件`/sdcard/reg.dat`，我们会发现里面是一些奇怪的字符：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_SYSQPW97F5KYE6U.png)

然后将文件中的内容存贮在`v6`中：

如果`v6`的值为`"EoPAoY62@ElRD"`，那么将`v8`的值设置为1，否则将`v8`的值设置为0

然后我们看一下`n3`函数：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_2282MB2HPMX85CR.png)

然后跟进 `v2 = getValue(a1);`这里看看：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_EVA7S2K9KNWHX5Z.png)

发现这里将`m`赋值，也就是刚刚给`v8`赋的值在这里传递给了`m`

最后我们来看一下`savesn`函数：

这里我展示一下修复之后的代码：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_U6QZ9BGGAQWCYWD.png)

先打开这个文件中的内容`/sdcard/reg.dat`

然后将我们输入的字符串`str`传入`v7`：

`v7 = (*env)->GetStringUTFChars(env, str, 0);`

然后通过解密这个函数，就能得到`flag`

最后得到的`flag`为：`xman{201608Am!2333}`

先附上`js`爆破的代码：

|  |  |
| --- | --- |
