
```java
private void testDexClassLoader(Context context, String dexfilepath) {
//构建文件路径：/data/data/com.emaxple.test02/app\_opt\_dex，存放优化后的dex,lib库
File optfile = context.getDir("opt\_dex",0);
File libfile = context.getDir("lib\_dex",0);
ClassLoader parentclassloader = MainActivity.class.getClassLoader();
ClassLoader tmpclassloader = context.getClassLoader();
//可以为DexClassLoader指定父类加载器
DexClassLoader dexClassLoader = new DexClassLoader(dexfilepath,optfile.getAbsolutePath(),libfile.getAbsolutePath(),parentclassloader);
Class clazz = null;
try {
clazz = dexClassLoader.loadClass("com.example.test.TestClass");
} catch (ClassNotFoundException e) {
e.printStackTrace();
}
if(clazz!=null){
try {
Method testFuncMethod = clazz.getDeclaredMethod("test02");
Object obj = clazz.newInstance();
testFuncMethod.invoke(obj);
} catch (NoSuchMethodException e) {
e.printStackTrace();
} catch (IllegalAccessException e) {
e.printStackTrace();
} catch (InstantiationException e) {
e.printStackTrace();
} catch (InvocationTargetException e) {
e.printStackTrace();
}
}
}
```

#### （5）获得类列表

我们通过`getClassNameList`来获取类列表

```
private static native String[] getClassNameList(Object cookie);
```

```java
public static void getClassListInClassLoader(ClassLoader classLoader){
//先拿到BaseDexClassLoader
try {
//拿到pathList
Class BaseDexClassLoader = Class.forName("dalvik.system.BaseDexClassLoader");
Field pathListField = BaseDexClassLoader.getDeclaredField("pathList");
pathListField.setAccessible(true);
Object pathListObj = pathListField.get(classLoader);
//拿到dexElements
Class DexElementClass = Class.forName("dalvik.system.DexPathList");
Field DexElementFiled = DexElementClass.getDeclaredField("dexElements");
DexElementFiled.setAccessible(true);
Object[] dexElementObj = (Object[]) DexElementFiled.get(pathListObj);
//拿到dexFile
Class Element = Class.forName("dalvik.system.DexPathList$Element");
Field dexFileField = Element.getDeclaredField("dexFile");
dexFileField.setAccessible(true);
Class DexFile =Class.forName("dalvik.system.DexFile");
Field mCookieField = DexFile.getDeclaredField("mCookie");
mCookieField.setAccessible(true);
Field mFiledNameField = DexFile.getDeclaredField("mFileName");
mFiledNameField.setAccessible(true);
//拿到getClassNameList
Method getClassNameListMethod = DexFile.getDeclaredMethod("getClassNameList",Object.class);
getClassNameListMethod.setAccessible(true);
for(Object dexElement:dexElementObj){
Object dexfileObj = dexFileField.get(dexElement);
Object mCookiedobj = mCookieField.get(dexfileObj);
String mFileNameobj = (String) mFiledNameField.get(dexfileObj);
String[] classlist = (String[]) getClassNameListMethod.invoke(null,mCookiedobj);
for(String classname:classlist){
Log.e("classlist",classLoader.toString()+"-----"+mFileNameobj+"-----"+classname);
}
}
} catch (ClassNotFoundException e) {
e.printStackTrace();
} catch (NoSuchFieldException e) {
e.printStackTrace();
} catch (IllegalAccessException e) {
e.printStackTrace();
} catch (NoSuchMethodException e) {
e.printStackTrace();
} catch (InvocationTargetException e) {
e.printStackTrace();
}
}
```

## 三、实验总结

花了一段时间，断断续续总算把这篇类加载器和动态加载的帖子写完了，从中学习到了很多，这里如果有什么错误，就请各位大佬指正了。

## 四、参考文献

```
http://gityuan.com/2017/03/19/android-classloader/
https://www.jianshu.com/p/7193600024e7
https://www.jianshu.com/p/ff489696ada2
https://www.jianshu.com/p/363a4ad0489d
https://github.com/huanzhiyazi/articles/issues/30
https://juejin.cn/post/6844903940094427150#heading-12
```

# Android加壳与脱壳（1）——深入理解类加载器和动态加载

## 目录

- [Android加壳与脱壳（1）——深入理解类加载器和动态加载](https://bbs.kanxue.com/#android加壳与脱壳（1）——深入理解类加载器和动态加载)
- [一、前言](https://bbs.kanxue.com/#一、前言)
- [二、类加载器](https://bbs.kanxue.com/#二、类加载器)
- [1.双亲委派模式](https://bbs.kanxue.com/#1.双亲委派模式)
- [（1）双亲委派模式定义](https://bbs.kanxue.com/#（1）双亲委派模式定义)
- [（2）双亲委派模式加载流程](https://bbs.kanxue.com/#（2）双亲委派模式加载流程)
- [（3）双亲委派的作用](https://bbs.kanxue.com/#（3）双亲委派的作用)
- 2. Android中类加载机制
- [（1）Android基本类预加载](https://bbs.kanxue.com/#（1）android基本类预加载)
- [（2）Android类加载器层级关系及分析](https://bbs.kanxue.com/#（2）android类加载器层级关系及分析)
- <1> BootClassLoader
- <2> PathClassLoader
- <3> DexClassLoader
- <4> BaseDexClassLoader
- [3.案例](https://bbs.kanxue.com/#3.案例)
- [（1）验证类加载器](https://bbs.kanxue.com/#（1）验证类加载器)
- [（2）遍历父类加载器](https://bbs.kanxue.com/#（2）遍历父类加载器)
- [（3）验证双亲委派机制](https://bbs.kanxue.com/#（3）验证双亲委派机制)
- [（4）动态加载](https://bbs.kanxue.com/#（4）动态加载)
- [（5）获得类列表](https://bbs.kanxue.com/#（5）获得类列表)
- [三、实验总结](https://bbs.kanxue.com/#三、实验总结)
- [四、参考文献](https://bbs.kanxue.com/#四、参考文献)

## 一、前言

最近一直在学习Android 加壳和脱壳，在进行Android加壳和脱壳的学习中，第一步便是深入理解类加载器和动态加载二者之间的关系，本文详细的介绍了类加载器和动态加载之间的关系和原理，之所以详细的讲解两者之间的关系，一是学习脱壳和加壳的需要，二是为后面Android插件化漏洞挖掘的讲解做铺垫。

## 二、类加载器

Android中的类加载器机制与JVM一样遵循双亲委派模式

### 1.双亲委派模式

#### （1）双亲委派模式定义

|  |  |
| --- | --- |
| 1  2 | `（1）加载.class文件时，以递归的形式逐级向上委托给父加载器ParentClassLoader加载，如果加载过了，就不用再加载一遍`  `（2）如果父加载器没有加载过，继续委托给父加载器去加载，一直到这条链路的顶级，顶级ClassLoader如果没有加载过，则尝试加载，加载失败，则逐级向下交还调用者加载` |

![image-20211009111527762](https://bbs.kanxue.com/upload/attach/202202/905443_WAMK8ZY23F6MM42.png)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException{`  `//1.先检查是否已经加载过--findLoaded`  `Class<?> c` `=` `findLoadedClass(name);`  `if` `(c` `=``=` `null) {`  `try` `{`  `//2.如果自己没加载过,存在父类,则委托父类`  `if` `(parent !``=` `null) {`  `c` `=` `parent.loadClass(name, false);`  `}` `else` `{`  `c` `=` `findBootstrapClassOrNull(name);`  `}`  `} catch (ClassNotFoundException e) {`  `}`    `if` `(c` `=``=` `null) {`  `//3.如果父类也没加载过,则尝试本级classLoader加载`  `c` `=` `findClass(name);`  `}`  `}`  `return` `c;`  `}` |

|  |  |
| --- | --- |
| 1  2  3  4 | `代码解释：`  `（1）先检查自己是否已经加载过class文件，用findLoadedClass方法，如果已经加载了直接返回`  `（2）如果自己没有加载过，存在父类，则委派父类去加载，用parent.loadClass(name,false)方法，此时会向上传递，然后去父加载器中循环第1步，一直到顶级ClassLoader`  `(3) 如果父类没有加载，则尝试本级classLoader加载，如果加载失败了就会向下传递，交给调用方式实现.class文件的加载` |

#### （2）双亲委派模式加载流程

![image-20211009111527762](https://bbs.kanxue.com/upload/attach/202202/905443_9TRKTFAGXAU92CP.png)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `我们要加载一个class文件，我们定义了一个CustomerClassLoader类加载器:`  `(1)首先会判断自己的CustomerClassLoader否加载过,如果加载过直接返回,`  `(2)如果没有加载过则会调用父类PathClassLoader去加载,该父类同样会判断自己是否加载过,如果没有加载过则委托给父类BootClassLoader去加载,`  `(3)这个BootClassLoader是顶级classLoader,同样会去判断自己有没有加载过,如果也没有加载过则会调用自己的findClass(name)去加载,`  `(4)如果顶级BootClassLoader加载失败了,则会把加载这个动作向下交还给PathClassLoader,`  `(5)这个PathClassLoader也会尝试去调用findClass(name);去加载,如果加载失败了,则会继续向下交还给CustomClassLoader来完成加载,这整个过程感觉是一个递归的过程,逐渐往上然后有逐渐往下,直到加载成功`  `其实这个String.class在系统启动的时候已经被加载了,我们自己定义一个CustomerClassLoader去加载,其实也是父类加载的` |

#### （3）双亲委派的作用

|  |  |
| --- | --- |
| 1  2  3 | `(1) 防止同一个.class文件重复加载`  `(2) 对于任意一个类确保在虚拟机中的唯一性.由加载它的类加载器和这个类的全类名一同确立其在Java虚拟机中的唯一性`  `(3) 保证.class文件不被篡改，通过委派方式可以保证系统类的加载逻辑不被篡改` |

### 2. Android中类加载机制

#### （1）Android基本类预加载

我们了解Android基本类预加载，首先我们回顾上文的Dalvik虚拟机启动相关：

![image-20211010143220857](https://bbs.kanxue.com/upload/attach/202202/905443_V58D7JCPJ8MMADK.png)

我们执行`app_process`程序，进入`main`函数里面，然后进行`AndroidRuntime::start`:

![image-20211010143128565](https://bbs.kanxue.com/upload/attach/202202/905443_V4ZJKYSSTPCY5RG.png)

|  |  |
| --- | --- |
| 1  2  3  4 | `Zygote native 进程主要工作：`  `（1）创建虚拟机–startVM`  `（2）注册JNI函数–startReg`  `（3）通过JNI知道Java层的com.android.internal.os.ZygoteInit 类，调用main 函数，进入java 世界` |

![image-20211010143826461](https://bbs.kanxue.com/upload/attach/202202/905443_275GXVBMF2K3P5U.png)

然后进入Java层：

![image-20211010144010024](https://bbs.kanxue.com/upload/attach/202202/905443_N3QHTBHU7RVY5Q2.png)

|  |  |
| --- | --- |
