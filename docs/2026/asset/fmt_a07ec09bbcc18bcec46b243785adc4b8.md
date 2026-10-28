
```c++
// Step 5: link libraries that are not destined to this namespace.
// Do this by recursively calling find_libraries on the namespace where the lib
// was found during Step 1.
for (auto&& task : load_tasks) {
soinfo* si = task->get_soinfo();
if (si->get_primary_namespace() != ns) {
const char* name = task->get_name();
if (find_libraries(si->get_primary_namespace(), task->get_needed_by(), &name, 1,
nullptr /* soinfos */, nullptr /* ld_preloads */, 0 /* ld_preload_count */,
rtld_flags, nullptr /* extinfo */, false /* add_as_children */,
false /* search_linked_namespaces */, readers_map, namespaces)) {
// If this lib is directly needed by one of the libs in this namespace,
// then increment the count
soinfo* needed_by = task->get_needed_by();
if (needed_by != nullptr && needed_by->get_primary_namespace() == ns && si->is_linked()) {
si->increment_ref_count();
}
} else {
return false;
}
}
}
//连接这个命名空间的库文件
// Step 6: link libraries in this namespace
soinfo_list_t local_group;
walk_dependencies_tree(
(start_with != nullptr && add_as_children) ? &start_with : soinfos,
(start_with != nullptr && add_as_children) ? 1 : soinfos_count,
[&] (soinfo* si) {
if (ns->is_accessible(si)) {
local_group.push_back(si);
return kWalkContinue;
} else {
return kWalkSkip;
}
});
soinfo_list_t global_group = ns->get_global_group();
bool linked = local_group.visit([&](soinfo* si) {
if (!si->is_linked()) {
LD_LOG(kLogDlopen,"so %s is not linked , now try to link ",si->get_soname());
if (!si->link_image(global_group, local_group, extinfo) ||
!get_cfi_shadow()->AfterLoad(si, solist_get_head())) {
return false;
}
}
return true;
});
if (linked) {
local_group.for_each([](soinfo* si) {
LD_LOG(kLogDlopen,"travser local_group list %s" , si->get_soname());
if (!si->is_linked()) {
si->set_linked();
}
});
failure_guard.Disable();
}
return linked;
```

涉及到重定位的镜像连接。

```c++
bool soinfo::link_image(const soinfo_list_t& global_group, const soinfo_list_t& local_group,
const android_dlextinfo* extinfo) {
// 这里仅保留了大家比较熟悉的类型。常规都是rel。
#if defined(USE_RELA)
if (rela_ != nullptr) {
DEBUG("[ relocating %s ]", get_realpath());
if (!relocate(version_tracker,
plain_reloc_iterator(rela_, rela_count_), global_group, local_group)) {
return false;
}
}
if (plt_rela_ != nullptr) {
DEBUG("[ relocating %s plt ]", get_realpath());
if (!relocate(version_tracker,
plain_reloc_iterator(plt_rela_, plt_rela_count_), global_group, local_group)) {
return false;
}
}
#else
//把重心放在这里
if ( != nullptr) {
DEBUG("[ relocating %s ]", get_realpath());
if (!relocate(version_tracker,
plain_reloc_iterator(rel_, rel_count_), global_group, local_group)) {
return false;
}
}
if (plt_rel_ != nullptr) {
LD_LOG(kLogDlopen,"[ relocating %s plt ]", get_realpath());
if (!relocate(version_tracker,
plain_reloc_iterator(plt_rel_, plt_rel_count_), global_group, local_group)) {
return false;
}
}
#endif
```

在Step3中找这`rel_`和`plt_rel_`的赋值，分别为`DT_JMPREL`和`DT_REL`对应`.rel.plt`和`.rel.dyn`段。

```c++
//该记录保留仅与重定位相关的PLT表的地址
/* ... */
case DT_JMPREL:
#if defined(USE_RELA)
plt_rela_ = reinterpret_cast<ElfW(Rela)*>(load_bias + d->d_un.d_ptr);
#else
plt_rel_ = reinterpret_cast<ElfW(Rel)*>(load_bias + d->d_un.d_ptr);
#endif
break;
//重定位相关plt表的大小
case DT_PLTRELSZ:
#if defined(USE_RELA)
plt_rela_count_ = d->d_un.d_val / sizeof(ElfW(Rela));
#else
plt_rel_count_ = d->d_un.d_val / sizeof(ElfW(Rel));
#endif
break;
/* ... */
case DT_REL:
rel_ = reinterpret_cast<ElfW(Rel)*>(load_bias + d->d_un.d_ptr);
break;
case DT_RELSZ:
rel_count_ = d->d_un.d_val / sizeof(ElfW(Rel));
break;
```

参数中有`plain_reloc_iterator`，传入`rel`的迭代器。

```c++
relocate(version_tracker, plain_reloc_iterator(rel_, rel_count_), global_group, local_group)
```

`Elf32_Rel`的结构体。

```c++
typedef struct
{
Elf32_Addr r_offset; /* Address */
Elf32_Word r_info; /* Relocation type and symbol index */
} Elf32_Rel;
```

对于`DT_REL`，该函数获取到`rel.dyn`段，到这里重定位的内容就很简单了。
