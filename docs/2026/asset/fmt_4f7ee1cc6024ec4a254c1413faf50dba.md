
```
/*
 * This is the 'heart' of the zoned buddy allocator.
 */
struct page *__alloc_pages_nodemask(gfp_t gfp_mask, unsigned int order, int preferred_nid, nodemask_t *nodemask)
{
    struct page *page;
    unsigned int alloc_flags = ALLOC_WMARK_LOW;
    gfp_t alloc_mask; /* The gfp_t that was actually used for allocation */
    struct alloc_context ac = { };

    /*
     * There are several places where we assume that the order value is sane
     * so bail out early if the request is out of bound.
     */
    if (unlikely(order >= MAX_ORDER)) {
        WARN_ON_ONCE(!(gfp_mask & __GFP_NOWARN));
        return NULL;
    }

    gfp_mask &= gfp_allowed_mask;
    alloc_mask = gfp_mask;
    if (!prepare_alloc_pages(gfp_mask, order, preferred_nid, nodemask, &ac, &alloc_mask, &alloc_flags))
        return NULL;

    /*
     * Forbid the first pass from falling back to types that fragment
     * memory until all local zones are considered.
     */
    alloc_flags |= alloc_flags_nofragment(ac.preferred_zoneref->zone, gfp_mask);

    /* First allocation attempt */
    page = get_page_from_freelist(alloc_mask, order, alloc_flags, &ac);
    if (likely(page))
        goto out;

    /*
     * Apply scoped allocation constraints. This is mainly about GFP_NOFS
     * resp. GFP_NOIO which has to be inherited for all allocation requests
     * from a particular context which has been marked by
     * memalloc_no{fs,io}_{save,restore}.
     */
    alloc_mask = current_gfp_context(gfp_mask);
    ac.spread_dirty_pages = false;

    /*
     * Restore the original nodemask if it was potentially replaced with
     * &cpuset_current_mems_allowed to optimize the fast-path attempt.
     */
    ac.nodemask = nodemask;

    page = __alloc_pages_slowpath(alloc_mask, order, &ac);

out:
    if (memcg_kmem_enabled() && (gfp_mask & __GFP_ACCOUNT) && page &&
        unlikely(__memcg_kmem_charge_page(page, gfp_mask, order) != 0)) {
        __free_pages(page, order);
        page = NULL;
    }

    trace_mm_page_alloc(page, order, alloc_mask, ac.migratetype);

    return page;
}
EXPORT_SYMBOL(__alloc_pages_nodemask);
```

首先是判断 `order` 大小是否超过 `MAX_ORDER`：

```
if (unlikely(order >= MAX_ORDER)) {
    WARN_ON_ONCE(!(gfp_mask & __GFP_NOWARN));
    return NULL;
}
```

#### a. `prepare_alloc_pages`

之后执行 `prepare_alloc_pages`，该函数主要是初始化 `ac` 变量——类型为 `alloc_context` 结构体（该结构及其字段含义见注释，不再赘述）：

```
/*
 * Structure for holding the mostly immutable allocation parameters passed
 * between functions involved in allocations, including the alloc_pages*
 * family of functions.
 *
 * nodemask, migratetype and highest_zoneidx are initialized only once in
 * __alloc_pages_nodemask() and then never change.
 *
 * zonelist, preferred_zone and highest_zoneidx are set first in
 * __alloc_pages_nodemask() for the fast path, and might be later changed
 * in __alloc_pages_slowpath(). All other functions pass the whole structure
 * by a const pointer.
 */
struct alloc_context {
    struct zonelist *zonelist;
    nodemask_t *nodemask;
    struct zoneref *preferred_zoneref;
    int migratetype;

    /*
     * highest_zoneidx represents highest usable zone index of
     * the allocation request. Due to the nature of the zone,
     * memory on lower zone than the highest_zoneidx will be
     * protected by lowmem_reserve[highest_zoneidx].
     *
     * highest_zoneidx is also used by reclaim/compaction to limit
     * the target zone since higher zone than this index cannot be
     * usable for this allocation request.
     */
    enum zone_type highest_zoneidx;
    bool spread_dirty_pages;
};
```

函数执行操作如下：

```
static inline bool prepare_alloc_pages(gfp_t gfp_mask, unsigned int order,
                  int preferred_nid, nodemask_t *nodemask,
                  struct alloc_context *ac, gfp_t *alloc_mask,
                  unsigned int *alloc_flags)
{
    ac->highest_zoneidx = gfp_zone(gfp_mask);
    ac->zonelist = node_zonelist(preferred_nid, gfp_mask);
    ac->nodemask = nodemask;
    ac->migratetype = gfp_migratetype(gfp_mask);

    if (cpusets_enabled()) {
        *alloc_mask |= __GFP_HARDWALL;
        /*
         * When we are in the interrupt context, it is irrelevant
         * to the current task context. It means that any node ok.
         */
        if (!in_interrupt() && !ac->nodemask)
            ac->nodemask = &cpuset_current_mems_allowed;
        else
            *alloc_flags |= ALLOC_CPUSET;
    }

    fs_reclaim_acquire(gfp_mask);
    fs_reclaim_release(gfp_mask);

    might_sleep_if(gfp_mask & __GFP_DIRECT_RECLAIM);

    if (should_fail_alloc_page(gfp_mask, order))
        return false;

    *alloc_flags = current_alloc_flags(gfp_mask, *alloc_flags);

    /* Dirty zone balancing only done in the fast path */
    ac->spread_dirty_pages = (gfp_mask & __GFP_WRITE);

    /*
     * The preferred zone is used for statistics but crucially it is
     * also used as the starting point for the zonelist iterator. It
     * may get reset for allocations that ignore memory policies.
     */
    ac->preferred_zoneref = first_zones_zonelist(ac->zonelist,
                    ac->highest_zoneidx, ac->nodemask);

    return true;
}
```

其中 `gfp_zone` 函数根据 `gfp_mask` 计算出 Zone（该函数返回值类型定义见 0x01.2 节）：

```
static inline enum zone_type gfp_zone(gfp_t flags)
{
    enum zone_type z;
    int bit = (__force int) (flags & GFP_ZONEMASK);

    z = (GFP_ZONE_TABLE >> (bit * GFP_ZONES_SHIFT)) &
        ((1 << GFP_ZONES_SHIFT) - 1);
    VM_BUG_ON((GFP_ZONE_BAD >> bit) & 1);
    return z;
}
```

其中 `GFP_ZONEMASK` 定义如下——即 0x0F（`gfp_mask` 低四位表示进行分配的 Zone）：

```
#define ___GFP_DMA        0x01u
#define ___GFP_HIGHMEM        0x02u
#define ___GFP_DMA32        0x04u
#define ___GFP_MOVABLE        0x08u
......
#define __GFP_DMA    ((__force gfp_t)___GFP_DMA)
#define __GFP_HIGHMEM    ((__force gfp_t)___GFP_HIGHMEM)
#define __GFP_DMA32    ((__force gfp_t)___GFP_DMA32)
#define __GFP_MOVABLE    ((__force gfp_t)___GFP_MOVABLE)  /* ZONE_MOVABLE allowed */
#define GFP_ZONEMASK    (__GFP_DMA|__GFP_HIGHMEM|__GFP_DMA32|__GFP_MOVABLE)
```

不同位置 1 结果如下：
