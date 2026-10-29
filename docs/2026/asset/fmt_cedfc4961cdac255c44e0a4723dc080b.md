
```
var func_addr = [...]
var func_name = [...]

function hook_dlopen(soName = '') {
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
    {
        onEnter: function (args) {
            var pathptr = args[0];
            if (pathptr !== undefined && pathptr != null) {
                var path = ptr(pathptr).readCString();
                if (path.indexOf(soName) >= 0) {
                    this.is_can_hook = true;
                }
            }
        },
        onLeave: function (retval) {
            if (this.is_can_hook) {
                //hook_sub_3530C();
                var times = 1;
                var module = Process.getModuleByName("libsec2023.so");
                this.pid = Process.getCurrentThreadId();
                console.log("start Stalker!");
                Stalker.follow(this.pid,{
                    events:{
                        call:false,
                        ret:false,
                        exec:false,
                        block:false,
                        compile:false
                    },
                    onReceive:function(events){
                    },
                    transform: function (iterator) {
                        var instruction = iterator.next();
                        do{
                            if (func_addr.indexOf(instruction.address - module.base) != -1){
                                console.log("call" + times+ ":" + func_name[func_addr.indexOf(instruction.address - module.base)])
                                times=times+1
                            }
                            iterator.keep();
                        } while ((instruction = iterator.next()) !== null);
                    },
                    onCallSummary:function(summary){
                    }
                });
                console.log("Stalker end!");
            }
        }
    }
    );
}
setImmediate(hook_dlopen, "libsec2023.so")
```

打开apk后`frida`输出如下

```
call1:JNI_OnLoad
call2:sub_FF14
call3:.memset
call4:.vsnprintf
call5:.time
call6:.localtime
call7:.__android_log_print
call8:sub_10070
call9:.fopen
call10:sub_21000
call11:.pthread_once
call12:sub_21054
call13:sub_412CC
call14:.malloc
call15:sub_21098
call16:sub_FABC
call17:sub_1010C
call18:sub_10194
call19:sub_11C4C
call20:.pthread_mutex_init
call21:sub_103D0
call22:sub_2BCA8
call23:sub_125E4
call24:sub_12660
call25:sub_1CE70
call26:sub_1CE34
call27:sub_1C664
call28:.pthread_mutex_lock
call29:.pthread_mutex_unlock
call30:.strlen
call31:sub_125F0
call32:sub_1D998
call33:sub_2C67C
call34:sub_2C40C
call35:.__strlcpy_chk
call36:.__strlen_chk
call37:sub_11BC4
call38:sub_2CCC0
call39:sub_355F0
call40:sub_35630
call41:sub_356C4
call42:sub_35700
call43:sub_11DF0
call44:sub_35870
call45:sub_36940
call46:sub_36B34
call47:sub_36B9C
call48:sub_36BC8
call49:sub_36BF0
call50:sub_36E00
call51:sub_36E70
call52:sub_36ED0
call53:sub_36F00
call54:sub_36F3C
call55:nullsub_17
call56:sub_36C8C
call57:sub_36CB8
call58:sub_2E318
call59:sub_2E288
call60:sub_2D590
call61:sub_1F450
call62:sub_20FD0
call63:.fstat
call64:sub_2DB5C
call65:sub_1FE3C
call66:sub_1FB70
call67:.sscanf
call68:sub_200C0
call69:.memcpy
call70:sub_1F6C0
call71:sub_1F8A4
call72:.free
call73:sub_36D38
call74:sub_36D70
call75:sub_36DA4
call76:sub_37060
call77:sub_41368
call78:sub_36A20
call79:sub_36D10
call80:sub_3C6A4
call81:sub_369B0
call82:sub_3DF74
call83:sub_3A054
call84:sub_3A090
call85:sub_3A0FC
call86:sub_3A138
call87:sub_24364
call88:sub_3852C
call89:sub_38D9C
call90:sub_36A90
call91:sub_3F2B0
call92:sub_36120
call93:sub_36144
call94:sub_370AC
call95:sub_36558
call96:sub_21BAC
call97:sub_21C20
call98:sub_21F50
call99:sub_21DB4
call100:j_.pthread_mutex_lock
call101:j_.pthread_mutex_unlock
call102:sub_11E30
call103:sub_11C60
call104:.pthread_attr_init
call105:.pthread_attr_setstacksize
call106:.pthread_attr_setdetachstate
call107:.pthread_create
call108:.pthread_attr_destroy
call109:sub_11C6C
call110:j_j_.free_2
call111:j_.free
call112:sub_37254
call113:sub_37740
call114:sub_377A8
call115:sub_377D8
call116:sub_37804
call117:sub_37A64
call118:sub_37AD4
call119:sub_37B34
call120:sub_37B64
call121:sub_37BA0
call122:nullsub_18
call123:sub_3789C
call124:sub_378C4
call125:sub_20DD0
call126:sub_20580
call127:sub_20CB0
call128:sub_20EBC
call129:j_.stat
call130:.stat
call131:sub_373B4
call132:sub_1240C
call133:sub_1F74C
call134:.lseek
call135:sub_1F9EC
call136:sub_1FA34
call137:sub_37940
call138:sub_37974
call139:sub_379A8
call140:sub_1235C
call141:sub_37134
call142:sub_37184
call143:sub_371AC
call144:sub_376CC
call145:sub_37704
call146:sub_3715C
call147:sub_36580
call148:sub_36538
call149:sub_36580
call150:sub_36178
...
```

# 对`libsec2023.so`打下硬件断点

这里需要用到的工具是`rwprocmem33`,具体的编译和使用可以在我写的[另一篇文章](https://bbs.kanxue.com/thread-278647.htm)进行阅读,这里不在过多赘述

运行下面的`frida`代码获取`libsec2023.so`的基址

```
function dump_so(so_name) {
    Java.perform(function () {
        var currentApplication = Java.use("android.app.ActivityThread").currentApplication();
        var dir = currentApplication.getApplicationContext().getFilesDir().getPath();
        var libso = Process.getModuleByName(so_name);
        console.log("[name]:", libso.name);
        console.log("[base]:", libso.base);
        console.log("[size]:", ptr(libso.size));
        console.log("[path]:", libso.path);
    });
}

rpc.exports = {
    dump_so: dump_so
};
```

```
frida -H 127.0.0.1:1234 -l "D:\frida\sec2023\get_so_base.js" -f "com.com.sec2023.rocketmouse.mouse"
```

![image-20230817132059046](https://bbs.kanxue.com/upload/attach/202308/963320_UH289TAP7SHP38T.png)

利用下面的命令获取进程的PID

```
ps -A | grep mouse
```

将得到的参数填入`rwprocmem33`中

首先对我们最感兴趣的`libil2cpp.so`调用的`libsec2023.so`的导出函数`sub_35404`下硬件断点看看,毕竟之前我们用`frida`去hook这个函数失败了嘛,硬件断点下的位置可以通过基址(`base`) + 偏移(`func offset`)得到

结果如下

![image-20230817132705290](https://bbs.kanxue.com/upload/attach/202308/963320_DBF6EWMQ3VAWDE5.png)

仅仅下了五秒的硬件断点,相同地址的命中次数进入达到了百万次

我们再对不同的地址打下硬件断点看看,发现均只有一个命中地址,并且命中次数都达到百万次

与基址相减得到偏移为`0x37704`

![image-20230817133029502](https://bbs.kanxue.com/upload/attach/202308/963320_C7AT8DAUBBYX5A2.png)

进入`ida`查看`sub_37704`函数,代码很短,按下交叉引用也没有输出

![image-20230817025119017](https://bbs.kanxue.com/upload/attach/202308/963320_N5YRVTKHGQ3MWSR.png)

这该怎么办呢?

还记得上面我们曾用`frida-stalker`打印出了`libsec2023.so`函数的调用链嘛,我们从`sub_37704`向上回溯看看

```
call135:sub_1F9EC
call136:sub_1FA34
call137:sub_37940
call138:sub_37974
call139:sub_379A8
call140:sub_1235C
call141:sub_37134
call142:sub_37184
call143:sub_371AC
call144:sub_376CC
call145:sub_37704
```

`sub_37704`是被`sub_376CC`调用的,`sub_376CC`中的这个`BR`跳转应该是调用了`sub_37704`

![image-20230817030756246](https://bbs.kanxue.com/upload/attach/202308/963320_86D5C8EEEAJWFTH.png)

再看调用`sub_376CC`的函数`sub_371AC`,在这个函数中,我们发现了一条有趣的指令,`CSEL`

![image-20230817133504656](https://bbs.kanxue.com/upload/attach/202308/963320_MAMHF68BBJV9QD9.png)

熟悉`arm`指令集的朋友肯定知道,`CSEL`是`arm`中的分支结构指令,而`BR`跳转的位置由`X8`决定,所以这段汇编便可以改变便程序控制流
