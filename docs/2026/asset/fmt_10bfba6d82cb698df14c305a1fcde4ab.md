
```js
//格式化后
function y() {
var t = (function() {
var B = !![];
return function(W, i) {
var F = B ? function() {
if (i) {
var g = i['apply'](W, arguments);
i = null;
return g;
}
} : function() {};
B = ![];
return F;
};
}());
var l = t(this, function() {
return l['toString']()['search']('(((.+)+)+)+$')['toString']()['constructor'](l)['search']('(((.+)+)+)+$');
});
l();
console['log']('aaaa');
console['log']('ccc');
};
y();
function Y() {
console['log']('bbbbb');
};
Y();
//格式化前
function y(){var t=(function(){var B=!![];return function(W,i){var F=B?function(){if(i){var g=i['apply'](W,arguments);i=null;return g;}}:function(){};B=![];return F;};}());var l=t(this,function(){return l['toString']()['search']('(((.+)+)+)+$')['toString']()['constructor'](l)['search']('(((.+)+)+)+$');});l();console['log']('aaaa');console['log']('ccc');};y();function Y(){console['log']('bbbbb');};Y();
```

执行格式化后的代码会出现递归爆炸的情况，因为匹配了换行符。

![](upload/attach/202205/865436_YY34E9QE9FFN44D.png)

# 简介

网络爬虫一直以来是让网站维护人员头痛的事情，即要为搜索引擎开方便之门，提升网站排名、广告引入等，又要面对恶意爬虫做出应对措施，避免数据被非法获取，甚至出售。因此促生出爬虫和反爬虫这场旷日持久的战斗。

爬虫的开发从最初的简单脚本到PhantomJs、selenium再进化到puppeteer、playwright等，和浏览器结合越来越密切。

反爬虫的手段从ua、Header检测到IP频率检测再到网站重构、验证码、JS加密等，手段越来越多样。

下表是爬虫攻防手段发展一个简单的对比

| 阶段 | 攻击 | 防御 |
| --- | --- | --- |
| 1 | 简单脚本编写爬虫，Python、Java等 | 通过检测User-Agent、HTTP Headers来区分是否为机器人 |
| 2 | 添加正常浏览器请求头部，伪装成正常用户访问 | 通过检测IP访问频率，分析出短时间出现访问次数异常的IP，进行封禁 |
| 3 | 使用IP代理池，秒拨等技术，拥有大量IP | 避免通过HTML访问直接获取数据，进行网站重构，使用Ajax动态传输数据 |
| 4 | 分析数据传输接口，直接访问接口获取数据 | 添加验证码、JS参数加密 |
| 5 | 深度学习破解验证码、JS调试破解参数加密 | 寻求第三方安全产品 |

反爬虫的手段到现在已经成体系化了，访问令牌（身份认证）、验证码（滑动、逻辑、三维等）、行为&指纹检测（人机区分）、请求&响应加密等。所有这些功能的实现都是依靠前端JS代码，对于攻击者，如何去绕过反爬虫手段，分析前端JS代码就成为了必经之路。那么JS如何不被破解，也成为了反爬虫的关键。

**本文只探讨JS如何防破解，其它反爬虫手段不展开讨论**

# JS防破解

JS防破解主要客户分为两个部分：`代码混淆`和`反调试`。

## 代码混淆

从代码布局、数据、控制三个方面入手，进行混淆。

### 布局混淆

常见手段有无效代码删除，常量名、变量名、函数名等标识符混淆等。

#### 无效代码删除

1. 注释文本对于理解代码逻辑有很多帮助，生产环境需要删除。
2. 调试信息对于开发者调试Bug有很大的帮助，生产环境需要删除。
3. 无用函数和数据需要删除，避免攻击者能够猜到开发者意图，和垃圾代码添加不同。
4. 缩进、换行符删除，减小代码体积，增加阅读难度。

#### 标识符重命名

1. 单字母。还可以是`aa`、`a1`等，需要注意避免作用域内标识符冲突。

   ```js
   var animal = 'shark' // 源代码
   var a = 'shark' // 重命名
   ```

2. 十六进制。

   ```js
   var animal = 'shark' // 源代码
   var _0x616e696d616c = 'shark' // 重命名
   ```

# 结语

本文只列举了一些常见的前端JS防破解手段，还有一些更高级的手段，例如：自定义编译器、WebAssembly、浏览器特性挖掘等。结合自身的业务合理的使用代码混淆和反调试手法，来保证业务不被恶意分析，避免遭到爬虫的危害。
