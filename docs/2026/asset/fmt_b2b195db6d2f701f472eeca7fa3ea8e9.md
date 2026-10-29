
# cocos2d逆向入门和某捕鱼游戏分析

[TOC]

# 初步认识cocos2d-x

先clone到本地

`git clone <mark class="encrypted">b30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3y4G2M7K6u0V1i4K6u0r3j5$3!0U0L8%4x3J5k6q4)9J5k6s2S2Q4x3X3g2Y4K9i4b7`.</mark>

`Cocos2d-x`是一个开源的移动2D游戏框架，底层支持各种平台，核心用c++封装了各种库，外面给了lua和c++的接口，所以关键代码可能在lua中，很多安卓游戏的逻辑也基本都在lua脚本里，盗用官网这张图

![](upload/attach/202107/877073\_8S5R433RFN2RNT7.png)

# 从c++进入lua世界

lua虚拟机相关代码在`cocos2d-x\cocos\scripting\lua-bindings\manual`里

`CCLuaEngine.h` lua引擎相关

`CCLuaStack.h` lua栈相关

进入虚拟机

`cocos2d-x\templates\lua-template-default\frameworks\runtime-src\Classes\AppDelegate.cpp\AppDelegate::applicationDidFinishLaunching`

## applicationDidFinishLaunching函数

应用结束加载中进入lua虚拟机

```
bool AppDelegate::applicationDidFinishLaunching()
// set default FPS
Director::getInstance()->setAnimationInterval(1.0 / 60.0f);//设置fps刷新率
// register lua module
auto engine = LuaEngine::getInstance();//创建lua虚拟机引擎
ScriptEngineManager::getInstance()->setScriptEngine(engine);//设置脚本引擎为lua引擎
lua_State* L = engine->getLuaStack()->getLuaState();//创建lua虚拟机环境lua_State
lua_module_register(L); //分配网络，控制台，ui界面等相关联的寄存器
register_all_packages();
LuaStack* stack = engine->getLuaStack();
stack->setXXTEAKeyAndSign("2dxLua", strlen("2dxLua"), "XXTEA", strlen("XXTEA"));
//这一步很关键，获取栈结构并调用setXXTEAKeyAndSign设置加密算法为xxtea，sign为XXTEA,KEY为2dxlua，很多游戏lua脚本都用的默认的sign和key
//如果没有用默认的，ida打开libxxxluaxxx.so直接搜索applicationDidFinishLaunching导出函数也基本都能直接找到
//register custom function
//LuaStack* stack = engine->getLuaStack();
//register_custom_function(stack->getLuaState());
#if CC_64BITS
FileUtils::getInstance()->addSearchPath("src/64bit");
#endif
FileUtils::getInstance()->addSearchPath("src"); //lua源码在src文件夹，资源在res文件夹
FileUtils::getInstance()->addSearchPath("res");
if (engine->executeScriptFile("main.lua")) //直接通过lua引擎调用main.lua进入lua的世界
{
return false;
}
return true;
}
```

这句`engine->executeScriptFile("main.lua")`调用了`cocos2d-x\cocos\scripting\lua-bindings\manual\CCLuaEngine.cpp`的`executeScriptFile`调用了`CCLuaStack.cpp`的`executeScriptFile`

## LuaStack::executeScriptFile

```
int LuaStack::executeScriptFile(const char* filename)
{
CCAssert(filename, "CCLuaStack::executeScriptFile() - invalid filename");
std::string buf(filename);
//
// remove .lua or .luac
//
size_t pos = buf.rfind(BYTECODE_FILE_EXT);//BYTECODE_FILE_EXT就是lua字节码，NOT_BYTECODE_FILE_EXT就是lua脚本源码
//static const std::string BYTECODE_FILE_EXT = ".luac";
if (pos != std::string::npos)
{
buf = buf.substr(0, pos);//截取前缀
}
else
{
pos = buf.rfind(NOT_BYTECODE_FILE_EXT);
if (pos == buf.length() - NOT_BYTECODE_FILE_EXT.length())
{
buf = buf.substr(0, pos);
}
}
FileUtils *utils = FileUtils::getInstance();
//
// 1. check .luac suffix
// 2. check .lua suffix
//
std::string tmpfilename = buf + BYTECODE_FILE_EXT;
if (utils->isFileExist(tmpfilename))
{
buf = tmpfilename;
}
else
{
tmpfilename = buf + NOT_BYTECODE_FILE_EXT;
if (utils->isFileExist(tmpfilename))
{
buf = tmpfilename;
}
}
std::string fullPath = utils->fullPathForFilename(buf);//获取绝对路径
Data data = utils->getDataFromFile(fullPath);//通过getDataFromFile读取lua文件到data
int rn = 0;
if (!data.isNull())
{
if (luaLoadBuffer(_state, (const char*)data.getBytes(), (int)data.getSize(), fullPath.c_str()) == 0)//通过luaLoadBuffer加载data
{
rn = executeFunction(0);
}
}
return rn;
}
```

## LuaStack::luaLoadBuffer

`luaLoadBuffer`里调用`xxtea_decrypt`解密了lua脚本，然后调用`luaL_loadbuffer`加载解密后的脚本，所以直接hook 这个函数`luaL_loadbuffer`把`(char*)content`这个字符dump出来就得到解密过的lua脚本了

```
int LuaStack::luaLoadBuffer(lua_State *L, const char *chunk, int chunkSize, const char *chunkName)
{
int r = 0;
if (_xxteaEnabled && strncmp(chunk, _xxteaSign, _xxteaSignLen) == 0)//这里判断是否开启xxtea加密，如果开启就需要解密
{
// decrypt XXTEA
xxtea_long len = 0;
unsigned char* result = xxtea_decrypt((unsigned char*)chunk + _xxteaSignLen,
(xxtea_long)chunkSize - _xxteaSignLen,
(unsigned char*)_xxteaKey,
(xxtea_long)_xxteaKeyLen,
&len);//调用xxtea_decrypt解密脚本，这个函数在cocos2d-x\external\xxtea\xxtea.cpp里，加解密都在这个cpp里
unsigned char* content = result;
xxtea_long contentSize = len;
skipBOM((const char*&)content, (int&)contentSize);//忽略utf8的bom
r = luaL_loadbuffer(L, (char*)content, contentSize, chunkName);//无论是否加密，解密后都会调用luaL_loadbuffer函数，所以直接hook这个函数把(char*)content这个字符dump出来就是解密过的lua脚本了
free(result);
}
else
{
skipBOM(chunk, chunkSize);
r = luaL_loadbuffer(L, chunk, chunkSize, chunkName);//这个返回值r会反映加载失败的类型，在下面的switch中打印出来
}
#if defined(COCOS2D_DEBUG) && COCOS2D_DEBUG > 0
if (r)
{
switch (r)
{
case LUA_ERRSYNTAX:
CCLOG("[LUA ERROR] load \"%s\", error: syntax error during pre-compilation.", chunkName);
break;
case LUA_ERRMEM:
CCLOG("[LUA ERROR] load \"%s\", error: memory allocation error.", chunkName);
break;
case LUA_ERRFILE:
CCLOG("[LUA ERROR] load \"%s\", error: cannot open/read file.", chunkName);
break;
default:
CCLOG("[LUA ERROR] load \"%s\", error: unknown.", chunkName);
}
}
#endif
return r;
}
```

而`luaL_loadbuffer`的源码没有，只有编译过的库`cocos2d-x\external\lua\luajit\prebuilt\android\armeabi-v7a\libluajit.a`，

要找它的实现需要下载luajit源代码分析了，这就完全进入了lua虚拟机的实现

a **Just-In-Time Compiler** for Lua. **采用C语言写的Lua的解释器的代码**

# 总结

1.从c++进入lua世界的调用逻辑

```
AppDelegate::applicationDidFinishLaunching
{
setXXTEAKeyAndSign
executeScriptFile
{
 getDataFromFile
 luaLoadBuffer
 {
 xxtea_decrypt
 luaL_loadbuffer
 {
 luajit
 }
 }
 executeFunction
}
}
```

2.加密算法为xxtea，如果没有修改，sign为XXTEA,KEY为2dxlua，如果有修改，可以ida打开libxxluaxx.so在applicationDidFinishLaunching里找到

3.无论是否加密，解密后都会调用`luaL_loadbuffer`函数，所以直接hook这个函数把`(char*)content`这个字符dump出来就是解密过的lua脚本了，缺点是要把游戏运行一遍，只能搞出执行过的代码

4.`cocos2d-x\external\xxtea\xxtea.cpp`里有完整的加密解密算法，逻辑清晰，可以写个python脚本直接本地解密，也可以在这里hook获取key和sign或者解密后脚本

# 实战

某捕鱼游戏，下载安装apk后，再其内部内置了捕鱼、麻将等十几款小游戏，嘿嘿，懂得都懂这是干啥的，直接点击捕鱼游戏下载，

下载后的游戏源码在`/data/data/com.q8wdw6.gyll9spfo.nycatp9/files/download/`里

`adb pull`出来，files文件夹里面已经暴露很多信息了，配置，下载的游戏，更新等等

![](upload/attach/202107/877073\_38GTBGTYV7U4YPN.png)

进入`files\download\107\res\`就可以看到luac，很明显crash里面存储了所有碰撞，`model_path_crash`和`path`里存储的是移动路径，我们可以看到所有的鱼的移动路径都是tm设定好的，models里面存储着游戏逻辑，views里面存储着界面显示逻辑

![](upload/attach/202107/877073\_JXFQE8PMCCXRCEE.png)

随机打开一个luac加密了，开头的`ZX_CS@56#D~d@dud`明显就是加密sign

![](upload/attach/202107/877073\_3R4FFSM95AXHN7B.png)

根据之前对cocos2d引擎的分析，找到apk下面带有lua也是最大的一个so `libqpry_lua.so`，用ida直接打开，定位到`AppDelegate::applicationDidFinishLaunching`函数，直接拖到最下面，看到这句

`(**(v7 + 1) + 116))(*(v7 + 1), "ZX_01RdsF~@!R8", 14, "ZX_CS@56#D~d@dud", 16);`很明显是调用了`stack->setXXTEAKeyAndSign`

对比源码`stack->setXXTEAKeyAndSign("2dxLua", strlen("2dxLua"), "XXTEA", strlen("XXTEA"));`

解密key为`ZX_01RdsF~@!R8`，sign为`ZX_CS@56#D~d@dud`

也可以直接搜索字符串，因为key和sign在一个函数调用，所以一般离的很近，`base/src/main.lua`这个字符串进一步验证了正确性

![](upload/attach/202107/877073\_2CNR8BXEVKJ5A37.png)

```
int __fastcall AppDelegate::applicationDidFinishLaunching(AppDelegate *this)
//前面是一堆初始化啥的没用的
....
(*(v7 + 1), "ZX_01RdsF~@!R8", 14, "ZX_CS@56#D~d@dud", 16);
if ( (*(*v7 + 28))(v7, "base/src/main.lua") )
return 0;
v26 = GetMCKernel();
if ( !v26 )
return 0;
v27 = *(this + 145);
if ( v27 )
v27 += 580;
v28 = (*(*v26 + 20))(v26, v27);
v29 = *(cocos2d::Director::getInstance(v28) + 152);
v32[0] = AppDelegate::GlobalUpdate;
v32[1] = -8;
cocos2d::Scheduler::schedule(v29, AppDelegate::GlobalUpdate, -8, this + 4, 0.0, 0xFFFFFFFE, 0.0, 0);
return v25;
}
```

有了key和sign就可以直接解密luac脚本了，照着写，很简单，可以看到sign的作用就是被忽略，只有长度有用，尴尬，解密用的主要是key

```
unsigned char* result = xxtea_decrypt((unsigned char*)chunk + _xxteaSignLen,
(xxtea_long)chunkSize - _xxteaSignLen,
(unsigned char*)_xxteaKey,
(xxtea_long)_xxteaKeyLen,
&len);
unsigned char *xxtea_decrypt(unsigned char *data, xxtea_long data_len, unsigned char *key, xxtea_long key_len, xxtea_long *ret_length)
{
unsigned char *result;
*ret_length = 0;
if (key_len < 16) {
unsigned char *key2 = fix_key_length(key, key_len);
result = do_xxtea_decrypt(data, data_len, key2, ret_length);
```
