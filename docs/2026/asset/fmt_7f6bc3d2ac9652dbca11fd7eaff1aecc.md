
```
data: {
    "apiVersion": "t141",
    "appVersion": "1.5.1",
    "areaCode": "CS000016",
    "channel": "online",
    "clientid": "a7ea53059fc868e2e3e2dd7c04027035",
    "device_id": "tv179yrhs3kv9RXjJv6uJNmdkN6kTbmaUHQE",
    "time": 1626080760465,
    "reRule": "4",
    "token": "7ae362df162da5ffbfc408ed8e3d4ff3",
    "viewSize": "720x1184",
    "networkType": "wifi",
    "isSimulator": false,
    "osType": "4",
    "scopeType": 1,
    "businessType": 2,
    "businessId": "17210001",
    "deliveryCircleType": "1",
    "body": {
        "longitude": "MTIwLjE1NDc3NQ==",
        "latitude": "MzAuMzA1ODIy"
    }
}
h5: yx_touch
paramsMD5: iOWz8O+xL9r9GX4k5Te/2U5HGTRk1GQ6YqLnMErWrAI=
```

手机重复操作，经多次调用抓包该接口后对比发现：

`h5` 这个值是固定的`yx_touch`

`paramsMD5` 通过字面意思判断为加密参数，但其数据格式不像MD5，猜测是用了MD5后又进行了其他的编码加密

观察可知获取门店要传入的经纬度入参也是加密的，正常来说经纬度均是数字

`{"longitude":"MTIwLjE1NDc3NQ==","latitude":"MzAuMzA1ODIy"}`

### 破解目标

1. **paramsMD5加密逻辑**

2. **经纬度加密逻辑**

# 初领妙道

## 逆向之旅

### 获取微信小程序包wxapkg

#### 所需工具

前述提到的RE文件管理器app

#### 小程序主子包判断依据

如今微信小程序单包体积不能超过4M（小程序基础依赖包除外），如果项目内容过大，开发者会使用分包模式

拿下图举例来说（下图所示小程序包是其他应用的，非本文要分析的case）

![image-20210712164945927](https://bbs.kanxue.com/upload/attach/202107/928237_MTTH9FQPPEZPYN6.jpg)

其中：

- `_2124598774_821.wxapkg` 3.3M 主包
- `_-588782754_76.wxapkg` 1.5M 子包
- `_152740959_13.wxapkg` 89k 子包
- `_1123949441_552.wxapkg` 14M 基础依赖包

#### 操作

打开小程序一顿操作后，会在小程序包存放目录下自动下载生成对应的包

通过re文件管理器直捣微信小程序包路径：

`/data/data/com.tencent.mm/MicroMsg/"$用户MD5"/appbrand/pkg/_*_xxx.wxapkg`

通过re文件管理器打成zip包发送到个人钉钉或者QQ、微信等，电脑完成文件接收

提示：若在之前打开过多个小程序，可以先进入目录全部删除，这样好区分小程序包的归属

### 反编译

#### 工具

##### wxUnpacker

下载地址：2b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3N6#2L8K6b7&6x3U0t1%4x3K6M7%4x3q4)9J5c8Y4N6^5j5i4m8H3g2h3&6H3j5h3y4C8k6i4t1`.

运行前提需要安装`node`环境

该工具运行需要一些`node`依赖库，安装指引在链接中`README.md`文档中有

#### 原理

等我弄明白了~， 有基础的同学可以参考这个 e3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0c8n7k6i4u0m8x3f1W2B7x3@1u0X3e0h3g2Y4x3V1I4m8x3r3y4E0y4h3M7`.

笔者太菜，看的不太懂~

#### 具体命令

```
# 主包反编译
node wxWxapkg.js ../../wxapkg/xxxx/_-2094256841_77.wxapkg
# 子包反编译
node wxWxapkg.js -s=/Users/toretto/crack/wxapkg/xxxx/_-2094256841_77 ../../wxapkg/xxxx/_571009734_77.wxapkg
....
#部分子包反编译可能会报错，但没关系，不影响后续的加密分析过程
```

# 渐入佳境

## 加密分析

### 工具

#### 微信开发者工具

下载地址：d11K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2N6$3g2A6P5r3W2F1i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3&6A6M7s2u0G2k6%4u0S2L8g2)9J5c8X3c8W2N6W2)9J5c8X3c8W2N6Y4c8G2L8$3I4K6i4K6u0r3M7%4c8S2j5X3I4W2i4K6u0W2K9s2c8E0L8l9`.`.

用于阅读代码，代码跳转追踪

### 分析环境预备

1. 导入反编译主包目录

![image-20210712165335021](https://bbs.kanxue.com/upload/attach/202107/928237_VZ8EVBJPS7DGSEH.jpg)

2. 起个名字，点击测试号生成个AppID，创建小程序

![image-20210712165410331](https://bbs.kanxue.com/upload/attach/202107/928237_Y8C6QK6PJB4EXVZ.jpg)

### 静态分析

#### paramsMD5分析

是骡子是马拉出来溜溜，不是有个加密叫paramsMD5吗，全局搜索试试看：

1. 关键位置定位

好嘛，定位到2处代码，直觉告诉我选`request.js`中的，直接定位到一个函数`getHmacSha256(n)`。

再看看这个`data`结构，含`data`、`h5`、`paramMD5`，和之前接口分析的结论一致，通用格式没的说~

![image-20210712165507451](https://bbs.kanxue.com/upload/attach/202107/928237_AF5EV58W9UN5GHQ.jpg)

2. `getHmacSha256(n)`点进去

```
function getHmacsha256(e) {
    var n = JSON.stringify(e) + e.isSimulator + e.viewSize + e.networkType + e.time, t = _common2.default.environment === _config.ENVIRONMENTS.BETA ? "@yx789*&^DKJ##CC" : "@653yx#*^&HrTy99";
    console.log("request.js@32 n: " + n);
    return _encBase2.default.stringify((0, _hmacSha2.default)(n, t));
}
```

变量`n`：

通过`JSON.stringify(e)`可以猜测，参数`e`是接口数据中的`data`，为json对象。`n`的逻辑基本上可以锁定是`data`本身的字符串再拼接上`data`的几个key对应的value值

变量`t`：

三目运算表达式，为`true`时貌似表示是BETA版本运行，那么正常使用的版本应该是`false`，所以猜测`t`=`"@653yx#*^&HrTy99"`，是一个固定盐值。

将前述接口分析中的参数`data`进行复制，粘贴进来改一下代码再加一个打印语句执行调试看看结果：

```
var a = {
    "apiVersion": "t141",
    "appVersion": "1.5.1",
    "areaCode": "CS000016",
    "channel": "online",
    "clientid": "a7ea53059fc868e2e3e2dd7c04027035",
    "device_id": "tv179yrhs3kv9RXjJv6uJNmdkN6kTbmaUHQE",
    "time": 1626080760465,
    "reRule": "4",
    "token": "7ae362df162da5ffbfc408ed8e3d4ff3",
    "viewSize": "720x1184",
    "networkType": "wifi",
    "isSimulator": false,
    "osType": "4",
    "scopeType": 1,
    "businessType": 2,
    "businessId": "17210001",
    "deliveryCircleType": "1",
    "body": {
        "longitude": "MTIwLjE1NDc3NQ==",
        "latitude": "MzAuMzA1ODIy"
    }
};

function getHmacsha256(e) {
    // var n = JSON.stringify(e) + e.isSimulator + e.viewSize + e.networkType + e.time, t = _common2.default.environment === _config.ENVIRONMENTS.BETA ? "@yx789*&^DKJ##CC" : "@653yx#*^&HrTy99";
    var n = JSON.stringify(e) + e.isSimulator + e.viewSize + e.networkType + e.time, t = "@653yx#*^&HrTy99";
    console.log("request.js@32 n: " + n);
    return _encBase2.default.stringify((0, _hmacSha2.default)(n, t));
}

console.log(getHmacsha256(a));
```

```
#运行
node request.js
# 运行结果报错：
regeneratorRuntime is not defined     在 comment.js中
```

针对这个错误，百度了一番找到个解决方案：

[bf7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4A6L8X3q4@1i4K6g2X3x3K6x3I4z5o6b7^5z5o6m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3#2y4e0x3K6x3o6V1#2](https://bbs.kanxue.com/elink@802K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4A6L8X3q4@1i4K6g2X3x3K6x3I4z5o6b7^5z5o6m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3#2y4e0x3K6x3o6V1#2)

小程序使用`async`出现`regeneratorRuntime is not defined`错误说是少个依赖库，下载之

```
#生成package.json
npm init
#下载缺少的包
npm install regenerator@0.13.1
# 将所缺文缺runtime.js移动到项目中
cd node_modules/regenerator-runtime/
#与common.js同目录
cp runtime.js /Users/toretto/crack/wxapkg/darunfa/_-2094256841_77/service/
```

修改`common.js`代码引入该包：

```
// 最上方添加
import regeneratorRuntime from './runtime.js'
```

```
#再次运行
node request.js
#还报错：
can not import modules from outside
#不能从外部导入文件，没有js基础的我盲猜可能是微信小程序无此语法(因为代码全局搜索import关键字后没有任何匹配项)。
```

**解决思路**

我看到小程序代码中有这样的片段：

```
var _wepy = require("./../npm/wepy/lib/wepy.js"),
```

看起来就是引入库的方式，于是我学了下写了这样一段：

```
var regeneratorRuntime = require('./runtime.js');
```

再次运行：

```
node reuqest.js
iOWz8O+xL9r9GX4k5Te/2U5HGTRk1GQ6YqLnMErWrAI=
//呵，可以运行了，照猫画虎成功~
//打印输出的加密和接口获取的一致
```

继续分析它的生成逻辑，追到相关代码，添加打印语句：
