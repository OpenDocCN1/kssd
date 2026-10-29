
**在ART里，除了`long`和`double`类型以8字节解析外，其余类型都以4字节解析**。

Java是没有指针概念的，所以必须将传入的指针转化为Java对象，而并没有直接将指针转化为Java的接口，**只能先将指针转化为JNI对象`jobject`，从JNI返回Java时会将`jobject`转化为Java对象**。

- **ArtMethod指针与JNI对象`jobject`转化接口，由JNI提供**

```
static jobject ToReflectedMethod(JNIEnv* env, jclass cls, jmethodID mid, jboolean isStatic)

static jmethodID FromReflectedMethod(JNIEnv* env, jobject method)
```

**1. `ToReflectedMethod`将ArtMethod指针转化为JNI对象`jobject`**

**2. `FromReflectedMethod`将JNI对象`jobject`转化为ArtMethod指针**

- **对象实际指针与JNI对象`jobject`转化接口，JNI不提供，需要解析相应符号**

```
jobject NewLocalRef(mirror::Object* obj)

ObjPtr<mirror::Object> Thread::DecodeJObject(jobject obj)
```

**1. `NewLocalRef`将对象实际指针转化为JNI对象`jobject`**，符号为`_ZN3art9JNIEnvExt11NewLocalRefEPNS_6mirror6ObjectE`

**2. `DecodeJObject`将JNI对象`jobject`转化为对象实际指针**，符号为`_ZNK3art6Thread13DecodeJObjectEP8_jobject`

按照上述规则，可以正确解析出方法参数

```
private static FastHookParam parseParam(long sp, Class[] paramType, boolean isStatic) {
        FastHookParam param = new FastHookParam();
    int offset = 0;
        List<Object> args = new ArrayList<Object>();
    if(!isStatic) {
            param.receiver = getObjectParam(sp,offset);
            offset += 4;
        }
    if(paramType == null)
            return param;
    for(Class type : paramType) {
        if(type.equals(boolean.class)) {
                boolean b = getBooleanParam(sp,offset);
                args.add(new Boolean(b));
                offset += 4;
            }else if(type.equals(byte.class)) {
                byte b2 = getByteParam(sp,offset);
                args.add(new Byte(b2));
                offset += 4;
            }else if(type.equals(char.class)) {
                char c = getCharParam(sp,offset);
                args.add(new Character(c));
                offset += 4;
            }else if(type.equals(short.class)) {
                short s = getShortParam(sp,offset);
                args.add(new Short(s));
                offset += 4;
            }else if(type.equals(int.class)) {
                int i = getIntParam(sp,offset);
                args.add(new Integer(i));
                offset += 4;
            }else if(type.equals(long.class)) {
                long l = getLongParam(sp,offset);
                args.add(new Long(l));
                offset += 8;
            }else if(type.equals(float.class)) {
                float f = getFloatParam(sp,offset);
                args.add(new Float(f));
                offset += 4;
            }else if(type.equals(double.class)) {
                double d = getDoubleParam(sp,offset);
                args.add(new Double(d));
                offset += 8;
            }else if(type.equals(void.class)) {
            }else {
                Object obj = getObjectParam(sp,offset);
                args.add(obj);
                offset += 4;
            }
        }
    if(!args.isEmpty()) {
            param.args = args.toArray(new Object[args.size()]);
        }
    return param;
}
```

```
jobject GetReflectedMethod(JNIEnv *env, jclass clazz, jlong art_method) {
    jobject result = (*env)->ToReflectedMethod(env,clazz,(void *)art_method,JNI_FALSE);
    return result;
}

jboolean GetBooleanParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jboolean result = (jboolean)ReadInt32((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jbyte GetByteParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jbyte result = (jbyte)ReadInt32((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jchar GetCharParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jchar result = (jchar)ReadInt32((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jshort GetShortParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jshort result = (jshort)ReadInt32((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jint GetIntParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jint result = (jint)ReadInt32((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jlong GetLongParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jlong result = (jlong)ReadInt64((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jfloat GetFloatParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jfloat result = (jfloat)ReadFloat((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jdouble GetDoubleParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    jdouble result = (jdouble)ReadDouble((unsigned char *)sp + pointer_size_ + offset);
    return result;
}

jobject GetObjectParam(JNIEnv *env, jclass clazz, jlong sp, jint offset) {
    void *obj = (void *)ReadInt32((unsigned char *)sp + pointer_size_ + offset);
    jobject result = new_local_ref_(env,obj);
    return result;
}
```

# 三、创建任意方法的代理方法

### 3.1 创建任意方法代理类

```
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler h)
```

`newProxyInstance`为Java动态代理接口，**可以明显看出Java对于动态代理的范围限制在了接口上，非接口方法不能代理**。

为了实现AOP，需要想办法绕过JAVA接口限制，实现创建任意方法的代理方法的功能。

```
private static native Class<?> generateProxy(String name, Class<?>[] interfaces,
                                                 ClassLoader loader, Method[] methods,
                                                 Class<?>[][] exceptions);
```

`generateProxy`进行实际代理类创建，可以接口方法最后是以`Method`对象的形式传入，做一个大胆的尝试，**将`Method[]`替换为我们想要的任意方法，这样变实现了创建任意方法的代理方法的功能**。如下所示：

```
private static native Class<?> generateProxy(name, null,loader, methods,null);
```

`interfaces`设置为`null`即可。

### 3.2 构造方法处理

构造方法的类型为`Constructor`，而不是`Method`，因此构造方法不能用`generateProxy`生成对应代理方法。需要想个办法将`Constructor`转为`Method`。

**在ART里，所有方法对对应一个`ArtMethod`对象，构造方法也不例外**。在前面的分析可知，**`ArtMethod`对象和Java对象是可以相互转化的，如果`ArtMethod`是构造方法则转化为`Constructor`对象，反之，则转化为`Method`对象**。

如果想将`Constructor`对象转化为`Method`对象，可以这么做

**1. 获取`Constructor`对象**

**2. 将其转化为`ArtMethod`对象**

**3. 将`ArtMethod`设置为非构造方法，通过清除`kAccConstructor`标志位实现**

**4. 将`ArtMethod`对象转化为`Method`对象**

**5. 还原`ArtMethod`为构造方法**

```
jobject ConstructorToMethod(JNIEnv *env, jclass clazz, jobject method) {
    jobject result = NULL;
    void *art_method = (void *)(*env)->FromReflectedMethod(env, method);
    ClearArtMethodAccessFlag(art_method,kAccConstructor);
    result = (*env)->ToReflectedMethod(env,clazz,(void *)art_method,JNI_FALSE);
    return result;
}

void MethodToConstructor(JNIEnv *env, jclass clazz, jobject method) {
    void *art_method = (void *)(*env)->FromReflectedMethod(env, method);
    AddArtMethodAccessFlag(art_method,kAccConstructor);
}
```

至此，可以得到任意方法的代理方法，即`Forward`方法，在需要的时候反射调用即可。

### 3.3 代理方法调用

假设有一个`Test`类

```
public class Test {
  public void test() {}
}
```

为其创建了代理类`ProxyTest`，如果需要调用原方法，只需调用`ProxyTest`的代理方法，并传入原方法参数。

```
Method test = ProxyTest.class.getMethod("test");
test.invoke(testObject);
```

**1. 通过反射获取代理方法`test`**

**2. 反射调用代理方法，传入原方法参数，`testObject`为`Test`实例，即`this`参数**

根据`FastHook`原理，这里调用必须是代理方法`test`，即`Forward`方法，不然将无法实现原方法的调用。**而代理方法`test`是`public`方法，反射调用的实际方法将由`testObject`来决定，`Test`也有一个`test`方法，因此实际调用的将是`Test`类的`test`方法而不是`ProxyTest`的`test`方法**。

因此无论在什么情况下，必须保证反射调用代理方法时，调用的都是其本身，**即代理方法必须是`Direct`方法**。

| 方法类型 | isDirect |
| --- | --- |
| static | true |
| private | true |
| constructor | true |

**当代理方法不是构造方法时，强制将其设置为`private`方法，以实现静态分派代理方法**。

```
void SetDirectMethod(JNIEnv *env, jclass clazz, jobject method) {
    void *art_method = (void *)(*env)->FromReflectedMethod(env, method);
    AddArtMethodAccessFlag(art_method,kAccPrivate);
}
```

代理方法`test`属于`ProxyTest`类，因此需要一个`ProxyTest`类型的实例，而现在传入的`testObject`是`Test`类型的，类型不匹配。**要想办法让`testObject`继承与`ProxyTest`类**。

`FastHook`采用一个巧妙的办法，**将`ProxyTest`的父类置空，让ART误认为这是`Object`类，众所周知，任何对象都继承与`Object`类，完美解决类型不匹配问题**。
