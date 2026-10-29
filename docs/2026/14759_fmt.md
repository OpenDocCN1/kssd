
# [原创] root 环境下直接修改 ro 属性

属性是由 `init` 进程读取到内存中，然后通过 `mmap` 只读方式共享到其他进程中。读取时，每个进程都可以直接读取；修改或者添加只能由 `init` 进程进行，这样方便 `selinux` 权限管理。

有 `root` 权限后，我们可以通过 `mmap` 读写的方式，将属性内存共享到我们的内存中，这样可以任意修改，绕过 `init` 的限制。

## 属性内存结构

上图其实很形象，是字典树和二叉树的混合。结点信息数据结构如下：

```c
typedef struct prop_bt {
    uint8_t namelen;        // 该结点名字长度
    uint8_t reserved[3];    // 对齐的作用
    uint32_t prop;          // 指向 prop_info，里面保存 key 和 value 信息
    uint32_t left;          // 左兄弟，长度小于或者长度相同而名字小于当前结点
    uint32_t right;         // 右兄弟，长度大于或者长度相同而名字大于当前结点
    uint32_t children;      // 子结点
    char name[0];
} prop_bt;

typedef struct prop_info {
    uint32_t serial;
    char value[PROP_VALUE_MAX]; // PROP_VALUE_MAX = 92，所以说属性值最大长度为 91，'\0' 结尾
    char name[0];               // 属性名字
} prop_info;
```

## 思路

1. Android N 上，首先读取 `/property_contexts` 文件，获取属性前缀和 `security context`（`selinux` 定义的）的关系。由于每个 `security context` 对应一个文件，所以需要通过前缀来找到对应的文件。6.0 及以下，所有属性都保存在 `/dev/__properties__` 文件中，可以不读取 `/property_contexts` 文件。
2. 打开对应的文件，将属性以 `.` 分隔，将第一部分和头结点对比。若相同，则将第二部分和 `children` 对比；若不同，且小于，则继续使用第一部分比较 `left` 结点；若大于，则比较 `right` 结点。依次类推，若最后部分和结点 N 相等，则 N 的 `prop` 就是所要找到的 `prop_info`。

附上源码：[22fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4i4K6u0V1g2#2c8W2M7V1c8S2L8W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4y4h3k6H3M7X3!0H3k6i4u0@1K9h3g2K6](https://bbs.kanxue.com/elink@18aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4i4K6u0V1g2#2c8W2M7V1c8S2L8W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4y4h3k6H3M7X3!0H3k6i4u0@1K9h3g2K6)

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
