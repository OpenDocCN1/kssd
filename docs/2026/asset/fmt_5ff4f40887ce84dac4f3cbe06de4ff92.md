
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

typedef unsigned char   _u8;
typedef unsigned int    _u32;
typedef unsigned short  _u16;

#define SAFE_FREE(p)    {if(p) free(p);}

struct list_item {
        _u32 key;
        _u16 pre_ok;
        _u16 total;
};

struct btb_field{
        _u32 addr;              /* 分支指令地址 */
        _u32 dst_addr;          /* 目标地址 */
        int state;              /* 状态-n计数器 */
};

/*
 * 相关预测器(m,n)预测器
 */
struct branch_predictor{
        _u32 history;           /* 最近分支历史(循环移位) */
        _u32 m;                 /* 最近跳转指令数 */
        _u32 n;                 /* n bit跳转计数器 */
        _u32 btb_size;          /* btb表大小 */

        struct list_item* addr_list;
        _u32 addr_num;
        _u32 addr_list_size;

        _u32 hit_count;
        _u32 total_count;
        _u32 capacity_miss_count;
        _u32 miss_count;

        int max_btb_state;

        struct btb_field* btb;
};

/* 返回插入位置 */
int get_addr_index(struct branch_predictor* predictor, void* addr)
{
        int lo = 0;
        int hi = predictor->addr_num - 1;
        int mid = (lo + hi) / 2;
        
        if (predictor->addr_num < 1)
                return -1;
        if ((_u32)addr <= predictor->addr_list[lo].key)
                return lo;
        if ((_u32)addr >= predictor->addr_list[hi].key)
                return hi;
        
        while (hi > lo + 1) {
                mid = (lo + hi) / 2;
                if ((_u32)addr > predictor->addr_list[mid].key)
                        lo = mid;
                else if ((_u32)addr < predictor->addr_list[mid].key)
                        hi = mid;
                else
                        return mid;
        }
        
        return mid;
}

struct list_item* add_addr_list(struct branch_predictor* predictor, void* addr)
{
        int index;
        
        if (!predictor->addr_list || predictor->addr_num < 1
                || predictor->addr_num >= predictor->addr_list_size) {
                predictor->addr_list_size += 0x400;
                predictor->addr_list = realloc(predictor->addr_list, predictor->addr_list_size * sizeof(struct list_item));
        }
        
        if (predictor->addr_num < 1) {
                memset(predictor->addr_list, 0, sizeof(struct list_item));
                predictor->addr_list[0].key = (_u32)addr;
                predictor->addr_num = 1;
                return predictor->addr_list;
        }
        
        index = get_addr_index(predictor, addr);
        
        if ((_u32)addr == predictor->addr_list[index].key){
                //重复添加
                return predictor->addr_list + index;
        } else if ((_u32)addr > predictor->addr_list[index].key)
                index += 1;
        
        memmove(predictor->addr_list + index + 1, 
                predictor->addr_list + index, 
                sizeof(struct list_item)*(predictor->addr_num - index));
        
        memset(predictor->addr_list + index, 0, sizeof(struct list_item));
        predictor->addr_list[index].key = (_u32)addr;
        predictor->addr_num++;
        
        return predictor->addr_list + index;
}

struct branch_predictor* init_predictor(int m, int n, int btb_size)
{
        struct branch_predictor* predictor = malloc(sizeof(struct branch_predictor));
        
        memset(predictor, 0, sizeof(*predictor));

        predictor->m = m;
        predictor->n = n;
        predictor->max_btb_state = 1<<n;
        predictor->btb_size = btb_size;
        predictor->btb = malloc(btb_size*(1<<m)*sizeof(struct btb_field));

        return predictor;
}

void free_predictor(struct branch_predictor* predictor)
{
        if (!predictor)
                return;

        SAFE_FREE(predictor->btb);
        SAFE_FREE(predictor->addr_list);
        memset(predictor, 0, sizeof(*predictor));
        SAFE_FREE(predictor);
}

void reset_predictor(struct branch_predictor* predictor)
{
        predictor->capacity_miss_count = 0;
        predictor->history = 0;
        predictor->hit_count = 0;
        predictor->miss_count = 0;
        predictor->total_count = 0;
        predictor->addr_num = 0;
}

void predictor_run(struct branch_predictor* predictor, _u32 addr, _u32 dst_addr, int is_jmp)
{
        struct btb_field* btb_item;
        struct list_item* addr_item;
        int index;
        int pre_jmp;

        /* 根据m 历史记录-计算base */
        index = predictor->history * predictor->btb_size;
        index += addr % predictor->btb_size;
        btb_item = predictor->btb + index;

        /* 统计每个地址情况 */
        addr_item = add_addr_list(predictor, (void*)addr);
        addr_item->total++;

        if (btb_item->addr == addr){
                pre_jmp = (btb_item->state >= predictor->max_btb_state/2)? 1:0;

                /* 更新btb状态计数器 0 - predictor->max_btb_state-1 */
                btb_item->state += is_jmp? 1:-1;
                if (btb_item->state > predictor->max_btb_state-1)
                        btb_item->state = predictor->max_btb_state-1;
                else if (btb_item->state < 0)
                        btb_item->state = 0;
        } else {
                /* 静态预测器-向下跳转预测为不跳转，向上跳转预测为跳转 */
                pre_jmp = (dst_addr < addr)? 1:0;
                /* 初始化btb项目 */
                btb_item->addr = addr;
                btb_item->dst_addr = dst_addr;
                btb_item->state = 1;
                /* 统计缺失情况 */
                predictor->miss_count++;
                if (addr_item->total > 1)
                        predictor->capacity_miss_count++;
        }

        if (pre_jmp == is_jmp){
                predictor->hit_count++;
                addr_item->pre_ok++;
        }
        
        predictor->total_count++;

        /* 更新历史记录 */
        predictor->history = predictor->history << 1;
        predictor->history |= (is_jmp ? 1:0);
        predictor->history &= (1<<predictor->m) - 1;
}

static void load_test_case(struct branch_predictor* predictor, char* filename)
{
        char buf[128];
        FILE* fp = fopen((const char*)filename, "r");

        while (fgets(buf, sizeof(buf), fp)){
                _u32 addr, dst_addr, is_jmp;
                sscanf(buf, "%x%x%d", &addr, &dst_addr, &is_jmp);
                //printf("%s %p %p %d\n", buf, addr, dst_addr, is_jmp);
                predictor_run(predictor, addr, dst_addr, is_jmp);
        }

        fclose(fp);
}

void show_result(struct branch_predictor* predictor)
{
        struct list_item* max_item;
        int i;
        int max_addr_id = 0;
        int max_addr_num = 1;
        printf("命中:%d/%d/%.2f%%\t错误:%d %.2f%%\n缺失:%4d%6.2f%%\t容量缺失:%d\n", 
                predictor->hit_count,
                predictor->total_count,
                100.0*predictor->hit_count/predictor->total_count,
                predictor->total_count-predictor->hit_count,
                100.0*(predictor->total_count-predictor->hit_count)/predictor->total_count,
                predictor->miss_count,
                100.0*predictor->miss_count/predictor->total_count,
                predictor->capacity_miss_count
                );
        
        /* 统计最多的指令数以及命中情况 */
        for (i=0; i < (int)predictor->addr_num; i++){
                if (predictor->addr_list[i].total > max_addr_num){
                        max_addr_num = predictor->addr_list[i].total;
                        max_addr_id = i;
                }
        }
        
        max_item = predictor->addr_list + max_addr_id;
        printf("最多跳转地址:%p 命中情况:%d/%d/%d/%.2f%%\n\n", 
                max_item->key, max_item->pre_ok, max_item->total,
                predictor->hit_count,
                100.0*max_item->pre_ok/predictor->hit_count);
}

int main(int argc, char** argv)
{
        int i;
        int btb_size = 1;
        struct branch_predictor* predictor;

        for (i=0; i<10; i++){
                predictor = init_predictor(0, 2, btb_size);
                load_test_case(predictor, "history.txt");
                printf("冷启动 BTB大小:%d\n", btb_size);
                show_result(predictor);
                free_predictor(predictor);
                btb_size *= 2;
        }
        
        /* warm up */
        btb_size = 1;
        for (i=0; i<10; i++){
                predictor = init_predictor(0, 2, btb_size);
                /* 首先冷启动 */
                load_test_case(predictor, "history.txt");
                reset_predictor(predictor);
                /* 热启动 */
                load_test_case(predictor, "history.txt");
                printf("热启动 BTB大小:%d\n", btb_size);
                show_result(predictor);
                free_predictor(predictor);
                
                btb_size *= 2;
        }

        return 0;
}
```
