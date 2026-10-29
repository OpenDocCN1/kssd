
# [原创]利用xhook安卓系统底层抹机原理

先上github地址[b54K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2Z5L8$3!0C8](https://bbs.kanxue.com/elink@2efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2Z5L8$3!0C8)

大概原理是：

- 先读取`/proc/self/maps`文件内容
- 正则匹配找到`so`文件路径和加载基址
- 解析`elf`格式找到要`hook`的函数的地址替换成自己指定的函数地址

```
//xh_core.c

static void xh_core_refresh_impl()    

{    

char                     line[512];    

FILE                    *fp;    

uintptr_t                base_addr;    

char                     perm[5];    

unsigned long            offset;    

int                      pathname_pos;    

char                    *pathname;    

size_t                   pathname_len;    

xh_core_map_info_t      *mi, *mi_tmp;    

xh_core_map_info_t       mi_key;    

xh_core_hook_info_t     *hi;    

xh_core_ignore_info_t   *ii;    

int                      match;    

xh_core_map_info_tree_t  map_info_refreshed = RB_INITIALIZER(&map_info_refreshed);    

if(NULL == (fp = fopen("/proc/self/maps", "r")))    

{    

XH_LOG_ERROR("fopen /proc/self/maps failed");    

return;    

}    

while(fgets(line, sizeof(line), fp))    

{    

if(sscanf(line, "%"PRIxPTR"-%*lx %4s %lx %*x:%*x %*d%n", &base_addr, perm, &offset, &pathname_pos) != 3) continue;    

//check permission    

if(perm[0] != 'r') continue;    

if(perm[3] != 'p') continue; //do not touch the shared memory    

//check offset    

//    

//We are trying to find ELF header in memory.    

//It can only be found at the beginning of a mapped memory regions    

//whose offset is 0.    

if(0 != offset) continue;    

//get pathname    

while(isspace(line[pathname_pos]) && pathname_pos < (int)(sizeof(line) - 1))    

pathname_pos += 1;    

if(pathname_pos >= (int)(sizeof(line) - 1)) continue;    

pathname = line + pathname_pos;    

pathname_len = strlen(pathname);    

if(0 == pathname_len) continue;    

if(pathname[pathname_len - 1] == '\n')    

{    

pathname[pathname_len - 1] = '\0';    

pathname_len -= 1;    

}    

if(0 == pathname_len) continue;    

if('[' == pathname[0]) continue;    

//check pathname    

//if we need to hook this elf?    

match = 0;    

TAILQ_FOREACH(hi, &xh_core_hook_info, link) //find hook info    

{    

if(0 == regexec(&(hi->pathname_regex), pathname, 0, NULL, 0))    

{    

TAILQ_FOREACH(ii, &xh_core_ignore_info, link) //find ignore info    

{    

if(0 == regexec(&(ii->pathname_regex), pathname, 0, NULL, 0))    

{    

if(NULL == ii->symbol)    

goto check_finished;    

if(0 == strcmp(ii->symbol, hi->symbol))    

goto check_continue;    

}    

}    

match = 1;    

check_continue:    

break;    

}    

}    

check_finished:    

if(0 == match) continue;    

//check elf header format    

//We are trying to do ELF header checking as late as possible.    

if(0 != xh_core_check_elf_header(base_addr, pathname)) continue;    

//check existed map item    

mi_key.pathname = pathname;    

if(NULL != (mi = RB_FIND(xh_core_map_info_tree, &xh_core_map_info, &mi_key)))    

{    

//exist    

RB_REMOVE(xh_core_map_info_tree, &xh_core_map_info, mi);    

//repeated?    

//We only keep the first one, that is the real base address    

if(NULL != RB_INSERT(xh_core_map_info_tree, &map_info_refreshed, mi))    

{    

#if XH_CORE_DEBUG    

XH_LOG_DEBUG("repeated map info when update: %s", line);    

#endif    

free(mi->pathname);    

free(mi);    

continue;    

}    

//re-hook if base_addr changed    

if(mi->base_addr != base_addr)    

{    

mi->base_addr = base_addr;    

xh_core_hook(mi);    

}    

}    

else    

{    

//not exist, create a new map info    

if(NULL == (mi = (xh_core_map_info_t *)malloc(sizeof(xh_core_map_info_t)))) continue;    

if(NULL == (mi->pathname = strdup(pathname)))    

{    

free(mi);    

continue;    

}    

mi->base_addr = base_addr;    

//repeated?    

//We only keep the first one, that is the real base address    

if(NULL != RB_INSERT(xh_core_map_info_tree, &map_info_refreshed, mi))    

{    

#if XH_CORE_DEBUG    

XH_LOG_DEBUG("repeated map info when create: %s", line);    

#endif    

free(mi->pathname);    

free(mi);    

continue;    

}    

//hook    

xh_core_hook(mi); //hook    

}    

}    

fclose(fp);    

//free all missing map item, maybe dlclosed?    

RB_FOREACH_SAFE(mi, xh_core_map_info_tree, &xh_core_map_info, mi_tmp)    

{    

#if XH_CORE_DEBUG    

XH_LOG_DEBUG("remove missing map info: %s", mi->pathname);    

#endif    

RB_REMOVE(xh_core_map_info_tree, &xh_core_map_info, mi);    

if(mi->pathname) free(mi->pathname);    

free(mi);    

}    

//save the new refreshed map info tree    

xh_core_map_info = map_info_refreshed;    

XH_LOG_INFO("map refreshed");    

#if XH_CORE_DEBUG    

RB_FOREACH(mi, xh_core_map_info_tree, &xh_core_map_info)    

XH_LOG_DEBUG("  %"PRIxPTR" %s\n", mi->base_addr, mi->pathname);    

#endif    

}
```

地址替换通过`PLT`表，详细原理[e39K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3$3y4o6t1$3x3U0l9$3](https://bbs.kanxue.com/elink@ba8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3$3y4o6t1$3x3U0l9$3)。
