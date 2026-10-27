
# [分享]一篇贴子中的JS脚本混淆

## 起因

昨天无意中看到了论坛内的一篇帖子 https://bbs.pediy.com/thread-260318.htm，当然看过的小伙伴不要指望我完成他未竟的事业，毕竟我对这安卓破解不是太在行而且我的老司机APP每日观看最大次数验证也是过不了，哈哈。

## 目的

那这篇贴子主要目的是因为他在文中提到了一个JS脚本的混淆，我想试着解一下他这个混淆，第二就是分享一个我之前用 `python` 写的一个解 `js` 混淆的脚本（只有两种格式的 `JS` 混淆，算是抛砖引玉了）

## 过程

原文中的混淆是这样的

![](upload/attach/202102/868245\_8VKA2JQUANM3JEU.png)

一开始的时候我也是蒙的，不管他了索性不如先手动排一下版吧，也许就能发现什么

```
eval(
function (p,a,c,k,e,r)
{
e=function(c){
return(c<a?'':e(parseInt(c/a)))+((c=c%a)>35?String.fromCharCode(c+29):c.toString(36))
};
if(!''.replace(/^/,String)){
while(c--)
r[e(c)]=k[c]||e(c);
k=[function(e){return r[e]}];
e=function(){
return'\\w+'
};
c=1
};
while(c--)
if(k[c])
p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c]);
return p
}
```

当我们把这个貌似解密函数排出来之后，发现其有六个参数，之后会发现紧跟着这个函数的就是它的参数。既然解密函数和参数都有了，那就简单了，打开浏览器（这里以 `windows` 自带的 `Edge` 为例），按下 `F12` 进入控制台，随后复制解密函数及其参数，然后再点击运行就能够得到我们想要的结果了。

![](upload/attach/202102/868245\_JC4QTWQZ2HPBCYK.png)

需要注意的就是加上函数名，别把 `eval` 也复制进去，最后再 `return p` 之前打印这个 `p` 就是解密之后的脚本。

## 结果

最后把红色的结果复制下来就行了，然后可以用 `NotePad++` 的 `JSTool` 插件中的 `JSFormat` 进行自动排版即可
