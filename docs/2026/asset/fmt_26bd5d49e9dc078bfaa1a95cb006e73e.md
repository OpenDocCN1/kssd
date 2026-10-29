
```javascript
function a1(){
    var pointerSize = Process.pointerSize
    var a = soAddr.add(0xCC6474).readPointer()
    for (var t = 0;t<1475;t++){
        var tt = a.add(t*pointerSize).readPointer()
        console.warn("---------")
        console.log("srcPointer \t--->\t"+a.add(t*pointerSize))
        console.log("Il2CppImage \t--->\t"+tt.add(pointerSize*0))
        console.log("name \t\t--->\t"+tt.add(pointerSize*2).readPointer().readCString())
        console.log("namespaze \t--->\t"+tt.add(pointerSize*3).readPointer().readCString())
    }
}
```

效果如下

![运行结果](https://bbs.kanxue.com/upload/attach/202012/868525_2QUSKYUW9EC4RSJ.jpg)

上述故意方法数多加一，看到最后空命名空间和`<Module>`基本就稳了，就是这意思没猜错

---

## 拓展

能拓展的东西就挺多的，这就涉及最初起点了

1. 让 [Il2CppDumperTool](https://bbs.kanxue.com/elink@cf7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2P5r3S2D9P5Y4W2Q4x3V1k6u0L8o6u0o6M7s2m8p5N6h3#2H3k6i4u0f1L8$3!0D9) 脱离 [Il2CppDumper](https://bbs.kanxue.com/elink@3d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7k6i4u0X3j5i4u0W2i4K6u0r3d9h3H3J5b7%4m8H3c8s2g2E0M7r3g2J5) 以及python脚本更易用

   ![](https://bbs.kanxue.com/upload/attach/202012/868525_KNMJ3G8XMGSSU9P.jpg)

2. 搞一些通用的API Hook，比如

   - `UnityEngine.GameObject.SetActive(Boolean)`
   - `UnityEngine.Object.GetName(UnityEngine:Object):String`
   - `UnityEngine.Application.get_identifier():String`
   - `UnityEngine.PlayerPrefs.GetInt(String,Int32):Int32`

......

---

以上展示只是一个简单的分析，其实就上述这样东西还是非常不好用，上述的`addr`并不能找到所有的方法及其地址，后续就想着让他更加智能一点，就尝试手动去解析一下结构体，然后就得到了一下效果

![找关键位置](https://bbs.kanxue.com/upload/attach/202012/868525_HS8JQH5U8RXQ3BW.png)

首先是针对我们需要的三个参数的自动解析，主要涉及到`ldr`寻址

```javascript
function FuckLDR(addr,isRealAddr){
    var ins = Instruction.parse(addr)
    if (ins.mnemonic != "ldr"){
        LOG("Not LDR Instruction")
        return ptr(0)
    }
    var off = ins.operands[1].value.disp
    // console.log(JSON.stringify(ins.operands))
    var pc_1 = addr.add(p_size*2)
    var pc_2 = addr.add(p_size*3)
    var codeEnd = pc_1.add(off).readPointer()
    var final = codeEnd.add(pc_2)
    return isRealAddr?final:final.sub(soAddr)
}
```

![findins](https://bbs.kanxue.com/upload/attach/202012/868525_WRWW65ZBYYBCP49.png)

这里为了实现找到我们需要的参数，主要是通过导出函数`il2cpp_init`的地址往下找找到`bl`进去然后再找到`ldr`，并解析到`ldr`加载的值，如果出现了`mscorlib.dll`，说明我们找到的`bl`是正确的，然后我们同理找下层的`global-metadata.dat`即可定位到函数`bool il2cpp::vm::MetadataCache::Initialize()`，找到上述位置后，我们稍微看一下就可以发现特征`IL2CPP_CALLOC`函数，老规矩通过这个`bl`指令去解析到函数地址，记录所有调用的该函数的当前地址放进数组，接下来就是通过数组访问到`[1]` `[4]` `[5]`位置就是我们需要的三个初始参数，从当前位置往上解析它，第一条`mov r0,#xx`的立即数就是我们要找的，好了介绍到这里，上效果：

![找参数](https://bbs.kanxue.com/upload/attach/202012/868525_Y6HNH72EXRQ8TCN.png)

这种找法效果还行，能解决大部分问题，也不能解决全部问题，总有些奇葩`adr`之类的东西，这些就自己手动 IDA 查看了

然后就是挨着列出

![列出namespace和class](https://bbs.kanxue.com/upload/attach/202012/868525_MKRGUTFJP5NEWXG.png)

![列出method](https://bbs.kanxue.com/upload/attach/202012/868525_UBMGYP93J7FCKFT.png)

也可以解析的更加详细（其实还可以继续深入解析出方法类型返回值类型的，暂时没有继续深入）

![详细method](https://bbs.kanxue.com/upload/attach/202012/868525_SEYQN9KDDHM6X3P.png)

用新的方法来解析找函数就不存在解析不到函数的问题了（随意解析这个`so`里面的任意函数）

![解析method](https://bbs.kanxue.com/upload/attach/202012/868525_3GRH6BX3AYESBPE.png)

---

## 实际用处

随便举个例：断点 `public extern void SetActive(GameObject obj,bool value);`

1. 有了 `gameObject` 就可以拿到 `transform`，以及他的名字，再遍历拿到父级名字，整个调用链你都清晰了

```javascript
function HookSetActive(){
    var f_getName = new NativeFunction(find_method("UnityEngine.CoreModule","Object","GetName",1,true),'pointer',['pointer'])
    var f_getLayer = new NativeFunction(find_method("UnityEngine.CoreModule","GameObject","get_layer",0,true),'int',['pointer'])
    var f_getTransform = new NativeFunction(find_method("UnityEngine.CoreModule","GameObject","get_transform",0,true),'pointer',['pointer'])
    var f_getParent = new NativeFunction(find_method("UnityEngine.CoreModule","Transform","GetParent",0,true),'pointer',['pointer'])

    Interceptor.attach(find_method("UnityEngine.CoreModule","GameObject","SetActive",1,true),{
        onEnter:function(args){
            if (args[1].toInt32() == 1 || args[1].toInt32() == 0) {
                LOG("\n--------------------------------------",LogColor.YELLOW)
                LOG("public extern void SetActive( "+(args[1].toInt32()==0?"false":"true")+" );",LogColor.COLOR_36)
                LOG("gameObj\t\t--->\t"+args[0],LogColor.COLOR_36)
                LOG("getName\t\t--->\t"+f_getName(args[0]).add(Process.pointerSize*3).readUtf16String(),LogColor.COLOR_36)
                LOG("getLayer\t--->\t"+f_getLayer(args[0]),LogColor.COLOR_36)
                var m_transform = f_getTransform(args[0])
                LOG("getTransform\t--->\t"+m_transform,LogColor.COLOR_36)
                var layerNames = ""
                try{
                    for (var i=0;i<10;i++){
                        var getName = f_getName(m_transform)
                        var spl =  layerNames == "" ? "" : " <--- "
                        layerNames = layerNames + spl + getName.add(Process.pointerSize*3).readUtf16String()
                        m_transform = f_getParent(m_transform)
                        if (m_transform == 0) break
                    }
                }catch(e){
                    LOG(e,LogColor.RED)
                }
                LOG("hierarchy\t--->\t"+layerNames,LogColor.COLOR_36)
            }
        },
        onLeave:function(retval){

        }
    })
}
```

![](https://bbs.kanxue.com/upload/attach/202012/868525_9RBW8G6QKAD4QSS.png)

2. 有了 `gameObject` 用以上的函数API可以随意的移动，显示隐藏 。。。。。。

......

---

## 补充：

`a9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2P5r3S2D9P5Y4W2Q4x3V1k6u0L8o6u0o6M7s2m8t1L8$3!0C8f1$3y4J5K9i4m8@1M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6u0L8o6u0U0M7s2m8t1L8$3!0C8`

---

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-5-31 17:22 被唱过阡陌编辑，原因：github地址修改

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)
