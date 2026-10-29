
```
 //so层hook

        //导出函数

        //var exports = Module.enumerateExportsSync("libnative-lib.so");

        //for(var i=0;i<exports.length;i++){

        //    send("name:"+exports[i].name+"  address:"+exports[i].address);

        // }



        //遍历模块找基址

        Process.enumerateModules({

            onMatch: function (exp) {

                if (exp.name == 'libnative-lib.so') {

                    send('enumerateModules find');

                    send(exp.name + "|" + exp.base + "|" + exp.size + "|" + exp.path);

                    send(exp);

                    return 'stop';

                }

            },

            onComplete: function () {

                send('enumerateModules stop');

            }

        });



        //通过模块名直接查找基址

        var soAddr = Module.findBaseAddress("libnative-lib.so");

        send("soAddr:" + soAddr);





        //   hook导出函数 通过函数名

        send("findExportByName add():" + Module.findExportByName("libnative-lib.so", "_Z3addii"));

        //send("findExportByName edit():"+Module.findExportByName("libnative-lib.so", "_ZL4editP7_JNIEnvP8_jobjecti"))

        // Interceptor.attach(Module.findExportByName("xxx.so", "xxxx"), {

        //     onEnter: function (args) {

        //         send("open(" + Memory.readCString(args[0]) + "," + args[1] + ")");

        //     },

        //     onLeave: function (retval) {

        //

        //     }

        // });





        //say(JNIEnv *env, jobject)

        //edit(JNIEnv *env, jobject,int)

        //mystr(JNIEnv *env, jobject,jstring s)

        //myarray(JNIEnv *env, jobject obj, jobjectArray array)



        // arm64-v8a 下地址

        //var fadd = 0xED7C;

        //var fsay=0xF12C;

        //var fedit=0xF04C;

        //var fmystr=0xF328;

        //var fmyarray=0xF4DC;



        //下面为x86模拟器中地址偏移  arm真机下thumb指令下地址+1

        var fadd = 0x8AD0;

        var fsay = 0x8FA0;

        var fedit = 0x8E70;

        var fmystr = 0x9200;

        var fmyarray = 0x9420;



        //armeabi-v7a

        //var fadd = 0x839C;





        var faddptr = new NativePointer(soAddr).add(fadd);//得到内存地址

        send("函数add() faddptr:" + faddptr);

        //调用add（5，6）

        var funadd = new NativeFunction(faddptr, "int", ['int', 'int']);

        var t = funadd(5, 6);

        send("调用native 方法fun():" + t);

        Interceptor.attach(faddptr, {

            onEnter: function (args) {

                send("onEnter add()");

                x = args[0];

                y = args[1];

                args[0] = ptr(x * 2);

                args[1] = ptr(y * 2);

                send("hook add()修改参数为原来的两倍 args[0]:" + args[0].toInt32() + "  args[1]:" + args[1].toInt32());

            },

            onLeave: function (retval) {

                send("onLeave  add()");

                //retval.replace(678);

                //send("add()修改返回值为："+retval.toInt32())

            }



        });



        var fsayptr = new NativePointer(soAddr).add(fsay);

        Interceptor.attach(fsayptr, {

            onEnter: function (args) {

                send("onEnter say()");

            },

            onLeave: function (retval) {

                send("onLeave say()");

                //jstring类型无法直接输出显示，可以类型转换到java.lang.String

                var s = Java.cast(retval, str);

                send("say() 原返回值：" + s);

                //调用env下的方法，构造jstring类型

                var env = Java.vm.getEnv();

                var jstring = env.newStringUtf("frida hook native");

                retval.replace(ptr(jstring));

                send("修改say()返回值:" + Java.cast(jstring, str));

            }

        });





        var feditptr = new NativePointer(soAddr).add(fedit);

        Interceptor.attach(feditptr, {

            onEnter: function (args) {

                send("onEnter edit()");

                send("edit() env：" + args[0] + "  jobject：" + args[1] + " jint:" + args[2].toInt32());

                //参数修改使用new NativePointer(s)  简写ptr(s)

                args[2] = ptr(4);

                send("hook edit() 修改后的参数jint：" + args[2]);

            },

            onLeave: function (retval) {

                send("onLeave edit()");

            }

        });



        var fmystrptr = new NativePointer(soAddr).add(fmystr);

        send("fmystrptr:" + fmystrptr);

        Interceptor.attach(fmystrptr, {

            onEnter: function (args) {

                send("onEnter mystr()");

                send("mystr() env：" + args[0] + "  jobject：" + args[1] + " jstring:" + args[2]);

                var s = Java.cast(args[2], str);

                send("mystr() jstring参数：" + s);



                //send("mystr："+Memory.readUtf16String(args[2],7));

                //send("mystr："+Memory.readUtf8String(args[2],7));

            },

            onLeave: function (retval) {

                send("onLeave mystr()");

                var env = Java.vm.getEnv();

                var jstring = env.newStringUtf("frida hook native");

                send("修改返回值jstring:" + jstring);

                retval.replace(ptr(jstring));

            }

        });

        // Java.choose("com.example.goal.DiyClass",{

        //     onMatch:function(instance){

        //         send("DiyClass instance:"+instance);

        //     },

        //     onComplete:function(){

        //

        //     }

        // });

        var fmyarrayptr = ptr(soAddr).add(fmyarray);

        //var fmyarrayptr = new NativePointer(soAddr).add(fmyarray);

        send("fmyarrayptr:" + fmyarrayptr);

        //var argptr;

        Interceptor.attach(fmyarrayptr, {

            onEnter: function (args) {

                send("onEnter myarray()");

                send("mystr() env：" + args[0] + "  jobject：" + args[1] + " jobjectArray:" + args[2]);

                send("jobjectArray参数：" + args[2].toString());

                //可以在onEnter中通过this.xxx保存变量 在onLeave中通过this.xxx读取

                this.argptr = args[2]



                //jstring 不同于wchar_t* (jchar*) 与 char*

                //send("mystr："+Memory.readUtf16String(args[2],7));

                //send("mystr："+Memory.readUtf8String(args[2],7));

            },

            onLeave: function (retval) {

                send("onLeave myarray()");

                send("argptr:" + this.argptr);



                var env = Java.vm.getEnv();

                var cla = env.findClass("com/example/goal/DiyClass");

                send("clazz:" + cla);

                var initid = env.getMethodId(cla, "<init>", "(I)V");

                send("initid:" + initid);

                var setid = env.getMethodId(cla, "setData", "(I)V");

                send("setid:" + setid);

                var getid = env.getMethodId(cla, "getData", "()I");

                send("getid:" + getid);

                //frida 中env 方法参考frida-java/lib/env.js  本人能力有限，有些方法确实搞不懂

                //调用env中的allocObject()方法创建对象，未初始化，

                var obj1 = env.allocObject(cla);

                send("obj1:" + obj1);



                var obj2 = env.allocObject(cla);

                send("obj2:" + obj2);



                var rtarray = env.newObjectArray(2, cla, ptr(0));

                send("env.newObjectArray:" + rtarray);



                //获取DiyClass类中public void setData(int data)方法

                var nvmethod = env.nonvirtualVaMethod("void", ["int"]);

                //NativeType CallNonvirtual<type>Method(JNIEnv *env, jobject obj,jclass clazz, jmethodID methodID, ...);

                //设置obj1中data值

                nvmethod(env, obj1, cla, setid, 11);

                //设置obj2中data值

                nvmethod(env, obj2, cla, setid, 22);

                send("env.nonvirtualVaMethod(JNIEnv,jobject,jclass,jmethodid,args):" + nvmethod);

                //设置数组中的元素

                env.setObjectArrayElement(rtarray, 0, obj1);

                env.setObjectArrayElement(rtarray, 1, obj2);

                send("env.newObjectArray:" + rtarray);



                send("原retval:" + retval);

                retval.replace(ptr(rtarray));

                send("修改后retval:" + retval);



                // //堆中分配空间

                // var memo=Memory.alloc(4);

                // //写入数据

                // Memory.writeInt(memo,0x40302010);

                // // 读取数据

                // console.log(hexdump(memo, {

                //         offset: 0,

                //         length: 64,

                //         header: true,

                //         ansi: true

                // }));

            }

        });



    });

});
```
