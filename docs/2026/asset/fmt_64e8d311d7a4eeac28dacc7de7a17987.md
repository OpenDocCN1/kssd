
使用`vmmap`命令，可以查看内存属性。

![](https://bbs.kanxue.com/upload/attach/201803/611407_K8DS9T5KHRRNGGH.jpg)

`0xee186000`是`MemoryHeapBase`的起始地址，大小为`0x2000`。

如果从`0xee187fff`覆盖，`0xee187fff + 0x2000 > 0xee188000`，这就导致堆溢出了，直接触发了堆保护页。

所以这个漏洞也是无法利用的。尴尬。。。。。。。。。

到此调试分析结束。

## 补丁分析

看看补丁吧：

`10aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5i4k6Q4x3V1k6Q4x3V1u0Q4x3V1j5^5y4K6p5@1x3e0u0U0k6X3p5H3y4e0M7%4x3r3y4X3k6o6S2T1k6e0m8S2x3e0x3H3j5U0j5^5x3K6R3$3y4K6M7#2y4o6b7#2x3o6f1%4i4K6t1#2y4f1g2Q4x3U0f1J5x3g2)9J5c8W2)9J5x3@1j5H3`

```cpp
//#define LOG_NDEBUG 0

#define LOG_TAG "ICrypto"

-#include <utils/Log.h>

-

#include <binder/Parcel.h>

#include <binder/IMemory.h>

+#include <cutils/log.h>

#include <media/ICrypto.h>

#include <media/stagefright/MediaErrors.h>

#include <media/stagefright/foundation/ADebug.h>

#include <media/stagefright/foundation/AString.h>

+#include <utils/Log.h>

namespace android {

@@ -362,6 +362,13 @@
                    reply->writeInt32(BAD_VALUE);
                    return OK;
                }

+               sp<IMemory> dest = destination.mSharedMemory;
+               if (totalSize > dest->size() ||
+                       (size_t)dest->offset() > dest->size() - totalSize) {
+                   reply->writeInt32(BAD_VALUE);
+                   android_errorWriteLog(0x534e4554, "71389378");
+                   return OK;
+               }
            }

            AString errorDetailMsg;
```

直接在判断完`destination`的类型后，就判断`dest->offset + totalSize`不能大于`dest->size`。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班，第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-3-23 01:30 被ID蝴蝶编辑，原因：
