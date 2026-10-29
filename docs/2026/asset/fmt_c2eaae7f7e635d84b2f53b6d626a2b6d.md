
```c
static inline struct page *
__alloc_pages_slowpath(gfp_t gfp_mask, unsigned int order,
struct alloc_context *ac)
{
bool can_direct_reclaim = gfp_mask & __GFP_DIRECT_RECLAIM;
const bool costly_order = order > PAGE_ALLOC_COSTLY_ORDER;
struct page *page = NULL;
unsigned int alloc_flags;
unsigned long did_some_progress;
enum compact_priority compact_priority;
enum compact_result compact_result;
int compaction_retries;
int no_progress_loops;
unsigned int cpuset_mems_cookie;
int reserve_flags;
//如果内存分配来自__GFP_ATOMIC（原子请求）、__GFP_DIRECT_RECLAIM（可直接回收），会产生冲突，取消原子标识
if (WARN_ON_ONCE((gfp_mask & (__GFP_ATOMIC|__GFP_DIRECT_RECLAIM)) == (__GFP_ATOMIC|__GFP_DIRECT_RECLAIM)))
gfp_mask &= ~__GFP_ATOMIC;
retry_cpuset:
compaction_retries = 0;
no_progress_loops = 0;
compact_priority = DEF_COMPACT_PRIORITY;
cpuset_mems_cookie = read_mems_allowed_begin();
//快速分配采用保守的alloc_flags，我们这里进行重新设置，降低成本。
alloc_flags = gfp_to_alloc_flags(gfp_mask);
//重新计算分配迭代zone的起始点。
ac->preferred_zoneref = first_zones_zonelist(ac->zonelist,
ac->highest_zoneidx, ac->nodemask);
if (!ac->preferred_zoneref->zone)
goto nopage;
//如果设置了ALLOC_KSWAPD，唤醒kswapds进程
if (alloc_flags & ALLOC_KSWAPD)
wake_all_kswapds(order, gfp_mask, ac);
//使用重新调整后的信息再次重新分配
page = get_page_from_freelist(gfp_mask, order, alloc_flags, ac);
if (page)
goto got_pg;
/*
* For costly allocations, try direct compaction first, as it's likely
* that we have enough base pages and don't need to reclaim. For non-
* movable high-order allocations, do that as well, as compaction will
* try prevent permanent fragmentation by migrating from blocks of the
* same migratetype.
* Don't try this for allocations that are allowed to ignore
* watermarks, as the ALLOC_NO_WATERMARKS attempt didn't yet happen.
*/
//示情况进行内存压缩
if (can_direct_reclaim &&
(costly_order ||
(order > 0 && ac->migratetype != MIGRATE_MOVABLE)) && !gfp_pfmemalloc_allowed(gfp_mask)) {
page = __alloc_pages_direct_compact(gfp_mask, order,
alloc_flags, ac,
INIT_COMPACT_PRIORITY,
&compact_result);
if (page)
goto got_pg;
//如果设置了__GFP_NORETRY，可能包含了一些THP page fault的分配
if (costly_order && (gfp_mask & __GFP_NORETRY)) {
if (compact_result == COMPACT_SKIPPED ||
compact_result == COMPACT_DEFERRED)
goto nopage;
//同步压缩开销太大，保持异步压缩
compact_priority = INIT_COMPACT_PRIORITY;
}
}
retry:
//保证kswapd不会睡眠，再次唤醒
if (alloc_flags & ALLOC_KSWAPD)
wake_all_kswapds(order, gfp_mask, ac);
//区分真正需要访问全部内存储备的请求和可以承受部分内存的被oom kill掉的请求。
reserve_flags = __gfp_pfmemalloc_flags(gfp_mask);
if (reserve_flags)
alloc_flags = gfp_to_alloc_flags_cma(gfp_mask, reserve_flags);
//当不允许在当前cpu-node中分配，且设置了reserve_flags，那么降低此时的分配标准，重置高优先级的迭代器再进行分配。
if (!(alloc_flags & ALLOC_CPUSET) || reserve_flags) {
ac->nodemask = NULL;
ac->preferred_zoneref = first_zones_zonelist(ac->zonelist,
ac->highest_zoneidx, ac->nodemask);
}
/* Attempt with potentially adjusted zonelist and alloc_flags */
page = get_page_from_freelist(gfp_mask, order, alloc_flags, ac);
if (page)
goto got_pg;
/* Caller is not willing to reclaim, we can't balance anything */
if (!can_direct_reclaim)
goto nopage;
/* Avoid recursion of direct reclaim */
if (current->flags & PF_MEMALLOC)
goto nopage;
//尝试先回收，再分配
page = __alloc_pages_direct_reclaim(gfp_mask, order, alloc_flags, ac,
&did_some_progress);
if (page)
goto got_pg;
//尝试直接压缩，再分配
page = __alloc_pages_direct_compact(gfp_mask, order, alloc_flags, ac,
compact_priority, &compact_result);
if (page)
goto got_pg;
/* Do not loop if specifically requested */
if (gfp_mask & __GFP_NORETRY)
goto nopage;
/*
* Do not retry costly high order allocations unless they are
* __GFP_RETRY_MAYFAIL
*/
if (costly_order && !(gfp_mask & __GFP_RETRY_MAYFAIL))
goto nopage;
//是否应当再次进行内存回收
if (should_reclaim_retry(gfp_mask, order, ac, alloc_flags,
did_some_progress > 0, &no_progress_loops))
goto retry;
//是否应该再次压缩
if (did_some_progress > 0 &&
should_compact_retry(ac, order, alloc_flags,
compact_result, &compact_priority,
&compaction_retries))
goto retry;
//在我们启动oom之前判断可能的条件竞争问题
if (check_retry_cpuset(cpuset_mems_cookie, ac))
goto retry_cpuset;
//回收失败，开启oomkiller，杀死一些进程以获得内存
page = __alloc_pages_may_oom(gfp_mask, order, ac, &did_some_progress);
if (page)
goto got_pg;
//避免nowatermark的内存无限使用
if (tsk_is_oom_victim(current) &&
(alloc_flags & ALLOC_OOM ||
(gfp_mask & __GFP_NOMEMALLOC)))
goto nopage;
if (did_some_progress) {
no_progress_loops = 0;
goto retry;
}
nopage:
if (check_retry_cpuset(cpuset_mems_cookie, ac))
goto retry_cpuset;
//当设置了__GFP_NOFAIL时，多次尝试
if (gfp_mask & __GFP_NOFAIL) {
//当所有的NOFAIL的请求都被blocked掉时，警告用户此时应该使用NOWAIT
if (WARN_ON_ONCE(!can_direct_reclaim))
goto fail;
WARN_ON_ONCE(current->flags & PF_MEMALLOC);
WARN_ON_ONCE(order > PAGE_ALLOC_COSTLY_ORDER);
/*
通过让他们访问内存储备来帮助不失败的分配，但不要使用ALLOC_NO_WATERMARKS，因为这可能耗尽整个内存储备，使情况变得更糟
*/
page = __alloc_pages_cpuset_fallback(gfp_mask, order, ALLOC_HARDER, ac);
if (page)
goto got_pg;
cond_resched();
goto retry;
}
fail:
warn_alloc(gfp_mask, ac->nodemask,
"page allocation failure: order:%u", order);
got_pg:
return page;
}
```

### `__free_pages`
```c
void __free_pages(struct page *page, unsigned int order)
{
//检查页框是否还有进程在使用，检查_count变量的值是否为0
if (put_page_testzero(page))
free_the_page(page, order);
//这里我个人理解时，类比于之前的set_page_guard那一步，分配的order大于需要的order，相当于分配了多页，那么这里就是挨个释放多页
else if (!PageHead(page))
while (order-- > 0)
free_the_page(page + (1 << order), order);
}
```

在 `free_the_page` 中：
```c
static inline void free_the_page(struct page *page, unsigned int order)
{
//如果是通过pcpulist分配
if (order == 0) /* Via pcp? */
free_unref_page(page);
//否则调用__free_pages_ok
else
__free_pages_ok(page, order, FPI_NONE);
}
```
```c
/*
* Free a 0-order page
*/
void free_unref_page(struct page *page)
{
unsigned long flags;
//获取page frame number
unsigned long pfn = page_to_pfn(page);
//进行free前检查
if (!free_unref_page_prepare(page, pfn))
return;
local_irq_save(flags);
free_unref_page_commit(page, pfn);
local_irq_restore(flags);
}
```

** `free_unref_page_commit` **
```c
static void free_unref_page_commit(struct page *page, unsigned long pfn)
{
struct zone *zone = page_zone(page);
struct per_cpu_pages *pcp;
int migratetype;
//获取当前page的迁移类型
migratetype = get_pcppage_migratetype(page);
__count_vm_event(PGFREE);
//percpu list只放入几种制定类型的page
if (migratetype >= MIGRATE_PCPTYPES) {
if (unlikely(is_migrate_isolate(migratetype))) {
//free
free_one_page(zone, page, pfn, 0, migratetype,
FPI_NONE);
return;
}
migratetype = MIGRATE_MOVABLE;
}
pcp = &this_cpu_ptr(zone->pageset)->pcp;
//将page用头插法放入pcp->lists[migratetype]链表头
list_add(&page->lru, &pcp->lists[migratetype]);
pcp->count++;
//如果pcp中的page数量大于最大数量，则将多余的page放入伙伴系统
if (pcp->count >= READ_ONCE(pcp->high))
free_pcppages_bulk(zone, READ_ONCE(pcp->batch), pcp);
}
```

** `free_one_page --> __free_one_page` **
```c
static inline void __free_one_page(struct page *page,
unsigned long pfn,
struct zone *zone, unsigned int order,
int migratetype, fpi_t fpi_flags)
{
struct capture_control *capc = task_capc(zone);
unsigned long buddy_pfn;
unsigned long combined_pfn;
unsigned int max_order;
struct page *buddy;
bool to_tail;
//获取最大order-1
max_order = min_t(unsigned int, MAX_ORDER - 1, pageblock_order);
VM_BUG_ON(!zone_is_initialized(zone));
VM_BUG_ON_PAGE(page->flags & PAGE_FLAGS_CHECK_AT_PREP, page);
VM_BUG_ON(migratetype == -1);
if (likely(!is_migrate_isolate(migratetype)))
__mod_zone_freepage_state(zone, 1 << order, migratetype);
VM_BUG_ON_PAGE(pfn & ((1 << order) - 1), page);
VM_BUG_ON_PAGE(bad_range(zone, page), page);
continue_merging:
// 循环扫描直到order == max_order-1
// 处理合并问题
```
