
```js
function pass_frida(){
var module = Process.findModuleByName("libDexHelper.so")
var base = module.base
var sub_9DE68 = base.add(0x9DE68)
Interceptor.attach(sub_9DE68, {
onEnter:function(args){
},
onLeave:function(ret){
if(ret != 0){
ret.replace(0)
}
}
})
}
```

除了通过hook `svc`调用来定位，还可以通过hook `pthread_create`来获取创建的所有线程来定位检测

```shell
>>> hex(0x000000764F830C5C - 0x000000764F792000)
'0x9ec5c'
>>> hex(0x000000764F83C97C - 0x000000764F792000)
'0xaa97c'
>>> hex(0x000000764F82F73C - 0x000000764F792000)
'0x9d73c'
>>> hex(0x000000764F875FD0 - 0x000000764F792000)
'0xe3fd0'
>>> hex(0x000000764F82F030 - 0x000000764F792000)
'0x9d030'
```

此时`frida`已经可以使用了

继续测试，会发现对部分`libc`库函数(比如`open`)进行`hook`时，一样会被检测到

猜测是存在代码的`hash`值校验，或比较了内存与文件中的代码是否一致

### ida调试检测分析

逆向分析时，肯定是少不了动态调试的，且遇到反调试肯定也是不可避免的

首先也是通过对`openat`进行`hook`来定位检测

![](upload/attach/202307/893279_27VAC68PTJGKKY5.jpg)

定位到线程`0xaa97c`

![](upload/attach/202307/893279_EPXSJHR3EBUX66M.jpg)

也是很常规的检测方式

![](upload/attach/202307/893279_BKFKRFFBK4FMEQQ.jpg)

简单修改一下，`JNI_OnLoad`就可以正常调试了

### 此时此刻，想要彻底攻克壳的保护，需要的就只剩下时间和耐心了

样本名`shansong`，某`bang`的壳

本文主要是分享在逆向中遇到的对抗以及处理办法

### 文件格式的处理

`010editor`打开`libDexHelper.so`

一眼假的`section`

操作系统加载并不关心`section`，先直接删除掉，后面修复`dynamic`后再重新生成一份

![](https://bbs.kanxue.com/upload/attach/202307/893279_RXZXQM5XCDZWKV6.jpg)

一眼假的`dynamic`

`dynamic`除了`vaddr`，其他成员都是可以伪造欺骗的，因为操作系统也只关心`dynamic`的`vaddr`

![](https://bbs.kanxue.com/upload/attach/202307/893279_88XG59X4VE8MRAG.jpg)

通过分析`PT_LOAD`的信息，即`RVA`和`FA`的转换，对`dynamic`的`file_offset`进行简单修复

![](https://bbs.kanxue.com/upload/attach/202307/893279_ZGBQEHX5ZV3NT5C.jpg)

跑工具重新生成了一份`section`后，`ida`已经可以正常的解析`so`了

### so解密后dump

简单看了一下`init_arrary`和`JNI_OnLoad`方法，都是被加密的

![](https://bbs.kanxue.com/upload/attach/202307/893279_NMPEWMPYV78DJMT.jpg)

猜测解密方法在`.init`方法中

暂时不去关心解密算法的实现，等`.init`方法运行结束后，直接`dump`内存，可以看到数据已经解密完成

`dump`内存可以写`ida`脚本，也可以直接使用`dd`来`dump`(注意先要使用`ida`把`app`挂起，壳对`/proc/self/mem`的读写做了监控)

![](https://bbs.kanxue.com/upload/attach/202307/893279_4D8ZR9RAZFAWQU6.jpg)

### 混淆的简单处理

#### ida识别寄存器跳转失败

类似于这样

![](https://bbs.kanxue.com/upload/attach/202307/893279_YEFR7D3J5KY7TZH.jpg)

实际是`ida`对 **switch-case**的识别失败

![](https://bbs.kanxue.com/upload/attach/202307/893279_7N5HTEXR9GJGVSW.jpg)

解决办法就是帮助`ida`正确识别 **switch-case**，具体可以参考[这篇文章](https://bbs.kanxue.com/elink@374K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1N6h3E0W2y4U0j5$3i4K6u0W2j5$3&6Q4x3V1j5J5x3o6t1I4i4K6u0r3x3o6S2Q4x3V1k6e0M7r3g2U0K9h3k6&6i4K6u0V1M7%4N6A6N6r3y4Z5i4K6u0V1M7%4c8S2N6r3g2E0k6h3&6@1i4K6u0V1K9h3&6Q4x3X3c8u0c8p5q4Q4x3X3c8b7M7X3!0Q4x3V1j5`.)

正确修复后，`ida`也就能正常反编译了

![](https://bbs.kanxue.com/upload/attach/202307/893279_CTQDVNEVG6NRH7Y.jpg)

#### 平坦化的处理

代码的正常流程被`switch-case`打乱了

简单分析了一下汇编的代码结构，可以发现直接跳转到分发块的`case`块，会直接写死下一个要执行的`case`块

于是可以通过计算获取到目标`case`块的地址，将`case`块跳回分发块，改成直接跳转到对应的`case`块的地址

![](https://bbs.kanxue.com/upload/attach/202307/893279_8AUV7G9299PWPXZ.jpg)

对于跳回`def`块的`case`块，则会通过条件执行来决定下一个要执行的`case`块

同样可以计算出不同条件所要跳转的`case`块，将对应的跳转改为条件跳转

![](https://bbs.kanxue.com/upload/attach/202307/893279_EN59T7FFPWPWX58.jpg)

脚本还有待完善，有一些跳转的修改可能需要手动去修改

|  |  |

| --- | --- |
