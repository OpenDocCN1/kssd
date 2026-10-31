
# [原创]Xposed Art-Runtime移植细节

前段时间想打造一个直接集成到本地MTK的Xposed环境。

手头有MTK的源代码，还有Xposed的源代码。所以移植就开始了。

在移植的过程中，看了老罗的Art虚拟机讲解，最后才开始移植。

在实战过程中，就是看代码3周，移植1小时。看来对Art虚拟机的足够了解，还是非常有必要的。

下面附上移植的一些细节，还有Art-Runtime代码的对照。

## Xposed移植点

### 一、背景

前段时间，对Xposed感兴趣，想移植一下Xposed到本地代码。移植成功后，做了以下对照。该移植在于Art虚拟机的移植。下面就分别介绍移植点。

### 二、移植位置

红色的是新增加的代码。可以用Compare软件，来进行移植工作，比较方便。

1. `art\runtime\base\logging.h`

![]()

![]()

增加Log属性。

2. `art\runtime\base\logging.cc`

![]()

实现一个Xposed的Log。

3. `art\runtime\entrypoints\entrypoint_utils.h`、`art\runtime\entrypoints\entrypoint_utils.cc`

![]()

操作Hook后的方法。

![]()

![]()

上面的方法，详细的解释了，如果是Xposed Hook的方法被检测到时，如何去调用的过程。

4. `art\runtime\art_method.h`、`art\runtime\art_method.cc`、`art\runtime\art_method-inl.h`

![]()

增加一个用于Hook的结构体。

这个结构体是Xposed能进行Hook的一个关键。

判断方法时，增加Xposed方法的判断。

![]()

![]()

Quick入口设置时，还有一定的要求。

![]()

增加一些与Xposed相关的数据结构调用方法及字段。

下面`StackReplaceMethod`的方法，用于更换。

![]()

![]()

这一段代码在`ArtMethod.cc`中，是进行方法替换的核心逻辑。

Xposed就是改造了Art的`ArtMethod`这一类，该类其实就是Art虚拟机在运行时的方法对象。

最后生成`hook_info`，通过`SetEntryPointFromJni`来设置到对方法区。需要用的时候再通过`soa.DecodeMethod(method)->GetXposedHookInfo()`来获取，进而调用。

![]()

5. `art\runtime\class_linker.cc`

因为`method->IsProxyMethod(true)`变了，导致的改变。

6. `art\runtime\debugger.cc`

![]()

7. `art\runtime\instrumentation.cc`

![]()

8. `art\runtime\jni_internal.cc`

![]()

9. `art\runtime\modifiers.h`

增加Xposed的Constant变量。

![]()

10. `art\runtime\oat.h`、`art\runtime\oat.cc`

![]()

![]()

![]()

![]()

一些Xposed版本的信息，其实这里可以不要，然后下面`oat_file_assistant`中检查的时候忽略。

11. `art\runtime\oat_file_assistant.cc`

![]()

11. `art\runtime\reflection.cc`、`art\runtime\runtime.h`、`art\runtime\thread.cc`、`art\runtime\utils.cc`

没有太多难懂的方法。

12. `art\runtime\stack.h`

![]()

![]()

增加一个方法设置。

### 三、总结

Xposed 在Art虚拟机的源代码的Runtime中做了一些修改。给`ArtMethod`增加了一个功能。最后实现了Hook功能。

文笔不好，请见谅。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Xposed框架对Art虚拟机的修改点.pdf](javascript:void(0);)
  （869.97KB，321次下载）
- [source.zip](javascript:void(0);)
  （3.83MB，182次下载）
