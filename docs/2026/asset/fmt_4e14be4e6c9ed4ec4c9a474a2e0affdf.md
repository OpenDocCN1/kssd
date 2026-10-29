
# [原创]安卓加固方案从落地加载到类指令抽取编写报告

# 一、前言以及环境配置

&ensp;&ensp;&ensp;&ensp;PS:突然想起来好久没在看雪发过啦，这次就同步一下吧！！！

&ensp;&ensp;&ensp;&ensp;PS:该文已经首发于某公众号，介意者勿喷！！！

&ensp;&ensp;&ensp;&ensp;安卓的加固方案是从19年底开始写的，到现在为止差不多快一年了，写这个目的还是学习怎么脱壳，前几个月再看雪看到有人直接分析壳来学习，不过我感觉从加壳写起也是一种浪漫。因为个人原因，在类指令抽取壳哪里为半完成状态，在今年大概率没有时间接着修改了，在java层的加固就止于此吧！！！(PS:以后有时间会接着修改)

&ensp;&ensp;环境配置：

&ensp;&ensp;&ensp;&ensp;

&ensp;&ensp;&ensp;&ensp;* Android studio v3.5.3

&ensp;&ensp;&ensp;&ensp;* 华为G621-TL00 android v4.4.4

---

# 二、第一代壳：落地加载

## 1、原理

&ensp;&ensp;&ensp;&ensp;a、原理很简单，就是首先将我们的dex文件或者apk文件解密，然后利用`DexClassLoader`加载器将其加载进内存中，然后利用反射加载待加固的apk的appkication，然后运行待加固程序即可，我画了个流程图详细说明如下：

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;![1.png](upload/attach/202009/851220_XNK2GWCRC77JXRH.jpg)

&ensp;&ensp;&ensp;&ensp;b、上面说了大概原理，现在来说明一下具体细节，我们知道，在一个app开始运行的时候，第一个加载的类是`ActivityThread`,该类有个关键属性`currentActivityThread`，通过该属性能够获取到一系列其他关键的属性，例如`mPackages`，通过该属性，我们可以获取到`mClassLoader`属性，通过替换该属性我们可以替换系统加载器，如下所示：

![2.png](upload/attach/202009/851220_78FWHCXUKXDF6NW.jpg)

&ensp;&ensp;&ensp;&ensp;接着来说怎么获取待加固apk的application，这个通过在脱壳apk的AndroidManifest.xml中使用`meta-data`来获取，如下所示：

![3.png](upload/attach/202009/851220_2KYNEXCPQVQ6MCR.jpg)

&ensp;&ensp;&ensp;&ensp;在然后就是怎么替换application，我们可以知道在android.app.LoadedApk类中有一个方法`makeApplication`可以生成一个application，通过该方法生成一个application，然后通过替换`android.content.ContentProvider`类中的`mContext`属性完成application的替换，如下图所示：

![4.png](upload/attach/202009/851220_ZX89ABK4URVTBTR.jpg)

## 2、实际操作

&ensp;&ensp;&ensp;&ensp;ps:因为第一代壳网上一大堆，所以讲得很粗略，同时这也不是本文的重点！！！

&ensp;&ensp;&ensp;&ensp;通过上面的代码我们可以得到脱壳apk，然鹅待加固的apk放在哪里，网上大多放在脱壳dex的尾部，我又画了一张图，应该可以看图就懂了：

![5.png](upload/attach/202009/851220_JJNBGGM5VR2DAKE.jpg)

&ensp;&ensp;&ensp;&ensp;这个我采用通过python读取二进制然后重新计算chunksum和签名字段实现，代码入戏：

```python
import binascii

import hashlib

import zlib

def fixCheckSum(shell):

shell.seek(0x0c)

data = shell.read()

checksum = zlib.adler32(data)

strchecksum = str(hex(checksum))

strchecksum = strchecksum.replace('0x','')

b = bytes(strchecksum.encode('utf-8'))

a = bytearray(b)

c = binascii.hexlify(binascii.unhexlify(bytes(a))[::-1])

dataCheckSum = bytearray(c)

shell.seek(0x08)

shell.write(dataCheckSum)

def fixSHA1(shell):

shell.seek(0x20)

signBytes = shell.read()

sha1 = hashlib.sha1()

sha1.update(signBytes)

sign = sha1.hexdigest()

tmp = bytes(sign.encode('utf-8'))

b = bytearray(tmp)

shell.seek(0x0c)

shell.write(b)

def fixFileSize(shell,num):

b = bytearray()

for i in range(4):

number = int(num % 256)

b.append(number)

num = num >> 8

shell.seek(0x20)

shell.write(b)

def IntToHex(num):

b = bytearray()

for i in range(4):

number = int(num % 256)

b.append(number)

num = num >> 8

b.reverse()

return b

def main():

sourceApk = open('sourceApk.apk','rb+',True)

unshell = open('unshell.dex','rb+',True)

filename = 'classes.dex'

tmpApk = sourceApk.read()

print('[\*] 成功读取待加壳的APK文件')

sourceArray = bytearray(tmpApk)

tmpDex = unshell.read()

print('[\*] 成功读取脱壳DEX文件')

unshellArray = bytearray(tmpDex)

print('[-] 待加壳APK文件开始加密，加密类型为：未加密')

sourceApkLen = len(sourceArray)

unshellLen = len(unshellArray)

print('[+] 加密后的APK大小为' + str(sourceApkLen) + 'Byte')

totalLen = sourceApkLen + unshellLen + 4

tmpByteArray = unshellArray + sourceArray

newdex = tmpByteArray + IntToHex(sourceApkLen)

print('[+] 所有二进制数据合成完毕')

shellTmp = open(filename,'wb+',True)

shellTmp.write(newdex)

shellTmp.close()

print('[+] 数据写入' + filename + '完毕')

shell = open(filename,'rb+',True)

fixFileSize(shell,len(newdex))

print('[+] 文件大小修改完毕')

fixSHA1(shell)

print('[+] 文件SHA-1签名头部修改完毕')

fixCheckSum(shell)

print('[+] 文件校验头头部修改完毕')

print('[+] 待加壳APK文件sourceApk.apk加壳完毕，加壳后DEX文件' + filename + '生成完毕')

shell.close()

if __name__ == '__main__':

main()
```

&ensp;&ensp;&ensp;&ensp;将上述apk重新签名后，安装运行，如下图所示：

![6.png](upload/attach/202009/851220_VGWJQQAM2Z5GTEY.jpg)

![7.png](upload/attach/202009/851220_CFG79P66KBTU59N.jpg)

## 3、遇到的问题

&ensp;&ensp;&ensp;&ensp;运行时报错如下所示：

![8.png](upload/attach/202009/851220_KDD54B9VG2ZS7PE.jpg)

&ensp;&ensp;&ensp;&ensp;解决方案：报错显示无法实例化activity，经过检查是无法加载到正确格式的dex文件，检查你的解密代码，即使是你加密是象征型的异或了一个0xff，解密时也不能因为异或0xff值不变而不异或0xff。其次是打包成apk之前删除签名文件之后在签名！！！

---

# 三、第二代壳：不落地加载

## 1、原理

&ensp;&ensp;&ensp;&ensp;大体原理和第一代壳相同，和第一代壳不同的是，第一代壳将dex文件解密出来会保存到文件中，在通过DexClassLoader加载进内存中，而不落地加载直接重写`DexClassLoader`使其可以直接加载字节数组，避免写入文件中。我们要做的是重写`DexClassLoader`，而这涉及到三个函数`defineClass`、`findClass`、`loadClass`，在一个类被加载的时候，会先后调用这三个函数加载一个类，所以我们需要重写这三个函数，但是我们怎么在重写的过程中操控dex中的类(通过字节数组加载进来的并不能直接操控)？其实系统的DexClassLoader加载dex进入内存的也必然是通过字节加载的，而在系统so中的`libdvm.so`中的`openDexFile`可以直接加载dex文件，那么现在清楚了，我们可以通过编写so文件调用`openDexFile`函数加载dex字节数组，值得注意的是，`openDexFile`函数返回值为一个`int`类型的cookie，可以简单理解成一个dex文件的'身份码'，通过该'身份码'即可操控这个dex文件,至于怎么调用该函数，可以通过`dlopen`和`dlsym`函数调用，相关代码如下所示：

![9.png](upload/attach/202009/851220_RDPZ3JK8J678SKE.jpg)

![10.png](upload/attach/202009/851220_BRGJV9BGQZ3RP4W.jpg)

## 2、实际操作

&ensp;&ensp;&ensp;&ensp;a、首先编写样本，这里我写了一个类和一个方法，作用就是打印一个特征字符串，如下所示：

![11.png](upload/attach/202009/851220_CAUHGHJV43P7YCF.jpg)

&ensp;&ensp;&ensp;&ensp;b、将上面的样本打包成apk后提取出dex文件然后放置到assest文件夹下(该文件夹需要自己建立)供程序调用（ps：我这里图方便未对dex文件加密然后解密，有需要的可以加上），然后脱壳apk和上面的第一代壳没什么区别，唯一不同的是就是我们使用的是我们自己重写的`DexClassLoader`，如下图所示：

![12.png](upload/attach/202009/851220_75QYA7DEG3UTNHX.jpg)

&ensp;&ensp;&ensp;&ensp;c、运行截图如下：

![13.png](upload/attach/202009/851220_7SPFQ7W96Z2N2ZB.jpg)

## 3、遇到的问题

&ensp;&ensp;&ensp;&ensp;a、报错`java.lang.UnsatisfiedLinkError: Native method not found`

&ensp;&ensp;&ensp;&ensp;解决方案：在配置文件中添加`packagingOptions{ pickFirst "lib/armeabi-v7a/libtwoshell.so" pickFirst "lib/arm64-v8a/libtwoshell.so" pickFirst "lib/x86/libtwoshell.so" pickFirst "lib/x86_64/libtwoshell.so" }`，如下所示：

![14.png](upload/attach/202009/851220_7WZCTRTAVF787U8.jpg)

&ensp;&ensp;&ensp;&ensp;b、运行到加载dex文件中的方法时，app直接闪退

&ensp;&ensp;&ensp;&ensp;解决方案：重写的`loadClass`方法有问题，不能通过直接super调用父类方法，而是应该通过反射调用`defineClassNative`方法，如下所示：

![15.png](upload/attach/202009/851220_BJKFB6RMUSP7JGX.jpg)

---

# 四、第三代壳：类指令抽取壳

## 1、原理

&ensp;&ensp;&ensp;&ensp;a、什么是类指令抽取壳，从名字就能看出来，就是把dex文件中的方法指令抽空，变成`nop`，然后在运行时再将指令还原！！！

&ensp;&ensp;&ensp;&ensp;b、指令抽取可以通过010修改，现在来说指令还原，其余代码和第二代基本一样，不一样的地方在加载完dex之后执行指令还原函数，指令还原现在有两种方法，第一种是通过读取`maps`文件获取加载的dex文件地址，然后对dex文件进行解析，找到被nop的指令处进行还原(ps：该种方法需要及其熟悉dex文件格式，不了解的可以看我之前的文章关于解析dex文件，因为我之前解析的时候用的是python，改成c要大量时间，所以我选择了第二种方法)；第二种方法就是通过免root hook系统函数(最简单的就是deFindClass函数)然后进行指令还原！！！

&ensp;&ensp;&ensp;&ensp;c、接下来就将一下怎么通过hook dexFindClass函数来进行指令还原(PS:看懂下面的内容需要理解dex文件格式)。`dexFindClass`函数在`libdvm.so`库中，如下所示：

![16.png](upload/attach/202009/851220_UN8A7DQH2DYKFKY.jpg)

&ensp;&ensp;&ensp;&ensp;免root hook框架有点多，我选择的是android inline hook，原因很简单，很适合在so层使用，其他的经过我测试不知道为啥我写出来的没反应，该框架github地址：<a href="elink@9a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8r3f1%4k6h3&6^5P5r3S2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1d9h3&6D9K9h3&6W2i4K6u0V1d9r3!0G2K9H3`.`.</a>，用法可以参考作者github，该inline hook框架需要原函数地址、新函数地址和原始函数的二级指针，用法如下所示(怎么使用不是重点，接下来的才是重点，所以这里比较粗略)：

![17.png](upload/attach/202009/851220_9AGUMDNRBHNTKMB.jpg)

&ensp;&ensp;&ensp;&ensp;我们要hook的是`dexFindClass`函数，该函数定义在`DexFile.h`文件中，该函数返回值为一个类结构指针，第二个参数为类名字，通过该参数我们就可以指定类进行指令还原，如下所示：

![18.png](upload/attach/202009/851220_NBG5MVXEEPAGR8W.jpg)

![19.png](upload/attach/202009/851220_PFDS87N97YBSUSJ.jpg)

&ensp;&ensp;&ensp;&ensp;上面我们得到的`classDataOff`，我们可以通过该地址获取到类数据，该偏移地址指向的是一个`DexClassData`结构，该结构的`header`存储了相关类信息，该结构的`directMethods`指针指向的方法的结构题，如下所示：

![20.png](upload/attach/202009/851220_JWG45G4UTX7REZ9.jpg)
