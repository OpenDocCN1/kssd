
```c
#define mi_track_page(page,access)  { size_t psize; void* pstart = _mi_page_start(_mi_page_segment(page),page,&psize); mi_track_mem_##access( pstart, psize); }
```

```c
static inline bool mi_check_is_double_free(const mi_page_t* page, const mi_block_t* block) {

  bool is_double_free = false;

  mi_block_t* n = mi_block_nextx(page, block, page->keys); // pretend it is freed, and get the decoded first field

  if (((uintptr_t)n & (MI_INTPTR_SIZE-1))==0 &&  // quick check: aligned pointer?

      (n==NULL || mi_is_in_same_page(block, n))) // quick check: in same page or NULL?

  {

    // Suspicous: decoded value a in block is in the same page (or NULL) -- maybe a double free?

    // (continue in separate function to improve code generation)

    is_double_free = mi_check_is_double_freex(page, block);

  }

  return is_double_free;

}

#else

static inline bool mi_check_is_double_free(const mi_page_t* page, const mi_block_t* block) {

  MI_UNUSED(page);

  MI_UNUSED(block);

  return false;

}
```

通过`mi_block_nextx`函数来获取内存块的`next`成员 如果为0或者和内存块位于同一个page 那么就会进入`mi_check_is_double_freex`函数进行更加详细的check 如果不开启debug模式的话 不会进行double free检测

## `mi_check_is_double_freex`

```c
static mi_decl_noinline bool mi_check_is_double_freex(const mi_page_t* page, const mi_block_t* block) {
  // The decoded value is in the same page (or NULL).
  // Walk the free lists to verify positively if it is already freed
  if (mi_list_contains(page, page->free, block) ||
      mi_list_contains(page, page->local_free, block) ||
      mi_list_contains(page, mi_page_thread_free(page), block))
  {
    _mi_error_message(EAGAIN, "double free detected of block %p with size %zu\n", block, mi_page_block_size(page));
    return true;
  }
  return false;
}
```

通过遍历page的三个链表来查找内存块是否已经被释放过了

相对来说比较好绕过 只要更改链表头的`next`成员 就可以让链表索引不到已经被释放过的内存块

## `_mi_free_generic`

```c
void mi_decl_noinline _mi_free_generic(const mi_segment_t* segment, mi_page_t* page, bool is_local, void* p) mi_attr_noexcept {
  mi_block_t* const block = (mi_page_has_aligned(page) ? _mi_page_ptr_unalign(segment, page, p) : (mi_block_t*)p);
  mi_stat_free(page, block); // stat_free may access the padding
  mi_track_free_size(block, mi_page_usable_size_of(page,block));
  _mi_free_block(page, is_local, block);
}
```

开始先进行了内存对齐的检测 随后就是老一套的内存块信息的记录

随后调用`_mi_free_block`函数释放内存块

## `_mi_free_block`

```c
static inline void _mi_free_block(mi_page_t* page, bool local, mi_block_t* block)
{
  // and push it on the free list
  //const size_t bsize = mi_page_block_size(page);
  if mi_likely(local) {
    // owning thread can free a block directly
    if mi_unlikely(mi_check_is_double_free(page, block)) return;
    mi_check_padding(page, block);
#if (MI_DEBUG>0) && !MI_TRACK_ENABLED && !MI_TSAN
    if (!mi_page_is_huge(page)) { // huge page content may be already decommitted
      memset(block, MI_DEBUG_FREED, mi_page_block_size(page));
    }
#endif
    mi_block_set_next(page, block, page->local_free);
    page->local_free = block;
    page->used--;
    if mi_unlikely(mi_page_all_free(page)) {
      _mi_page_retire(page);
    }
    else if mi_unlikely(mi_page_is_in_full(page)) {
      _mi_page_unfull(page);
    }
  }
  else {
    _mi_free_block_mt(page,block);
  }
}
```

仍然是先进行了内存越界和double free的检测 同时根据是否开启了debug模式 来决定是否要调用`memset`函数清空内存块的内容

随后就是更新`local_free`链表和`used`的值 接着检查是否要释放整个page 或者是当前page是否已经成为满页 如果是满页则从满页列表中移除

## 释放

总结一下释放 实际上就是先进行没啥软用的double free检查 然后根据要释放的内存块是否就是整个page 来决定要不要用`_mi_free_generic`函数来释放

释放完的话 是加入到`local_free`链表 并且`used`的值会减少 相对来说逻辑还是比较简单的

## 实例利用

### 泄露libc基址和任意写

在泄露libc基址上同ptmalloc不一样 因为供用户申请的内存块就算被释放后 也不会根据大小进入bin中 从而在`fd`域或者`bk`域写入libc地址 我们在获得一个内存块后 只能获得其`next`域的下一个内存块的地址

![image-20230802153413131](https://bbs.kanxue.com/upload/attach/202308/983628_TUZ8D3PHA879EUX.jpg)

而在获得内存块后 我们就可以计算得到page的地址

![image-20230802153526770](https://bbs.kanxue.com/upload/attach/202308/983628_RZDNMNW4497WHYW.jpg)

page中就存放着libc地址 经过偏移计算即可得到libc基址

问题在于如何构造任意写 经过上面的源码分析我们可以得知 内存块的申请是优先从`free`链表中获取的 如果我们拥有溢出的机会 修改下一个内存块的`next`域 是不是就可以实现任意地址申请

同时还需要注意一点 由于mimalloc内存地址的特殊性 如果我们申请的内存块过小 就会导致低字节处`\x00`截断输出 进而妨碍我们泄露内存块地址

![image-20230802153814311](https://bbs.kanxue.com/upload/attach/202308/983628_GV4XV7TSGHEQ769.jpg)

![image-20230802154451205](https://bbs.kanxue.com/upload/attach/202308/983628_52DA7933ZQSJM63.jpg)

可以看到此时`free`链表中已经写入了我们想要用来泄露libc地址的fake chunk

此时我们再次申请出同样size的两个内存块 打印第二个内存块的内容 就可以得到`_mi_heap_main`的地址 从而泄露libc基址

同时要注意一下 libc基址和`libmimalloc`基址是不一样的

#### 2.34以上的libc版本

由于2.34以上的版本的tls结构体的偏移进行了随机化 所以还需要进行爆破一个字节才能得到正确的libc的基址

### getshell

回顾一下`mi_malloc_generic`函数

```c
// call potential deferred free routines
_mi_deferred_free(heap, false);
```

会发现其调用了这个函数 当时我的分析是将标记释放的内存块加入到延迟释放链表中

而在ida的汇编代码中你可以看到 其是通过`call`指令来调用的

![image-20230802160433594](https://bbs.kanxue.com/upload/attach/202308/983628_ZJJ6JWHQQMRG272.jpg)

偏偏这个地址还是可写的 所以我们可以通过往这个地址写入`system`函数 从而进行任意函数调用

![image-20230802160454759](https://bbs.kanxue.com/upload/attach/202308/983628_2CWFXA4VYFTTHYS.jpg)

但是很快你会发现我们并没有办法控制`rdi`寄存器

![image-20230802172730420](https://bbs.kanxue.com/upload/attach/202308/983628_VFTQSFEU9D4F2UH.jpg)

但是可以控制`rdx`寄存器 联想ptmalloc我们是如何实现的orw 就会想到`setcontext`这一手法

### 调试题目和脚本

需要的可以自行下载或者编译

链接: [d22K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4f1U0k6B7K9q4y4m8L8$3b7@1k6K6S2d9f1g2A6f1z5i4c8c8f1$3N6Q4y4h3k6Y4i4K6y4r3M7s2N6V1i4K6y4p5k6o6p5&6x3H3`.`.](https://bbs.kanxue.com/elink@07eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4f1U0k6B7K9q4y4m8L8$3b7@1k6K6S2d9f1g2A6f1z5i4c8c8f1$3N6Q4y4h3k6Y4i4K6y4r3M7s2N6V1i4K6y4p5k6o6p5&6x3H3`.`.) 提取码: d193

```c
#include<mimalloc.h>
#include<stdio.h>
#include <stdlib.h>

char *chunk_ptr[0x20];
int chunk_size[0x20];
int count=0;

void init(){
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
}

void menu(){
    puts("1.mi_malloc");
    puts("2.mi_free");
    puts("3.edit");
    puts("4.show");
    puts("5.exit");
    printf(">> ");
}

int my_read(){
    char buf[0x8];
    read(0,buf,0x8);
    return atoi(buf);
}

void add(){
    int size;
    puts("Size :");
    size = my_read();
    chunk_size[count] = size;
    chunk_ptr[count] = mi_malloc(size);
    count ++;
}

void delete(){
    puts("Index :");
    int index = my_read();
    mi_free(chunk_ptr[index]);
}

void edit(){
    puts("Index :");
    int index = my_read();
    puts("Size :");
    int size = my_read();
    read(0,chunk_ptr[index],size);
}

void show(){
    puts("Index :");
    int index = my_read();
    puts(chunk_ptr[index]);
}

int main(){
    init();
    while(1){
        menu();
        int choice = my_read();
        switch(choice){
            case 1:
                add();
                break;
            case 2:
                delete();
                break;
            case 3:
                edit();
                break;
            case 4:
                show();
                break;
            case 5:
                exit(0);
            default:
                puts("Unknown option");
                break;
        }
    }
}
```
