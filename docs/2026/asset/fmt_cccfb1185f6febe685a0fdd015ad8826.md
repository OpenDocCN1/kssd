
```
#ifdef __cplusplus

}

#endif

#endif
```

**未Hook前运行截图：**

![18.png](upload/attach/202003/851220_A9UNKFD9WSZ24TY.jpg)

**&ensp;&ensp;&ensp;&ensp;2、接下来是python Hook脚本（只解释与上面有差异的代码），Hook的函数具体函数名还是使用IDA去寻找**

![19.png](upload/attach/202003/851220_MEM4S7ZSPVJEFC9.jpg)

**python代码：**

```
import frida

import sys

jscode = """

Java.perform(function(){

Interceptor.attach(Module.findExportByName("libfridaso.so","Java_com_example_fridasostring_fridaSoString_FridaSo"),{

onEnter: function(args) {

send("Hook start");

send("args[2]=" + args[2]);

},

onLeave: function(retval){

send("return:"+retval);

var env = Java.vm.getEnv(); //获取env对象，也就是native函数的第一个参数

var jstrings = env.newStringUtf("tamper"); //因为返回的是字符串指针，使用我们需要构造一个newStringUtf对象，用来代替这个指针

retval.replace(jstrings); //替换返回值

}

});

});

"""

def printMessage(message,data):

if message['type'] == 'send':

print('[*] {0}'.format(message['payload']))

else:

print(message)

process = frida.get_remote_device().attach('com.example.fridasostring')

script = process.create_script(jscode)

script.on('message',printMessage)

script.load()

sys.stdin.read()
```

**&ensp;&ensp;&ensp;&ensp;3、运行脚本后，点击按钮，我们可以看到字符串已经被替换成了`tamper`，如下所示：**

![20.png](upload/attach/202003/851220_5JWEY9QXT7HRSZZ.jpg)

![21.png](upload/attach/202003/851220_PM77BUDFG9D37F6.jpg)

**&ensp;&ensp;&ensp;&ensp;4、关于为导出函数的Hook，大体上差不多，差别在于需要通过ida找到偏移值计算地址，而不是像导出函数这么方便罢了，但原理都是差不多的，就不在细说了！！！**

---

## 五、相关附件

**&ensp;&ensp;&ensp;&ensp;python Hook代码已经demo下载链接：<a href="elink@bbcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4h3V1y4u0k6f1A6j5P5X3g2f1M7q4p5^5N6f1Z5&6c8i4M7#2L8X3&6s2f1b7`.`."><mark class="encrypted">945K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4h3V1y4u0k6f1A6j5P5X3g2f1M7q4p5^5N6f1Z5&6c8i4M7#2L8X3&6s2f1b7`.`.</mark></a>**

**&ensp;&ensp;&ensp;&ensp;提取码：z94i**

---

## 一、Frida概述

**Frida是一款轻量级HOOK框架，可用于多平台上，例如android、windows、ios等。**

**frida分为两部分，服务端运行在目标机上，通过注入进程的方式来实现劫持应用函数，另一部分运行在系统机器上。**

**frida上层接口支持js、python、c等。**

**Frida官方github地址为：[frida官方github地址](https://bbs.kanxue.com/elink@838K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5b7`.`.)**

---

## 二、Frida安装

**1 、安装`python3.7`并配置好环境变量（官方推荐`python3`以上版本至少为3.7），python安装包官方下载地址：[d9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3P5i4c8Z5L8$3&6Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1j5`.](https://bbs.kanxue.com/elink@6e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3P5i4c8Z5L8$3&6Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1j5`.)。**

**2 、安装frida模块，命令为`pip install frida`(配置了多个python版本环境的可以使用命令`python -m pip install frida`防止用`pip install frida`命令报错)。**

![1.png](https://bbs.kanxue.com/upload/attach/202003/851220_S9A734FNVRBPQUK.jpg)

**3、安装frida-tools模块，命令同上，`pip install frida-tools`或者`python -m pip install frida-tools`。**

![2.png](https://bbs.kanxue.com/upload/attach/202003/851220_RFWXG9N8AQR3K3N.jpg)

**4、下载运行在目标机上的frida-sever端，官方下载地址：[250K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2i4K6u0r3M7X3g2D9k6h3q4K6k6i4x3`.](https://bbs.kanxue.com/elink@b82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2i4K6u0r3M7X3g2D9k6h3q4K6k6i4x3`.)，下载时要选择对应的版本下载，例如我的机器为arm32为架构，就选择`frida-server-12.8.14-android-arm.xz`下载。（可以在`adb`使用命令`cat /proc/cpuinfo`查询）**

![3.png](https://bbs.kanxue.com/upload/attach/202003/851220_ADC5DYYBQJ7S4FS.jpg)

![4.png](https://bbs.kanxue.com/upload/attach/202003/851220_T73HFVJ8J8SCTC3.jpg)

**5、将第四步下载好的文件解压，然后通过命令`adb push 你的电脑是存放位置 /data/local/tmp`将文件传输到手机中，然后通过`adb shell`进入手机端，给文件赋权777，并于root权限启动。**

![5.png](https://bbs.kanxue.com/upload/attach/202003/851220_FRJZGBBFBC6UVMC.jpg)

![6.png](https://bbs.kanxue.com/upload/attach/202003/851220_GZRZGWKBTWPBGQE.jpg)

**6、做完以上几步后，新开一个命令行输入命令`frida-ps -U`查看手机进程，如果出现以下结果，则frida安装成功。**

![7.png](https://bbs.kanxue.com/upload/attach/202003/851220_67Y5F8K48S7U82P.jpg)

---

## 三、Frida Hook Java层

**1、编写一个小demo用来hook，该demo关键部分代码如下：**

![8.png](https://bbs.kanxue.com/upload/attach/202003/851220_6RZ9PGRHM685FPU.jpg)

**2、现在我们将该apk安装好，运行看一下未Hook前的显示字符串！！！**

![9.png](https://bbs.kanxue.com/upload/attach/202003/851220_JEK4G77HPPMVEG6.jpg)

**3、现在来编写Hook的Python脚本，脚本代码如下：**

```
import frida  #导入frida模块

import sys    #导入sys模块



jscode = """  #从此处开始定义用来Hook的javascript代码

    Java.perform(function(){  

        var MainActivity = Java.use('com.example.testfrida.MainActivity'); //获得MainActivity类

        MainActivity.testFrida.implementation = function(){ //Hook testFrida函数，用js自己实现

            send('Statr! Hook!'); //发送信息，用于回调python中的函数

            return 'Change String!' //劫持返回值，修改为我们想要返回的字符串

        }

    });

"""



def on_message(message,data): #js中执行send函数后要回调的函数

    print(message)



process = frida.get_remote_device().attach('com.example.testfrida') #得到设备并劫持进程com.example.testfrida（该开始用get_usb_device函数用来获取设备，但是一直报错找不到设备，改用get_remote_device函数即可解决这个问题）

script = process.create_script(jscode) #创建js脚本

script.on('message',on_message) #加载回调函数，也就是js中执行send函数规定要执行的python函数

script.load() #加载脚本

sys.stdin.read()
```

**4、现在python脚本编写完毕，我们来执行该脚本，首先手机端执行frida，然后通过命令`adb forward tcp:27043 tcp:27043`和`adb forward tcp:27042 tcp:27042`来转发这两个端口，接着在手机上运行该应用程序，在命令行中执行脚本，最后点击应用的按钮，即可看到字符串已经被替换成我们要替换的了！！！**

![6.png](https://bbs.kanxue.com/upload/attach/202003/851220_GZRZGWKBTWPBGQE.jpg)

![10.png](https://bbs.kanxue.com/upload/attach/202003/851220_M79QJUNWDG2A7Q8.jpg)

![11.png](https://bbs.kanxue.com/upload/attach/202003/851220_RQ6RKEZPE2VBEAQ.jpg)

![12.png](https://bbs.kanxue.com/upload/attach/202003/851220_2R4NR7ETCCVN738.jpg)

---

## 四、Frida Hook Native层

### 4.1、Hook native层返回值为int类型的demo

**1、还是先写一个小demo，下面贴一下关键代码（很简单c语言代码就不再解释了，至于native层函数怎么编写，由于本篇主要不是讲怎么编写so函数，就不过多叙述了，实在不会的可以看一下我的一篇博客，我觉得写得还是挺详细的，博客编写native层函数链接：[cd7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5g2N6^5N6X3c8E0g2Y4y4W2h3r3x3H3i4K6u0r3M7q4)9J5c8U0p5I4y4e0j5@1z5o6l9&6i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@6caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5g2N6^5N6X3c8E0g2Y4y4W2h3r3x3H3i4K6u0r3M7q4)9J5c8U0p5I4y4e0j5@1z5o6l9&6i4K6u0W2K9s2c8E0L8l9`.`.)）和未Hook前截图：**

![23.png](https://bbs.kanxue.com/upload/attach/202003/851220_GSW47YWSBDYQCN5.jpg)

**代码：**

```
/* DO NOT EDIT THIS FILE - it is machine generated */

#include <jni.h>

/* Header for class com_example_fridaso_FridaSoDefine */



#ifndef _Included_com_example_fridaso_FridaSoDefine

#define _Included_com_example_fridaso_FridaSoDefine

#ifdef __cplusplus

extern "C" {

#endif

/*

* Class:     com_example_fridaso_FridaSoDefine

* Method:    FridaSo

* Signature: (II)I

*/

JNIEXPORT jint JNICALL Java_com_example_fridaso_FridaSoDefine_FridaSo(JNIEnv *env, jclass obj, jint a, jint b)

{

    int c;

    c = a + b;

    return c;

}



#ifdef __cplusplus

}

#endif

#endif  
```

![13.png](https://bbs.kanxue.com/upload/attach/202003/851220_D8JNQBW3GWCNSYP.jpg)

![14.png](https://bbs.kanxue.com/upload/attach/202003/851220_MG92HDNB7TYNCWP.jpg)

**2、接下来我们来写python hook脚本，我们需要hook native层这个函数，达到返回值修改为0的效果。写到这里需要说明一下关于so文件当中的函数，分为导出函数和未导出函数两种，导出函数打开IDA后能够在导出表中找到的函数就是导出函数，未导出函数则在导出表中寻找不到，一般来说静态编写的native函数都能在导出表中寻找到，而动态加载的则无法在导出表中发现！！！**

![15.png](https://bbs.kanxue.com/upload/attach/202003/851220_XA58CG73X8GCXD3.jpg)

**代码如下：（跟上面hook java层重复的代码不在注释详讲了！！！）**
