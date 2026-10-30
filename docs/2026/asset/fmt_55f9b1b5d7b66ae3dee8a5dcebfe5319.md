
```bash
-rw------- 1 root root 8 Jan 19 14:36 resource1
```

`resource0` 对应 MMIO 空间。  
`resource1` 对应 PMIO 空间。

```bash
ubuntu@ubuntu:~$ cat /sys/devices/pci0000:00/0000:00:03.0/resource

/* start addr end addr flags */

0x00000000febf1000 0x00000000febf10ff 0x0000000000040200 （MMIO）
0x000000000000c050 0x000000000000c057 0x0000000000040101 （PMIO）
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
```

从左至右依次是：起始地址、结束地址、标志位。  
第一行是 MMIO、第二行是 PMIO。

## 从 QOM(Qemu Object Model) 模型看 `strng` 源码

虽然是第一次接触 `qemu-pwn`，不过对于著名的 QOM 模型早已经有所耳闻，于是就跟着 QOM 模型梳理一下 `strng` 的源码，加深一下理解。

### 什么是 QOM(Qemu Object Model)？

*Everything in QOM is a device*

QOM 是 QEMU **在 C 的基础上自己实现的一套面向对象机制**，负责将 device、bus 等设备都抽象成为对象。

**下面给出一些重要的 QOM Terminology(术语)**  
来源：<mark class="encrypted">898K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9K9h3&6#2P5q4)9J5k6r3E0$3L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2E0j5h3N6W2M7#2)9J5c8X3k6Q4x3V1k6X3y4W2)9J5c8U0t1H3x3e0u0Q4x3X3c8X3L8%4u0#2L8g2)9J5k6q4q4a6e0g2)9#2k6V1y4b7g2g2)9J5k6i4m8V1k6R3`.`.</mark>

• Type（类型） ‒ Defines class  
• Class（类） ‒ Stores static data & virtual method pointers ‒ Lazily initialized from type and, optionally, static data（存储静态数据以及一些函数指针）  
• Object（对象） ‒ Stores dynamic data (“chunk of allocated memory”) （储存动态数据）  
• Property（属性） ‒ Accessor to dynamic object data ‒ Inspectable via monitor interface（访问对象中的动态数据，通过 interface 可以进行监控）

### QOM 如何工作？

在 QOM 中有几个重要的结构体：

<mark class="encrypted">319K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4k6h3#2#2i4K6u0r3M7h3g2E0N6g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6A6L8X3y4D9N6h3c8W2i4K6u0r3M7h3!0E0i4K6u0r3L8$3u0B7k6h3y4@1i4K6u0W2K9l9`.`.</mark>

#### （1）`TypeInfo`

```c
struct TypeInfo
{
    const char *name;
    const char *parent; //父类
    size_t instance_size;
    size_t instance_align;
    void (*instance_init)(Object *obj); //实例初始化函数
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj); //实例“析构”函数
    bool abstract; //是否为抽象类
    size_t class_size; //类大小，官方文档中提出.class_size字段设置为sizeof(MyClass)
    void (*class_init)(ObjectClass *klass, void *data); //在所有父类被初始化完成后调用的子类初始化函数，可用于override virtual methods
    void (*class_base_init)(ObjectClass *klass, void *data);
    void *class_data;
    InterfaceInfo *interfaces; //封装了const char *type;
};
```

类比到 `strng` 源码：

```c
static void pci_strng_register_types(void)
{
    static const TypeInfo strng_info = { //这里就是一个TypeInfo结构体
        .name = "strng",
        .parent = TYPE_PCI_DEVICE, //父类为 TYPE_PCI_DEVICE
        .instance_size = sizeof(STRNGState),
        .instance_init = strng_instance_init,
        .class_init = strng_class_init, //类初始化，虚函数覆写
    };
    type_register_static(&strng_info);
}

type_init(pci_strng_register_types)
```

可以看到，在 `pci_strng_register_types` 中首先定义了 `TypeInfo strng_info` 并对其中一些内容进行赋值。

在代码最后部分，首先调用 `type_init()` 负责注册。

```c
#define type_init(function) module_init(function, MODULE_INIT_QOM)
#define module_init(function, type) \
    static void __attribute__((constructor)) do_qemu_init_ ## function(void)
    { \
        register_module_init(function,type); \
    }
```

最终调用了 `register_module_init()`

```c
typedef struct ModuleEntry
{
    void (*init)(void);
    QTAILQ_ENTRY(ModuleEntry) node;
    module_init_type type;
} ModuleEntry;

static void init_lists(void)
{
    static int inited;
    int i;
    if (inited) {
        return;
    }
    for (i = 0; i < MODULE_INIT_MAX; i++) {
        QTAILQ_INIT(&init_type_list[i]);
    }
    QTAILQ_INIT(&dso_init_list);
    inited = 1;
}

static ModuleTypeList *find_type(module_init_type type)
{
    init_lists();
    return &init_type_list[type];
}

void register_module_init(void (*fn)(void), module_init_type type)
{
    ModuleEntry *e;
    ModuleTypeList *l;
    e = g_malloc0(sizeof(*e));
    e->init = fn;
    e->type = type;
    l = find_type(type); //调用init_lists()做了初始化
    QTAILQ_INSERT_TAIL(l, e, node);
}
```

主要做了如下事情：

1. `malloc` 一个新的 `ModuleEntry` 结构体。并插入 `init_type_list[Module_Init_Type]` 链表中。
2. 将 `ModuleEntry` 的 `init` 函数指针赋值为 `fn`。（这个 `init` 也是当前这个 `module` 的初始化函数）

而需要注意的是，由于 `__attribute__((constructor))` 的存在，这一部分内容作为 `构造函数` 都将在 `main` 函数之前调用。  
具体的规则可以看：  
<a href="elink@3b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3K9r3!0^5K9i4y4Q4x3X3g2G2M7X3N6Q4x3V1j5J5x3o6p5I4i4K6u0r3x3o6c8Q4x3V1j5J5y4#2)9J5c8X3y4Q4x3X3c8D9j5h3&6Y4N6h3q4Y4k6g2)9J5k6r3y4G2L8Y4y4@1M7Y4g2U0N6r3!0J5M7#2)9J5k6r3q4F1k6q4)9J5k6r3c8W2M7%4c8J5N6h3y4@1L8%4u0K6i4K6u0V1N6$3W2@1K9q4)9J5k6r3N6U0j5#2)9J5c8R3`.`.">C Language Constructors and Destructors with GCC</a>

总结一下：  
在 QEMU 真正启动之前，所有 `module_entry` 都被插入 `init_type_list`。并且已经确定了初始化函数。

而接下来，在 qemu 启动阶段 `main` 函数调用的时候，会对 `init_type_list[Module_Init_Type]` 链表中的每一个 `ModuleEntry` 调用其 `init` 函数。所以对于我们的 `strng` 里就是 `pci_strng_register_types()`。  
`pci_strng_register_types()` 中首先对设备的一些信息做了初始化，接着调用了：`type_register_static(&strng_info)`

#### （2）`TypeImpl`

定义如下：

```c
struct TypeImpl
{
    const char *name;
    size_t class_size;
    size_t instance_size;
    size_t instance_align;
    void (*class_init)(ObjectClass *klass, void *data);
    void (*class_base_init)(ObjectClass *klass, void *data);
    void *class_data;
    void (*instance_init)(Object *obj);
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj);
    bool abstract;
    const char *parent;
    TypeImpl *parent_type;
    ObjectClass *class; //指向 ObjectClass 的指针
    int num_interfaces;
    InterfaceImpl interfaces[MAX_INTERFACES];
};
```

由 `type_register_static(&strng_info)` 最终调用了 `type_register_internal(&strng_info)`

```c
static TypeImpl *type_register_internal(const TypeInfo *info)
{
    TypeImpl *ti;
    ti = type_new(info); //根据info信息，创建一个TypeImpl对象
    type_table_add(ti); //将新建的TypeImpl对象注册到全局哈希表 type_table 中
    return ti;
}
```

至此，这个 `ModuleEntry` 向 `TypeImpl` 表中的注册就算完成了。

#### （3）`ObjectClass`

`ObjectClass` 是所有 class 的基类，定义如下：

```c
struct ObjectClass
{
    /* private: */
    Type type;
    GSList *interfaces;
    const char *object_cast_cache[OBJECT_CLASS_CAST_CACHE];
    const char *class_cast_cache[OBJECT_CLASS_CAST_CACHE];
    ObjectUnparent *unparent;
    GHashTable *properties;
};
```

在前面几步中，实际上我们只完成了 **Module 的注册**，而没有真正完成 **类的初始化**。

接下来我们通过 `TypeInfo strng_info` 中的 `.class_init = strng_class_init` 简单看一下 **Type(类)的初始化**。  
在初始化每一个 type 的时候调用的是：`type_initialize` 函数。  
给出对应的注释后的源码：

```c
static void type_initialize(TypeImpl *ti) //针对每一个注册的 TypeImpl 去做type 初始化
{
    TypeImpl *parent;
    //在 TypeImpl 中有 ObjectClass *class
    if (ti->class) { //如果 ti->class存在，那么说明初始化过了，直接return
        return;
    }
    ti->class_size = type_class_get_size(ti);
    ti->instance_size = type_object_get_size(ti);
    if (ti->instance_size == 0) { //如果instance_size为0，标记为抽象类
        ti->abstract = true;
    }
    if (type_is_ancestor(ti, type_interface)) {
        ......
    }
    ti->class = g_malloc0(ti->class_size);
    parent = type_get_parent(ti); // 是否有父类？即是否存在继承关系？
    if (parent) { // 如果存在继承关系的话
        type_initialize(parent); //初始化父类
        GSList *e;
        int i;
        g_assert(parent->class_size <= ti->class_size);
        g_assert(parent->instance_size <= ti->instance_size);
        memcpy(ti->class, parent->class, parent->class_size);//首先用父类数据 初始化
        ti->class->interfaces = NULL;
        for (e = parent->class->interfaces; e; e = e->next) {
            InterfaceClass *iface = e->data;
            ObjectClass *klass = OBJECT_CLASS(iface);
            type_initialize_interface(ti, iface->interface_type, klass->type);//初始化对应接口
        }
    }
}
```
