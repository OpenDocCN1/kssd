
# [原创]一道简单但不简约的堆题——CISCN2026初赛堆题robo分析

## 题目描述

![](upload/attach/202605/1060264_3DPDBSWFNTGP8HT.png)

CISCN2026线上初赛的题目，线上只有一解，查了下发现过去五个月了还没有人写过WP，就尝试了一下，结果发现不是很难，可能是限时+看到cpp逆向大家都不愿意动手写吧（附件在结尾）

## 基本信息

![](upload/attach/202605/1060264_32BBK68P5NEAS82.png)

Glibc版本2.42，算很新的版本了，高版本Glibc的ptmalloc机制更新可以移步我的上一篇文章[[原创]从2026CCB决赛堆题CreditMarket学习Glibc2.39后ptmalloc的机制更新](https://bbs.kanxue.com/thread-291123.htm)，以及熟悉的保护全开

## 漏洞分析

### 伪代码分析

![](upload/attach/202605/1060264_DQD6CRNUBHASG73.png)

一打开IDA就是一股浓厚的cpp味，不知道大佬们cpp逆向一般怎么做比较快，我只会土办法，一边看伪代码一边还原cpp的逻辑，还原出的cpp逻辑如下

```cpp
class SafeHeapManager {
public:
    struct Chunk {
        std::size_t size = 0;
        bool in_use = false;
        bool contains(std::uintptr_t chunk_base, std::uintptr_t addr, std::size_t len) const {
            if (in_use != true || len == 0) {
                return len == 0;
            }
            if (addr < chunk_base) {
                return false;
            }
            if (size >= len) {
                return size + chunk_base - len >= addr;
            }
            return false;
        }
    };

    struct Shard : public std::map<std::uintptr_t, Chunk> {};
    Shard &get_shard(std::uintptr_t ptr) {
        return shards_[(ptr >> 4) & 0x3ff];
    }
    std::pair<std::uintptr_t, Chunk *> find_in_shard(Shard &shard, std::uintptr_t addr) {
        auto it = shard.upper_bound(addr);
        if (it == shard.begin()) {
            return {0, nullptr};
        }
        --it;
        const std::uintptr_t chunk_base = it->first;
        Chunk &chunk = it->second;
        if (addr < chunk_base || addr >= chunk_base + chunk.size) {
            return {0, nullptr};
        }
        return {chunk_base, &chunk};
    }
    void register_chunk(void *ptr, std::size_t size) {
        const auto addr = reinterpret_cast<std::uintptr_t>(ptr);
        if (addr != 0 && size != 0) {
            Chunk &chunk = get_shard(addr)[addr];
            chunk.size = size;
            chunk.in_use = true;
        }
    }
    bool unregister_chunk(void *ptr) {
        const auto addr = reinterpret_cast<std::uintptr_t>(ptr);
        if (addr == 0) {
            return false;
        }
        Shard &shard = get_shard(addr);
        auto it = shard.find(addr);
        if (it == shard.end()) {
            return false;
        }
        it->second.in_use = false;
        shard.erase(it);
        return true;
    }
    void safe_free(void *ptr) {
        if (unregister_chunk(ptr)) {
            std::free(ptr);
        }
    }
    bool safe_write(void *ptr, const void *src, std::size_t len) {
        if (ptr == nullptr || src == nullptr) {
            return false;
        }
        if (len == 0) {
            return true;
        }
        const auto addr = reinterpret_cast<std::uintptr_t>(ptr);
        Shard &shard = get_shard(addr);
        auto [chunk_base, chunk] = find_in_shard(shard, addr);
        if (chunk == nullptr || !chunk->contains(chunk_base, addr, len)) {
            return false;
        }
        std::memcpy(ptr, src, len);
        return true;
    }
    bool safe_read(void *ptr, void *dst, std::size_t len) {
        if (ptr == nullptr || dst == nullptr) {
            return false;
        }
        if (len == 0) {
            return true;
        }
        const auto addr = reinterpret_cast<std::uintptr_t>(ptr);
        Shard &shard = get_shard(addr);
        auto [chunk_base, chunk] = find_in_shard(shard, addr);
        if (chunk == nullptr || !chunk->contains(chunk_base, addr, len)) {
            return false;
        }
        std::memcpy(dst, ptr, len);
        return true;
    }
private:
    std::array<Shard, 1024> shards_;
};
static SafeHeapManager g_safe_heap_manager;
static unsigned char *base = nullptr;
static int sandbox() {
}
static void init() {
    setvbuf(stdin, nullptr, _IONBF, 0);
    setvbuf(stdout, nullptr, _IONBF, 0);
    setvbuf(stderr, nullptr, _IONBF, 0);
    sandbox();
    base = static_cast<unsigned char *>(std::malloc(0x1000));
    std::free(base);
}
static void menu() {
    std::cout << "1. malloc" << std::endl;
    std::cout << "2. free" << std::endl;
    std::cout << "3. write" << std::endl;
    std::cout << "4. read" << std::endl;
}
int main() {
    init();
    while (true) {
        int choice = 0;
        std::size_t size = 0;
        menu();
        std::cout << "YourChoice:";
        if (!(std::cin >> choice)) {
            return 0;
        }
        if (choice == 1) {
            std::cout << "size:";
            std::cin >> size;
            void *chunk = std::calloc(1, size);
            static void *chunk_pointer_1 = nullptr;
            static std::size_t size_2 = 0;
            if (chunk != nullptr) {
                chunk_pointer_1 = chunk;
                size_2 = size;
            }
            g_safe_heap_manager.register_chunk(chunk_pointer_1, size_2);
            std::cout << "allocated at 0x" << std::hex
                      << reinterpret_cast<std::uintptr_t>(chunk)
                      << std::dec << std::endl;
        } else if (choice == 2) {
            std::cout << "address:";
            std::cin >> value;
            g_safe_heap_manager.safe_free(base + value);
            std::cout << "freed" << std::endl;
        } else if (choice == 3) {
            std::size_t size = 0;
            std::cout << "address:";
            std::cin >> value;
            std::cout << "size:";
            std::cin >> size;
            std::vector<unsigned char> data(size);
            std::cout << "data (hex):";
            for (std::size_t i = 0; i < size; ++i) {
                unsigned int byte = 0;
                std::cin >> std::hex >> byte >> std::dec;
                data[i] = static_cast<unsigned char>(byte);
            }
            if (g_safe_heap_manager.safe_write(base + value, data.data(), data.size())) {
                std::cout << "write success" << std::endl;
            } else {
                std::cout << "write failed" << std::endl;
            }
        } else if (choice == 4) {
            std::size_t size = 0;
            std::cout << "address:";
            std::cin >> value;
            std::cout << "size:";
            std::cin >> size;
            std::vector<unsigned char> data(size);
            if (g_safe_heap_manager.safe_read(base + value, data.data(), data.size())) {
                for (unsigned char byte : data) {
                    std::printf("%02x ", byte);
                }
                std::cout << std::endl;
            } else {
                std::cout << "read failed" << std::endl;
            }
        } else {
            std::cout << "invalid choice" << std::endl;
            std::exit(0);
        }
    }
}
```

可以发现，这道题设计了一个堆管理器，维护了一个结构体，并且通过对应的结构体来确定堆块是否使用、大小，并且以初始化时的一个堆作为基地址，返回一个`offset`，并且查找堆块的逻辑是通过`offset`而非`index`，除此之外，`add`使用了`calloc`（Glibc2.41之后`calloc`也会优先使用`tcache`），但是`add`的`size`输入没有限制，`add`、`edit`、`show`、`delete`的次数均无限制。

### 调试分析

![](upload/attach/202605/1060264_R8T2JU94XZYYDY4.png)

申请了一个0x400的堆块，发现除了我们申请的之外，程序还自己申请了一个0x30的堆块，我们可以看一下内容

![](upload/attach/202605/1060264_J95M96Z4N2P6Z3K.png)

到这里还看不是很太懂+0x0到+0x18的对应值，但是+0x20和+0x28的值很明显存放了我们所申请堆块的用户指针和用户大小，那就只能再申请一个堆块看结果了

![](upload/attach/202605/1060264_JMV5QKZU4R9JV8Q.png)

可以看到，确实是每申请一个新堆块就会伴随这个0x30的小块，而且也确定+0x20以及+0x28处存放的一定是堆块的用户指针和用户大小

![](upload/attach/202605/1060264_W2T2AZ72EG26GXT.png)

接下来我们尝试一下`free`，可以发现，需要我们传入的是一个`address`，但是这里很坑，最开始试着传了0x300和768，结果发现都不能正常free，最后发现要传300..

![](upload/attach/202605/1060264_JR48BH5NMGJH6ED.png)

发现堆块也确实被正常`free`了（pwndbg对高版本Glibc的支持有点奇怪，对进入`tcache`的块判定有点问题，但是通过看堆块被写入的`next`指针和`key`可以判断其已经进入了`tcache`）,同时，伴随堆块也同时被free了

![](upload/attach/202605/1060264_ZM4CEZBKVH3X8QS.png)

`edit`和`show`的逻辑就一起展示了，这个传入数据要求hex，`show`的输出格式也需要单独写

![](upload/attach/202605/1060264_2XVCY97Z2K4JSPV.png)

测了一下发现没有UAF，也没有堆溢出，虽然没直接找到漏洞，但这短短几分钟的调试能帮我们少分析很大一部分代码

### 再回伪代码
