
**FastHook**：<a href="elink@c89K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4u0A6L8X3N6Q4x3X3c8@1k6h3y4Z5L8X3W2U0K9h3q4F1i4K6u0r3c8X3q4K6N6p5S2G2L8$3D9`."><mark class="encrypted">cb6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4u0A6L8X3N6Q4x3X3c8@1k6h3y4Z5L8X3W2U0K9h3q4F1i4K6u0r3c8X3q4K6N6p5S2G2L8$3D9`.</mark></a>

# 一、 概述

**经过实际项目大量测试验证，FastHook表现出了远超YAHFA的优异稳定性**。用户反馈未出现Hook引发的稳定性问题、压力测试也未发生Hook引发的稳定问题。之所以FastHook拥有优异的稳定性，**除了框架实现原理的优越性之外，还得益于FastHook出色的细节处理**。

本文将通过FastHook实现原理优越性与一些出色的细节处理来解释为何FastHook拥有优异的稳定性，最后对比其YAHFA。

# 二、先天优势

如果你还未了解FastHook，请移步**[FastHook——一种高效稳定、简洁易用的Android Hook框架](<https://bbs.kanxue.com/elink@417K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2c8#2M7X3W2F1k6#2c8W2j5$3S2F1K9h3y4A6j5h3&6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3^5y4U0p5K6y4e0f1#2)**。

FastHook相较YAHFA原理上最大的优势、也是最大的亮点便是：**不需要备份原方法！不需要备份原方法！不需要备份原方法！**

科学上有一个著名的“奥卡姆剃刀定律”，什么意思呢？如果一个现象有两个或者多个不同的理论解释，那么选最简单的那个。做Hook框架，也可以用剃刀定律来做指导：**实现相同的功能，选对系统状态改动最小的**。

**“备份原方法”是一种隐患颇多的方式，引发了诸如方法解析出错、Moving GC空指针等问题**。尽管其他框架通过一些手段来提高稳定性，比如保证方法不被再次解析、检查Moving GC是否移动了原方法相关对象等，但是这些都不是理论安全的，**就像地上有个坑，你不去补上，而是让人不要去踩**。

反观FastHook，Hook时对系统原有状态的改变是最小的。

1. **Inline模式改变的仅是几个字节的指令，因平台而异，不篡改任何方法。**

2. **EntryPoint模式替换了方法EntryPoint，但是原方法将强制为解释执行，也可等价的看为未做修改。**

**简而言之，FastHook就是用Hook方法hook原方法，原方法hook Forward方法来实现最小改动hook。完美地从实现层面解决了YAHFA不能解决的问题，而且无需做一些其他操作，YAHFA都需要一些其他的操作来提高稳定性，而FastHook不需要做任何其他处理，更简洁、更优雅**。

# 三、比YAHFA更出色的细节处理

## 3.1 JIT状态检查

**如果你看YAHFA代码，你会发现其未做JIT状态检查**。JIT状态检查的目的是为了保证hook的安全性，但这也不是理论安全的，也无法做到理论安全。这是为什么呢？

### 3.1.1 Inline模式

如果原方法未编译则需要进行手动JIT编译。那么问题来了，什么时候编译才是安全的呢。下面列举出所有可能出现的情景:

1. **原方法未进行JIT编译，此时手动JIT编译时安全的**

2. **原方法未进行JIT编译，即将进入编译等待队列或已进入编译等待队列，此时手动JIT编译是不安全的**

3. **原方法正在JIT编译，此时手动JIT编译是不安全的**

4. **原方法编译完成，此时手动编译是安全的**

上述4中情景，其中2、3是不安全的。如果要保证手动JIT编译的安全性，必须做到以下两点：

1. **禁止JIT编译，防止从1变化到2**

2. **能够判断2、3，当处于2、3状态时，等待其变化到4**

现在来看看FastHook到底是怎么处理的

```
int CheckJitState(JNIEnv *env, jclass clazz, jobject target_method) {

    void *art_method = (void *)(*env)->FromReflectedMethod(env, target_method);

    //添加kAccCompileDontBother，禁止JIT、AOT编译

    AddArtMethodAccessFlag(art_method, kAccCompileDontBother);

    uint32_t hotness_count = GetArtMethodHotnessCount(art_method);

    if(hotness_count >= kHotMethodThreshold) {

        //hotness_count >= hot_threshold，肯定就不是1了，看看是2、3、4中的哪一个

        long entry_point = (long)GetArtMethodEntryPoint(art_method);

        if((void *)entry_point == art_quick_to_interpreter_bridge_) {

            void *profiling = GetArtMethodProfilingInfo(art_method);

            void *save_entry_point = GetProfilingSaveEntryPoint(profiling);

            if(save_entry_point) {

                //JIT垃圾回收会改变方法EntryPoint，虽然方法已经编译了，但是EntryPoint也可能是art_quick_to_interpreter_bridge

                return kCompile;

            }else {

                //JIT状态保存在profiling中，通过其来判断是否是正在编译，如果不是可能是正在等待或者已经编译失败。

                bool being_compiled = GetProfilingCompileState(profiling);

                if(being_compiled) {

                    return kCompiling;

                }else {

                    return kCompilingOrFailed;

                }

            }

        }

        return kCompile;

    }else {

        //hotness_count < hot_threshold，可能是1，也可能是2，即将进入编译等待队列，统一加一个增量，如果此时大于hot_threshold，就认为是2，反之是1

        uint32_t assumed_hotness_count = hotness_count + kHotMethodMaxCount;

        if(assumed_hotness_count > kHotMethodThreshold) {

            return kCompiling;

        }

    }

    return kNone;

}
```

```
class ProfilingInfo {

 private:

  ProfilingInfo(ArtMethod* method, const std::vector<uint32_t>& entries);



  // Number of instructions we are profiling in the ArtMethod.

  const uint32_t number_of_inline_caches_;



  // Method this profiling info is for.

  // Not 'const' as JVMTI introduces obsolete methods that we implement by creating new ArtMethods.

  // See JitCodeCache::MoveObsoleteMethod.

  ArtMethod* method_;



  // Whether the ArtMethod is currently being compiled. This flag

  // is implicitly guarded by the JIT code cache lock.

  // TODO: Make the JIT code cache lock global.

  bool is_method_being_compiled_;

  bool is_osr_method_being_compiled_;



  // When the compiler inlines the method associated to this ProfilingInfo,

  // it updates this counter so that the GC does not try to clear the inline caches.

  uint16_t current_inline_uses_;



  // Entry point of the corresponding ArtMethod, while the JIT code cache

  // is poking for the liveness of compiled code.

  const void* saved_entry_point_;



  // Dynamically allocated array of size `number_of_inline_caches_`.

  InlineCache cache_[0];

};
```

1. **`AddArtMethodAccessFlag(art_method, kAccCompileDontBother)`，设置`kAccCompileDontBother`禁止JIT、AOT。防止1变化到2**。

2. **如果`hotness_count > hot_threshold`，这时肯定就不是1了**，还需要判断是2、3、4中哪一个。

3. **通过判断entry point是否为解释执行入口来判断是否是4，因为entry point不是解释执行入口肯定不会是2和3**。

4. 这里有个关键点一定要注意，即使JIT编译后entry point也有可能为解释执行入口，**因为JIT垃圾回收会将entry point设置为解释执行入口，将实际入口保存在`save_entry_point`**。如果`save_entry_point`不为空，那证明已经编译过了。

5. 怎么判断2、3呢？**每个方法都有一个profiling info，保存一些运行过程信息和JIT编译信息，其中就有是否在JIT编译的信息**。如果为true，则为3，如果为false，则为2（这里也可能是编译失败了的，为了简便都做2看待）

6. 如果`hotness_count < hot_threshold`，能说明一定是1吗？答案是不能，也有可能是2。这是为什么呢？有一种罕见的情况，当我们检查状态时，`hotness_count`还未执行到更新的代码，而当其更新之后大于`hot_threshold`，那么实际就是2。**因此假设`hotness_count`会更新，给一个增量**（理论上给不了准确的数值，因为其增量受权重影响，也可能是批量处理的增量，因此这不是理论安全的），**这里给一个比较大的值**（50），**如果此时大于`hot_threshold`，就认为是2**（这个也不是完全准确的，因为可能`hotness_count`根本不会更新）。

### 3.1.2 小结

1. **hook之前先做JIT状态检查，如果安全就立即hook，反之放入一个异步队列延迟hook**

2. **上述分析可知，该检查也不是绝对安全的，但是已经将出现问题的场景缩小到一个可以忽略不计的范围**。

3. **EntrypPoint替换模式的检查与Inline模式一致，不做重复分析**。

## 3.2 判断方法是否需要编译

**如果只是简单用entry point与解释入口比较来判断，通过3.1的分析可知这是不完备的**。

JIT垃圾回收会改变entry point为解释入口，必须做进一步判断是否为JIT编译方法。FastHook的做法很简单，**判断`hotness_count`是否小于`hot_threshold`，如果其小于`hot_threshold`，那肯定还未被JIT编译，因此可以判定其需要进行手动JIT编译**。

并且，这一步是在JIT检查成功基础上进行的，可以不用担心JIT状态的影响。

```
bool IsCompiled(JNIEnv *env, jclass clazz, jobject method) {

    bool ret = false;

    void *art_method = (void *)(*env)->FromReflectedMethod(env, method);

    void *method_entry = (void *)ReadPointer((unsigned char *)art_method + kArtMethodQuickCodeOffset);

    int hotness_count = GetArtMethodHotnessCount(art_method);

   if(method_entry != art_quick_to_interpreter_bridge_)

        ret = true;

    if(!ret && hotness_count >= kHotMethodThreshold)

        ret = true;

    return ret;

}
```

## 3.3 线程状态恢复

当一个java方法进入JNI时，线程状态由runnable状态变为native状态，返回java前恢复为runable状态。**而JIT编译方法会将参数thread的状态转变为runnable状态**。

**最开始FastHook在手动JIT编译方法时不做其他处理**。但是后来项目上有反馈，有概率出现crash，出现的位置正好是编译完成后返回java的地方，异常原因是线程状态错误。

FastHook之前的解决方案是：新建native线程用于JIT编译，避免当前线程编译。这时出现了新的问题，如何获取native线程的thread对象？

通过研究android代码发现，**art获取线程thread对象是通过TLS来获取的，thread存储在TLS固定位置**。但实际上，这种方案虽然解决了crash的问题，但也导致了新的问题：**线程错误地等待**。

究其缘由，都是线程状态异常引起的，**因此根治的方法便是恢复线程状态**。通过研究`Thread`代码发现，线程状态是一个union结构体`StateAndFlags`，保存在thread对象里，因此可以通过偏移的方式来访问。

```
static inline void *CurrentThread() {

    return __get_tls()[kTLSLotArtThreadSelf];

}

#if defined(__aarch64__)

# define __get_tls() ({ void** __val; __asm__("mrs %0, tpidr_el0" : "=r"(__val)); __val; })

#elif defined(__arm__)

# define __get_tls() ({ void** __val; __asm__("mrc p15, 0, %0, c13, c0, 3" : "=r"(__val)); __val; })

#endif
```
