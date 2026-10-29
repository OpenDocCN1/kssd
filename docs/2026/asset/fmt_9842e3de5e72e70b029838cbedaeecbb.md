
`public static void loadClassAndInvoke(ClassLoader appClassloader, String eachclassname, Method dumpMethodCode_method) {`
`Class resultclass` `=` `null;`
`Log.i(`"ActivityThread", `"go into loadClassAndInvoke->"` `+` `"classname:"` `+` `eachclassname);`
`try` `{`
`// 由于我们反射取到了类名，所以这里直接用我们前面看到的loadClass就可以取到对应的classloader了`
`resultclass` `=` `appClassloader.loadClass(eachclassname);`
`} catch (Exception e) {`
`e.printStackTrace();`
`return;`
`} catch (Error e) {`
`e.printStackTrace();`
`return;`
`}`
`// 反射遍历所有构造函数，并且调用前面准备好了传过来的保存构造函数代码的方法，直接调用就可以dump每个构造函数的代码了`
`if` `(resultclass !=` `null) {`
`try` `{`
`Constructor<?> cons[]` `=` `resultclass.getDeclaredConstructors();`
`for` `(Constructor<?> constructor : cons) {`
`if` `(dumpMethodCode_method !=` `null) {`
`try` `{`
`dumpMethodCode_method.invoke(null, constructor);`
`} catch (Exception e) {`
`e.printStackTrace();`
`continue;`
`} catch (Error e) {`
`e.printStackTrace();`
`continue;`
`}`
`}` `else` `{`
`Log.e(`"ActivityThread", `"dumpMethodCode_method is null ");`
`}`
`}`
`} catch (Exception e) {`
`e.printStackTrace();`
`} catch (Error e) {`
`e.printStackTrace();`
`}`
`// 这里反射获取所有的函数，然后用前面准备的dump方法把所有函数都dump下来`
`try` `{`
`Method[] methods` `=` `resultclass.getDeclaredMethods();`
`if` `(methods !=` `null) {`
`for` `(Method m : methods) {`
`if` `(dumpMethodCode_method !=` `null) {`
`try` `{`
`dumpMethodCode_method.invoke(null, m);`
`} catch (Exception e) {`
`e.printStackTrace();`
`continue;`
`} catch (Error e) {`
`e.printStackTrace();`
`continue;`
`}`
`}` `else` `{`
`Log.e(`"ActivityThread", `"dumpMethodCode_method is null ");`
`}`
`}`
`}`
`} catch (Exception e) {`
`e.printStackTrace();`
`} catch (Error e) {`
`e.printStackTrace();`
`}`
`}`
`}`

接下来就看看我们提前准备的`dumpMethod`是如何dump函数的了。找到`DexFile`里面的`native`实现的代码

```
private static native void dumpMethodCode(Object m);
static void DexFile_dumpMethodCode(JNIEnv* env, jclass,jobject method) {
if(method!=nullptr)
{
ArtMethod* proxy_method = jobject2ArtMethod(env, method);
myfartInvoke(proxy_method);
}
return;
}
```

这里先是把`java`传递过来的函数指针转换成了`ArtMethod`了。然后`myfartInvoke`来处理每个函数。继续看看里面怎么做的

```
extern "C" void myfartInvoke(ArtMethod* artmethod) REQUIRES_SHARED(Locks::mutator_lock_) {
JValue *result=nullptr;
Thread *self=nullptr;
uint32_t temp=6;
uint32_t* args=&temp;
uint32_t args_size=6;
artmethod->Invoke(self, args, args_size, result,"fart");
}
```

看到是调用了`artmethod`的`Invoke`。然后最后一个参数来告知里面，这是一个`fart`主动调用的函数。继续看看里面处理。代码有点多，下，面代码只放上和`fart`相关的处理

```
void ArtMethod::Invoke(Thread* self, uint32_t* args, uint32_t args_size, JValue* result,
const char* shorty) {
if (self==nullptr) {
dumpArtMethod(this);
return;
}
}
```

这里看到。其实他虽然传了个`fart`字符串进来想作为判断的。但是实际上没有使用。最后还是根据第一个参数`self`是否为`nullptr`来判断用不用`dump`这个函数的。继续看`dumpArtMethod`的处理
