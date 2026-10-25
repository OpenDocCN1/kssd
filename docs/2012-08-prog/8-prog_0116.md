
# hashmap 实现-不知道是否理解准确

> 原文：[`www.pediy.com/kssd/pediy11/107065.html`](https://www.pediy.com/kssd/pediy11/107065.html)

测试时候要修改下 `sock_alloc_mm` 以及 `sock_free_mm` 宏：

```
#define sock_alloc_mm(size) malloc(size)
#define sock_free_mm(p) free(p)
```

代码:

```
#include "key_object.h"

struct object_hash_item{
         struct object_hash_item* next;          /* 单链表 */
          struct object_hash_item** entry;        /* hash 表入口头 */
         unsigned long key;
         void*   object;
};

struct object_hash_pool{
         int hash_size;
         struct object_hash_item** hash_table;    /* hash 表 */
         int pool_size;                       /* object 数量 */
         struct object_hash_item* object_pool;    /* object 池 */
         int pool_stick;                       /* object 轮转分配位置 */
};

/*
 * 初始化一个 hash_pool, pool_size <=0 表明不使用 object 池，不限制 object 数量
 */
void* alloc_object_hash_pool(int hash_size, int pool_size)
{
         struct object_hash_pool* pool;
         if (hash_size <= 0)
                 return NULL;

         pool = sock_alloc_mm(sizeof(struct object_hash_pool));
         if (!pool)
                 return NULL;

         /* 初始化数据结构 */
         memset(pool, 0, sizeof(struct object_hash_pool));
          
         /* 动态分配 hash 表 */
         pool->hash_size = hash_size;
         pool->hash_table = sock_alloc_mm(hash_size * sizeof(struct object_hash_item*));
         if (!pool->hash_table){
                 free_object_hash_pool(pool);
                 return NULL;
         }

         memset(pool->hash_table, 0, hash_size * sizeof(struct object_hash_item*));
          
         /* 预分配 object 池 */         
         if (pool_size > 0){
                 pool->pool_size = pool_size;
                 pool->object_pool = sock_alloc_mm(pool_size * sizeof(struct object_hash_item));
                 if (!pool->object_pool){
                         free_object_hash_pool(pool);
                         return NULL;
                 }
                 memset(pool->object_pool, 0, pool_size * sizeof(struct object_hash_item));
         }

         return pool;
}

/*
 * 释放 object_hash_pool
 */
void free_object_hash_pool(void* pool_handle)
{
         int i;
         struct object_hash_item* enum_item;
         struct object_hash_item* hash_head;
         struct object_hash_item* hash_item;
         struct object_hash_pool* pool = (struct object_hash_pool*)pool_handle;

         if (pool){
                 /* 释放 object */
                 if (pool->object_pool){
                         sock_free_mm(pool->object_pool);
                 } else if (pool->hash_table) {
                         /* 遍历 hash 表，依次释放 object */
                         for (i = 0; i < pool->hash_size; i++){
                                 hash_head = pool->hash_table[i];
                                 for (enum_item = hash_head; enum_item;){
                                         hash_item = enum_item;
                                         enum_item = hash_item->next;
                                         sock_free_mm(hash_item);
                                 }
                         }
                 }
                 
                 /* 释放 hash 表 */
                 if (pool->hash_table)
                         sock_free_mm(pool->hash_table);

                 memset(pool, 0, sizeof(struct object_hash_pool));
                 sock_free_mm(pool);

         }
}

static struct object_hash_item* key_to_hash_object(struct object_hash_pool* pool, unsigned long key)
{
         int hash_index;
         struct object_hash_item* hash_head;
         struct object_hash_item* enum_item;
          
         if (!pool)
                 return NULL;
          
         /* 计算待插入的 hash 链表头 */
         hash_index = key % pool->hash_size;
         hash_head = pool->hash_table[hash_index];
          
         /* head->自己*/
         for (enum_item = hash_head; enum_item; enum_item = enum_item->next){
                 if (enum_item->key == key)
                         return enum_item;
         }
          
         return NULL;
}

/*
 * 插入一个 key 到 hash 表,key 不允许重复
 */
int add_object_hash_pool(void* pool_handle, unsigned long key, void* object)
{
         int hash_index;
         struct object_hash_item* enum_item;
         struct object_hash_item* hash_item;
         struct object_hash_pool* pool = (struct object_hash_pool*)pool_handle;

         if (!pool)
                 return -1;

         /* 计算待插入的 hash 链表头 */
         hash_index = key % pool->hash_size;

         /* 检查是否 key 冲突 */
         if (key_to_hash_object(pool, key))
                 return -1;

         /* 申请一个 object */
         if (pool->object_pool){
                 pool->pool_stick++;
                 pool->pool_stick = pool->pool_stick % pool->pool_size;
                 hash_item = pool->object_pool + pool->pool_stick;
         } else {
                 hash_item = sock_alloc_mm(sizeof(struct object_hash_item));
                 memset(hash_item, 0, sizeof(struct object_hash_item));
         }

         /* 从原来的 hash 队列中删除 */
         if (hash_item->entry && *hash_item->entry){
                 if (*hash_item->entry == hash_item){
                         *hash_item->entry = hash_item->next;
                 } else {
                         for (enum_item = *hash_item->entry; enum_item; enum_item = enum_item->next){
                                 if (enum_item->next == hash_item){
                                         enum_item->next = hash_item->next;
                                         break;
                                 }
                         }
                 }
         }
         
         hash_item->entry = pool->hash_table + hash_index;
         hash_item->next = NULL;
         hash_item->key = key;
         hash_item->object = object;
         
         if (*hash_item->entry){
                 /* 挂入队列尾 */
                 for (enum_item = *hash_item->entry; enum_item->next; enum_item = enum_item->next);
                 
                 enum_item->next = hash_item;         
         } else {
                 *hash_item->entry = hash_item;
         }

         return 0;
}

/*
 * 通过 key 快速检索到一个 object
 */
void* get_object_hash_pool(void* pool_hanle, unsigned long key)
{
         struct object_hash_item* hash_item = key_to_hash_object((struct object_hash_pool*)pool_hanle, key);
         return hash_item ? hash_item->object : NULL;
}

/*
 * 删除指定 key 对应的 object
 */
int del_object_hash_pool(void* pool_handle, unsigned long key)
{
         struct object_hash_pool* pool = (struct object_hash_pool*)pool_handle;
         struct object_hash_item* enum_item;
         struct object_hash_item* hash_item = key_to_hash_object(pool, key);

         /* 从链表中删除该 object */
         for (enum_item = *hash_item->entry; enum_item && enum_item->next != hash_item;)
                 enum_item = enum_item->next;

         /* 删除该节点 */
         if (enum_item && enum_item->next == hash_item)
                 enum_item->next = hash_item->next;

         if (!pool->object_pool)
                 sock_free_mm(hash_item);

         return 0;
}
```

下面是测试代码

代码:



```c
#include "..\\key\_object.h"
#include <stdlib.h>
#include <stdio.h>

#define TEST\_NUM        1024

int keys\[TEST\_NUM\];

int main(int argc, char\*\* argv)
{
        void\* hash\_pool = alloc\_object\_hash\_pool(TEST\_NUM, 512);
        int i,obj,err\_num = 0, ok\_num = 0, fail\_num = 0;
        
        for (i=0; i<TEST\_NUM; i++){
                keys\[i\] = rand();
                //printf("\[%d\] key:%d\\n", i, keys\[i\]);
                if (add\_object\_hash\_pool(hash\_pool, keys\[i\], keys + i) < 0)
                        keys\[i\] = -1;
        }

        for (i=0; i<TEST\_NUM; i++){
                obj = (int)get\_object\_hash\_pool(hash\_pool, keys\[i\]);
                if ((int)(keys + i) == obj){
                        printf("\[%d\] key:%08x \\tobj:%08x ok\\n", i, keys\[i\], obj);
                        ok\_num++;
                }else if (keys\[i\] == -1){
                        printf("\[%d\] insert fail!\\n", i);
                        fail\_num++;
                }else{
                        printf("\[%d\] key:%08x \\tobj:%08x err!!!%08x\\n", i, keys\[i\], obj, keys + i);
                        err\_num++;
                }
        }

        free\_object\_hash\_pool(hash\_pool);

        printf("Total %d:ok %d fail %d err %d\\n", TEST\_NUM, ok\_num, fail\_num, err\_num);
        return 0;
}
```
