
```
PVOID LocateLoadedAssemblies() {
#ifdef ver5_6_5_x86
/*
Function: mono_assembly_foreach
Byte:
56                            push    esi
57                            push    edi
BF 84 B4 20 10                mov     edi, offset stru_1020B484
57                            push    edi; lpCriticalSection
FF 15 4C 61 14 10             call    ds : EnterCriticalSection
FF 35 7C B4 20 10             push    loaded_assemblies
*/
PVOID monoModule = GetModuleHandleA("mono.dll");
PVOID mono_assembly_foreach = GetProcAddress((HMODULE)monoModule, "mono_assembly_foreach");
vector<ULONG64> res;
int Cnt = SearchMemory((HANDLE)-1, (char*)"56 57 BF ?? ?? ?? ?? 57 ?? ?? ?? ?? ?? ?? FF 35 ?? ?? ?? ??", (unsigned long long)mono_assembly_foreach, (unsigned long long)((DWORD)mono_assembly_foreach + 50), 10, res);
Print("SigCnt2 = %d", Cnt);
if (Cnt == 1) {
    ULONG64 OffsetAddress = res[0] + 16;
    int LoadedAssemblies = *(int*)OffsetAddress;
    Print("LoadedAssemblies = %X", LoadedAssemblies);
    return (PVOID)(LoadedAssemblies);
}
return 0;
#else
return 0;
#endif
}

PVOID LocateLoadedImagesHashAddress() {
#ifdef ver5_6_5_x86
/*
Function: mono_image_init
Byte:
57                            push    edi
56                            push    esi
E8 FA 74 FC FF                call    sub_100012C0
57                            push    edi
56                            push    esi
A3 F4 B6 20 10                mov     loaded_images_hash, eax
*/
PVOID monoModule = GetModuleHandleA("mono.dll");
Print("MonoModule = %X", monoModule);
PVOID mono_image_init = GetProcAddress((HMODULE)monoModule, "mono_images_init");
Print("mono_image_init = %X", mono_image_init);
vector<ULONG64> res;
int Cnt = SearchMemory((HANDLE)-1, (char*)"57 56 E8 ?? ?? ?? ?? 57 56 A3 ?? ?? ?? ??", (unsigned long long)mono_image_init, (unsigned long long)((DWORD)mono_image_init + 50), 10, res);
Print("SigCnt = %d", Cnt);
if (Cnt == 1) {
    ULONG64 OffsetAddress = res[0] + 10;
    int LoadedImagesHashAddress = *(int*)OffsetAddress;
    Print("LoadedImagesHashAddress = %X", LoadedImagesHashAddress);
    return (PVOID)(LoadedImagesHashAddress);
}
return 0;
#else
return 0;
#endif
}
```

# 遍历

```
GHashTable* loaded_images_hash;
GList* loaded_assemblies;
loaded_images_hash = *(GHashTable**)LocateLoadedImagesHashAddress();
loaded_assemblies = *(GList**)LocateLoadedAssemblies();
g_hash_table_foreach(loaded_images, image_Foreach_routine,NULL);
g_list_foreach(loaded_assemblies, assemblies_Foreach_routine, NULL);
```

`g_hash_table_foreach`和`g_list_foreach`等都是glib的函数，在mono的源码里有，自己去CV吧

## 遍历回调函数

```
static void
image_Foreach_routine(gpointer key, gpointer val, gpointer user_data)
{
    Print("Foreach key = %X",key);
    Print("Foreach val = %X", val);
    Print("Foreach key->string = %s", key);
    string test = (char*)key;
    MonoImage* image = (MonoImage*)val;
    if (test.find("data-") != string::npos) {
        Print("Find Cheat!");
        Print("Foreach key->string = %s", image->assembly_name);
    }
}
```

```
static void
assemblies_Foreach_routine(gpointer data, gpointer user_data) {
    MonoAssembly* ass = (MonoAssembly*) data;
    string assName = ass->aname.name;
    if (assName.find("BobHSSJJ") != string::npos) {
        Print("Find Cheat!");
    }
    Print("AssName = %s", ass->aname.name);
    Print("BaseDir = %s", ass->basedir);
}
```

# 后记

小伙子忙说对不起对不起，我不懂规矩啊，我说他是乱打的，他可不是乱打啊，铮铮鞭腿左刺拳训练有素，后来他说他练过三四年泰拳，看来是有备而来

他说他把链表断了，然后把PE头也清掉了，还把所有字符串都清空了
