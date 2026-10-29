
```js
function hook\_native\_sub\_8B04(){
Java.perform(function(){
var base\_address = Module.getBaseAddress("libnative-lib.so");
if(base\_address !=undefined){
console.log("开始native层hook");
//定位拦截位置
var sub\_8B04 = base\_address.add(0x8B04+1);
if(sub\_8B04){
console.log("进入native层0x8B04");
}else{
console.log("拦截器加载失败");
}
//使用拦截器
Interceptor.attach(sub\_8B04,{
//开始进入函数,传入了三个参数依次打印
onEnter:function(args){
//保存第一个参数
this.arg0 = args[0];
this.arg1 = args[1];
this.arg2 = args[2];
console.log("========>ENTER<========");
// console.log("sub\_8B04未初始化时第一个参数是===>"+hexdump(this.arg0,{length:32,header:false}));
// console.log("sub\_8B04未初始化时第一个参数是===>"+hexdump(this.arg1,{length:32,header:false}));
//console.log("sub\_8B04未初始化时第一个参数是===>"+hexdump(this.arg2,{length:32,header:false}));
var hook\_aAyzpq23ijrtffg = base\_address.add(0x1B000);
if(hook\_aAyzpq23ijrtffg!=undefined){console.log("aAyzpq23ijrtffg进入8B04的形状=====>"+hexdump(hook\_aAyzpq23ijrtffg))};
},
onLeave:function(result){
console.log("========>LEAVE<========");
console.log("sub\_8B04第一个参数处理后======>"+hexdump(this.arg0)); //打印处理后的第一个参数
//console.log("sub\_8B04得到的结果是====>"+result.readCString());
// console.log("sub\_8B04处理后第一个参数是===>"+hexdump(this.arg0,{length:32,header:false}));
console.log("sub\_8B04处理后时第一个参数是===>"+result);
// console.log("sub\_8B04处理后时第一个参数是===>"+hexdump(this.arg2,{length:32,header:false}));
}
})
}
})
}
```

码表果然发生改变

![image-20231104105804418](upload/attach/202311/940967\_X69KSGSS6857WVM.png)

看看是不是只是简单换了码表，其他地方有没有进行加密

![image-20231104110924545](upload/attach/202311/940967\_RJ2T69G7JYSNRFF.png)

是的，和猜测的结果完全一致！

再来看看码表是在哪个位置发生改变的，在`sub\_8ABC`找到码表发生变化的位置,并且发现传参`a1`就是加密前的字符串被`int`强转

![image-20231104111158979](upload/attach/202311/940967\_FV6AC424K65UVQH.png)

完整的`hook`代码如下

```js
function hook\_js(){
Java.perform(function(){
var enrandomcode = Java.use("com.kanxue.algorithmbase.MainActivity")
//如果定义好了
if(enrandomcode !=undefined)
{
console.log("开始JAVA层Hook");
enrandomcode.encodeFromJni\_12.implementation = function(intput){
var res = this.encodeFromJni\_12(intput);
console.log("传入参数是====>"+intput);
console.log("加密后的参数是====>"+res);
return res;
}
}
})
}
function hook\_native(){
Java.perform(function(){
var base\_address = Module.getBaseAddress("libnative-lib.so");
if(base\_address !=undefined){
console.log("开始native层hook");
//定位拦截位置
var sub\_8B04 = base\_address.add(0x8B04+1);
console.log("sub\_8b04:",sub\_8B04)
if(sub\_8B04){
console.log("进入native层0x8B04");
}else{
console.log("拦截器加载失败");
}
//使用拦截器
Interceptor.attach(sub\_8B04,{
//开始进入函数,传入了三个参数依次打印
onEnter:function(args){
//保存第一个参数
this.arg0 = args[0];
this.arg1 = args[1];
this.arg2 = args[2];
console.log("========>ENTER<========");
// console.log("sub\_8B04未初始化时第一个参数是===>"+this.arg0.readCString());
// console.log("sub\_8B04未初始化时第二个参数是===>"+this.arg1.readCString());
//console.log("sub\_8B04未初始化时第三个参数是===>"+hexdump(this.arg2,{length:64,header:false}));
var hook\_aAyzpq23ijrtffg = base\_address.add(0x1B000);
if(hook\_aAyzpq23ijrtffg!=undefined){console.log("aAyzpq23ijrtffg进入8B04的形状=====>"+hook\_aAyzpq23ijrtffg.readCString().length())};
},
onLeave:function(result){
console.log("========>LEAVE<========");
// console.log("sub\_8B04已经初始化时第一个参数是===>"+hexdump(this.arg0,{length:64,header:false}));
// console.log("sub\_8B04已经初始化时第二个参数是===>"+hexdump(this.arg1,{length:64,header:false}));
//console.log("sub\_8B04已经初始化时第三个参数是===>"+hexdump(this.arg2,{length:64,header:false}));
}
}
)
}
})
}
function hookencodeFromJni(){
Java.perform(function(){
var base\_address = Module.getBaseAddress("libnative-lib.so");
var base\_address\_method = Module.findExportByName("libnative-lib.so","Java\_com\_kanxue\_algorithmbase\_MainActivity\_encodeFromJni\_112")
if(base\_address\_method!=undefined)
{
console.log("=======>进入初始界面进行Hook<=======");
//使用拦截器
Interceptor.attach(base\_address\_method,{
onEnter:function(args){
//打印我们想要的param1和param2
// console.log("args[0]===>"+args[0]);
// console.log("args[1]===>"+args[1]);
// console.log("args[2]===>"+args[2]);
// console.log("args[3]===>"+args[3]);
var hook\_aAyzpq23ijrtffghijklmn = base\_address.add(0x1B000+1);
console.log("aAyzpq23ijrtffghijklmn在导出函数里面=====>"+hook\_aAyzpq23ijrtffghijklmn.readCString());
},
onLeave:function(nreval){
nreval.replace(0);
console.log(nreval);
}
})
}
})
}
function hook\_native\_sub\_8B04(){
Java.perform(function(){
var base\_address = Module.getBaseAddress("libnative-lib.so");
if(base\_address !=undefined){
console.log("开始native层hook");
//定位拦截位置
var sub\_8B04 = base\_address.add(0x8B04+1);
if(sub\_8B04){
console.log("进入native层0x8B04");
}else{
console.log("拦截器加载失败");
}
//使用拦截器
Interceptor.attach(sub\_8B04,{
//开始进入函数,传入了三个参数依次打印
onEnter:function(args){
//保存第一个参数
this.arg0 = args[0];
this.arg1 = args[1];
this.arg2 = args[2];
console.log("========>ENTER<========");
// console.log("sub\_8B04未初始化时第一个参数是===>"+hexdump(this.arg0,{length:32,header:false}));
// console.log("sub\_8B04未初始化时第一个参数是===>"+hexdump(this.arg1,{length:32,header:false}));
//console.log("sub\_8B04未初始化时第一个参数是===>"+hexdump(this.arg2,{length:32,header:false}));
var hook\_aAyzpq23ijrtffg = base\_address.add(0x1B000);
if(hook\_aAyzpq23ijrtffg!=undefined){console.log("aAyzpq23ijrtffg进入8B04的形状=====>"+hexdump(hook\_aAyzpq23ijrtffg))};
},
onLeave:function(result){
console.log("========>LEAVE<========");
console.log("sub\_8B04第一个参数处理后======>"+hexdump(this.arg0)); //打印处理后的第一个参数
//console.log("sub\_8B04得到的结果是====>"+result.readCString());
// console.log("sub\_8B04处理后第一个参数是===>"+hexdump(this.arg0,{length:32,header:false}));
console.log("sub\_8B04处理后时第一个参数是===>"+result);
// console.log("sub\_8B04处理后时第一个参数是===>"+hexdump(this.arg2,{length:32,header:false}));
}
})
}
})
}
function main(){
hook\_js();
hook\_native\_sub\_8B04()
hook\_native();
hookencodeFromJni()
}
setImmediate(main);
```

使用`C++`还原加密流程
