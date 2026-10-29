
### 3.SandHook hook实操

1. 将 SandHook 中 `cpp` 文件夹下的包全部导入到项目中，并修改 `CMakeLists.txt`，添加 `native.cpp`，修改 Java 层导入的 so 库为 `sandHook-native`。
2. 配置相关环境。

   在配置文件 `build.gradle` 中配置：

   ```gradle
   externalNativeBuild {
       cmake {
           arguments '-DBUILD_TESTING=OFF'
           cppFlags "-frtti -fexceptions -Wpointer-arith"
           abiFilters 'armeabi-v7a', 'arm64-v8a'
       }
   }
   ```

3. 编译可以成功通过。
4. 使用：

   ```cpp
   const char *libc = "/system/lib64/libc.so";
   old_fopen = reinterpret_cast<void *(*)(char *, char *)>(
       SandInlineHookSym(
           libc,
           "fopen",
           reinterpret_cast<void *>(new_fopen)
       )
   );
   ```

   参数 `2`：hook 的函数。参数 `3`：新的函数。

   添加原理 hook 旧函数的声明：

   ```cpp
   void *(*old_fopen)(char *, char *);
   ```

   实现新的函数功能：

   ```cpp
   void *new_fopen(char *a, char *b) {
       __android_log_print(6, "windaa", "I am from new open %s", a);
       return old_fopen(a, b);
   }
   ```

5. 运行测试是否成功启动。

### 4. PLT/GOT hook 实操

前面我们已经介绍了 GOT 表 hook 的原理，下面我们实例操作一下导入表函数的 hook。

参考博客：[020K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0G2k6r3S2S2j5$3E0W2M7W2)9J5c8Y4m8Q4x3V1j5&6x3K6l9$3z5e0V1%4i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@80eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0G2k6r3S2S2j5$3E0W2M7W2)9J5c8Y4m8Q4x3V1j5&6x3K6l9$3z5e0V1%4i4K6u0W2K9s2c8E0L8l9`.`.)

#### 原理

通过解析 ELF 格式，分析 Section header table，找出静态的 `.got` 表的位置，并在内存中找到相应的 `.got` 表位置。此时，内存中的 `.got` 表保存着导入函数的地址。读取目标函数地址，与 `.got` 表每一项函数入口地址进行匹配，找到后直接替换新的函数地址，这样就完成了一次导入表的 Hook 操作。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_ZQSAKZF2U7B5GNE.png)

首先，我们编写 demo。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_QM94VN789TPJJTK.png)

我们编译后使用 010Editor 打开 `libnative-lib.so`。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_EKYS5SE78TQUKNV.png)

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_GVKDZZSTB8VBTEA.png)

然后我们用 IDA 打开，并直接跳转到该地址。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_V3PY8VJY2TYXEEX.png)

在 GOT 表中我们找到对应的 `mywin0` 函数。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_6WBC3YU8VG6KQMG.png)

#### 4.1 获得 so 模块的加载地址

我们可以使用 `/proc/self/maps` 去获得 so 模块的加载地址。

```cpp
char line[1024];
int *start;
int *end;
int n = 1;

// 1.拿到 so 的起始地址
// 749e5d7000-749e5db000 r--p 000f4000 103:09 441 /system/bin/linker64
// 749e5db000-749e5dc000 rw-p 000f8000 103:09 441 /system/bin/linker64

FILE *fd = fopen("/proc/self/maps", "r");
while (fgets(line, sizeof(line), fd)) {
    if (strstr(line, "libnative-lib.so")) {
        __android_log_print(6, "windaa", "%s", line);
        if (n == 1) {
            start = reinterpret_cast<int *>(
                strtoul(strtok(line, "-"), NULL, 16)
            );
            end = reinterpret_cast<int *>(
                strtoul(strtok(NULL, " "), NULL, 16)
            );
        } else {
            strtok(line, "-");
            end = reinterpret_cast<int *>(
                strtoul(strtok(NULL, " "), NULL, 16)
            );
        }
        n++;
    }
}
```

#### 4.2 找到 GOT 表的位置

我们首先根据段头找到 `section_header` 的首地址。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_Q7NVUZ5B2X57YS4.png)

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_TVKKU6G357P3HWZ.png)

然后我们遍历这个表就可以找到 `.got`，再根据 GOT 表地址轮询找到函数地址。

因为这种方法不能在内存中直接找到段头，内存中会抹去段头，所以我们可以通过加载 so 文件来定位。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_K8TVAMSKQ6KQC6U.png)

#### 4.3 定位到节表的地址

然后我们来获得节表的地址：

```cpp
// 读取 ELF 文件
Elf64_Ehdr ehd;
int fp = open("/data/local/tmp/libnative-lib.so", O_RDONLY);

if (fp == -1) {
    __android_log_print(4, "windaa", "%s", "error");
}

// 读取 ELF 文件的文件头
read(fp, &ehd, sizeof(Elf64_Ehdr));

// 读取节表的地址
unsigned long shof = ehd.e_shoff;

// 读取节表的数量
int shnum = ehd.e_shnum;

// 读取每个节表的大小
int shsize = ehd.e_ehsize;

// 记录一下 str 表的偏移，主要是获取后面 GOT 的字符串值
int shstr = ehd.e_shstrndx;
```

我们打印一下此时 `shof` 的值，验证一下节表的地址。

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_VJRHUBSV8PK6TYC.png)

这里可以发现成功读取。

#### 4.4 定位到 GOT 表的位置和函数位置

然后我们拿到字符串的偏移值，定位到 GOT 表，再进一步定位到函数。

```cpp
// 2.拿到字符串表
Elf64_Shdr shdr;

// 定位字符串，节表地址加字符串表偏移 × 节表个数
lseek(fp, shof + shstr * shsize, SEEK_SET);

// 此时节表就定位到字符串表开头
read(fp, &shdr, shsize);

// 分配一个字符串表大小
char *strtable = (char *)malloc(shdr.sh_size);

__android_log_print(6, "windaa", "shdrsize %p", shdr.sh_offset);

// 将字符串指针移动到 0x34104 上
lseek(fp, shdr.sh_offset, SEEK_SET);
read(fp, strtable, shdr.sh_size);

// 将指针移动到节表开头
lseek(fp, shof, SEEK_SET);

// 遍历查找到 GOT
for (int i = 0; i < shnum; i++) {
    // 从节表开头开始读取字符串，每次读取一个节表
    read(fp, &shdr, shsize);

    // 通过节表的索引找到字符串表中对应的值
    if (strcmp(&strtable[shdr.sh_name], ".got") == 0) {
        // 定位到 GOT 表的地址
        int *saddr = start + shdr.sh_addr / 4;

        // 整个 GOT 表的大小
        int size = shdr.sh_size;

        // 遍历 GOT 表中的函数
        for (int j = 0; j < size; j = j + 8) {
            uint64_t value = *(uint64_t *)(saddr + j / 4);

            // 找到 mywind 的地址
            if (reinterpret_cast<uint64_t>(mywin0) == value) {
                __android_log_print(6, "windaa", "value %p", value);

                // 替换 mywind 地址
                // 获取当前内存分页的大小
                uint64_t page_size = getpagesize();

                // 获取内存分页的起始地址（需要内存对齐）
                // page 要保护的是函数的绝对地址，而不是相对地址
                uint64_t entry_page_start =
                    (uint64_t)(saddr + j / 4) & (~(page_size - 1));

                // 修改内存属性为可读可写可执行
                if (mprotect(
                        (uint64_t *)entry_page_start,
                        page_size,
                        PROT_READ | PROT_WRITE | PROT_EXEC
                    ) == -1) {
                    __android_log_print(
                        6,
                        "windaa",
                        "%s",
                        "mprotect failed"
                    );
                }

                value = (uint64_t)mywin1;

                // 将 mywind0 函数的地址换成 mywind1 函数的地址
                memcpy((saddr + j / 4), &value, 16);
            }
        }
    }
}
```

![image-20220517193556517](https://bbs.kanxue.com/upload/attach/202205/905443_TX7ZCATWPAERB55.png)

这里我们就可以发现成功的 hook。

#### GOT hook 使用姿势

1. 使用 `/proc/self/maps` 去获得 so 模块的加载地址。
2. 使用 `ElfHeader` 找到 `Section` 的首地址，并计算 offset 和 size 来获取 `StringTable`。
3. 找到 GOT 表位置，计算其内存位置，并指针指向 GOT 表首地址。
4. 遍历 GOT 表中的函数，找到要 hook 的函数，使用 `mprotect` 进行 hook。
5. 将 hook 的函数地址替换为我们定义的函数地址。

### 5. Unicorn hook 使用

这里我们简单了解一下基于 Unicorn 的框架 Unidbg 的 hook 使用。

开源地址：[43aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6\*7K9r3E0D9x3o6t1J5z5q4)9J5c8Y4g2F1K9h3c8T1k6H3`.`.](https://bbs.kanxue.com/elink@e81K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9r3E0D9x3o6t1J5z5q4)9J5c8Y4g2F1K9h3c8T1k6H3`.`.)

这里我们直接用 IDEA 将项目拉取下来，然后等待项目环境配置完成。

![image-20220517214320334](https://bbs.kanxue.com/upload/attach/202205/905443_KR77QZB8XUD3JQQ.png)

配置完成后，我们直接启动里面的示例代码，查看 hook 效果。

![image-20220517214506632](https://bbs.kanxue.com/upload/attach/202205/905443_JJDDRDSMTYJCX7D.png)

这里 Unidbg 使用了 xHook，xHook 是一种 PLT hook 的方式。当然，这只是 Unidbg 强大功能中的一种，也是 hook 技术中的一种，这里就简单介绍到这，后续再详细讲如何使用。

## 六、实验总结

本文从程序加载的原理出发，讲解了当下常用的一些基本的 hook 方式和手段，后续对其中一些 hook 方式再次深入讲解。实验的一些样本和代码会上传到知识星球和 GitHub。文章参考了很多大佬的文章和大佬星球的内容，参考文献放在末尾。有什么问题，就请各位大佬一一指出了。

GitHub 的地址：[github](https://bbs.kanxue.com/elink@450K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2)

## 参考文献

### 参考书籍

| 序号 | 内容 |
| --- | --- |
| 1 | 参考书目：《程序员的自我修养——链接、装载与库》 |

### GOT 和 PLT
