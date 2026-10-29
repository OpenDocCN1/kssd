
该函数会检查是否移动单页，如果是直接调用 `move_to_free_list`：

```c
static inline void move_to_free_list(struct page *page, struct zone *zone,
unsigned int order, int migratetype)
{
struct free_area *area = &zone->free_area[order];

list_move_tail(&page->lru, &area->free_list[migratetype]);
}
```

`list_move_tail` 相关定义如下：

```c
/*
 * Insert a new entry between two known consecutive entries.
 *
 * This is only for internal list manipulation where we know
 * the prev/next entries already!
 */
static inline void __list_add(struct list_head *new,
struct list_head *prev,
struct list_head *next)
{
if (!__list_add_valid(new, prev, next))
return;

next->prev = new;
new->next = next;
new->prev = prev;
WRITE_ONCE(prev->next, new);
}

/*
 * list_add_tail - add a new entry
 * @new: new entry to be added
 * @head: list head to add it before
 *
 * Insert a new entry before the specified head.
 * This is useful for implementing queues.
 */
static inline void list_add_tail(struct list_head *new, struct list_head *head)
{
__list_add(new, head->prev, head);
}

/*
 * list_move_tail - delete from one list and add as another's tail
 * @list: the entry to move
 * @head: the head that will follow our entry
 */
static inline void list_move_tail(struct list_head *list,
struct list_head *head)
{
__list_del_entry(list);
list_add_tail(list, head);
}
```

如果是移动 Block，则调用 `move_freepages_block`：

```c
int move_freepages_block(struct zone *zone, struct page *page,
int migratetype, int *num_movable)
{
unsigned long start_pfn, end_pfn;
struct page *start_page, *end_page;

if (num_movable)
*num_movable = 0;

start_pfn = page_to_pfn(page);
start_pfn = start_pfn & ~(pageblock_nr_pages-1);
start_page = pfn_to_page(start_pfn);
end_page = start_page + pageblock_nr_pages - 1;
end_pfn = start_pfn + pageblock_nr_pages - 1;

/* Do not cross zone boundaries */
if (!zone_spans_pfn(zone, start_pfn))
start_page = page;
if (!zone_spans_pfn(zone, end_pfn))
return 0;

return move_freepages(zone, start_page, end_page, migratetype,
num_movable);
}
```

该函数计算完起始与终止 Page，PFN 之后，调用 `move_freepages` 函数进行移动：
