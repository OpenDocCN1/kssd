
```
char il2cpp::vm::MetadataCache::Initialize()
{
    v0 = sub_180261550("global-metadata.dat");
    *&xmmword_182B7C2D8 = v0;
    if ( v0 )
    {
        *(&xmmword_182B7C2D8 + 1) = v0;
        qword_182B7B948 = j_j__calloc_base(*(qword_182B7C2C0 + 48), 8i64);
        qword_182B7B950 = j_j__calloc_base(*(*(&xmmword_182B7C2D8 + 1) + 164i64) / 0x5Cui64, 8i64);
        qword_182B7B958 = j_j__calloc_base(*(*(&xmmword_182B7C2D8 + 1) + 52i64) >> 5, 8i64);
        qword_182B7B968 = j_j__calloc_base(*(qword_182B7C2C0 + 64), 8i64);
        dword_182B7B970 = *(*(&xmmword_182B7C2D8 + 1) + 172i64) / 0x28ui64;
        qword_182B7B978 = j_j__calloc_base(dword_182B7B970, 80i64);
        dword_182B7B980 = *(*(&xmmword_182B7C2D8 + 1) + 180i64) / 0x44ui64;
        qword_182B7B988 = j_j__calloc_base(dword_182B7B980, 96i64);
        v1 = *(&xmmword_182B7C2D8 + 1);
```

对比下源码不难发现 `sub_180261550` 就是 `il2cpp::vm::MetadataLoader::LoadMetadataFromFile`

需要注意的是这个 `v0` 指针十分重要，因为解混淆或是解密都会调用到这个指针（一般是在加载前解密，而不是在加载后解密，这样会导致未解密的数据残余在内存中影响性能），通常来说应用程序在访问 `metadata` 前执行解密，或是在 `il2cpp::vm::MetadataLoader::LoadMetadataFromFile` 之前或之中。

### Finding the metadata loader: What if there is no `il2cpp_init`?

那么以上套路是针对我们能在 il2cpp 里找到 `il2cpp_init` 的函数，但是如果没有，我们就要继续往上找 `UnityPlayer.dll` 或者 `libunity.so`，而这个我们没有办法用源码对照，不过我们可以创建一个 Unity 项目生成 PDB，就可以看到名字和符号那些。

对于这种情况我们主要关心 `il2cpp_init.cpp` 到底是哪里被引用与调用，那么未被混淆的流程如此

![](https://bbs.kanxue.com/upload/attach/202308/930234_P63GZM9GRPTJZP7.png)

`UnityMainImpl` 有很多函数调用，不过我们可以通过字符串来寻找

```
winutils::DisplayErrorMessagesAndQuit("Data folder not found");
}
DetectIL2CPPVersion();
v78.m_data = 0i64;
v78.m_size = 0i64;
v78.m_label.identifier = 68;
v78.m_internal[0] = 0;
core::StringStorageDefault<char>::assign(&v78, "GameAssembly.dll", 0x10ui64);
v27 = !LoadIl2Cpp(&v78);
if ( v78.m_data && v78.m_capacity > 0 )
    operator delete(v78.m_data, v78.m_label);
if ( v27 )
    winutils::DisplayErrorMessagesAndQuit("Failed to load il2cpp");
v78.m_data = 0i64;
v78.m_size = 0i64;
v78.m_label.identifier = 68;
v78.m_internal[0] = 0;
core::StringStorageDefault<char>::assign(&v78, "il2cpp_data", 0xBui64);
```

`LoadIl2Cpp` 也同样可以字符串快速寻找

```
v2 = 1;
il2cpp_init = LookupSymbol(v1, "il2cpp_init", kSymbolRequired);
if ( !il2cpp_init )
{
    v2 = 0;
    printf_console("il2cpp: function il2cpp_init not found\n");
}
il2cpp_init_utf16 = LookupSymbol(gIl2CppModule, "il2cpp_init_utf16", kSymbolRequired);
if ( !il2cpp_init_utf16 )
{
    v2 = 0;
    printf_console("il2cpp: function il2cpp_init_utf16 not found\n");
}
il2cpp_shutdown = LookupSymbol(gIl2CppModule, "il2cpp_shutdown", kSymbolRequired);
if ( !il2cpp_shutdown )
{
    v2 = 0;
    printf_console("il2cpp: function il2cpp_shutdown not found\n");
}
```

`InitializeIl2CppFromMain`

```
char __fastcall InitializeIl2CppFromMain(const core::basic_string<char,core::StringStorageDefault<char> > *monoConfigPath, const core::basic_string<char,core::StringStorageDefault<char> > *dataPath, int argc, const char **argv)
{
    v4 = argv;
    v5 = argc;
    v6 = dataPath;
    v7 = monoConfigPath;
    RegisterAllInternalCalls();
    il2cpp_runtime_unhandled_exception_policy_set(IL2CPP_UNHANDLED_POLICY_LEGACY);
    il2cpp_set_commandline_arguments(v5, v4, 0i64);
    v8 = v7->m_data;
    if ( !v7->m_data )
        v8 = &v7->8;
    il2cpp_set_config_dir(v8);
    v9 = v6->m_data;
    if ( !v6->m_data )
        v9 = &v6->8;
    il2cpp_set_data_dir(v9);
    v10 = GetMonoDebuggerAgentOptions(&result, 0);
    v11 = v10->m_data;
    if ( !v10->m_data )
        v11 = &v10->8;
    il2cpp_debugger_set_agent_options(v11);
    if ( result.m_data && result.m_capacity )
        operator delete(result.m_data, result.m_label);
    il2cpp_init("IL2CPP Root Domain");
    il2cpp_set_config("unused_application_configuration");
    profiling::ScriptingProfiler::Initialize();
    return 1;
}
```

然而有相当不同的变化在不同的版本中，他们的共同特点是对 `il2cpp_init` 的调用和 `IL2CPP Root Domain` 与 `unused_application_configuration`，所以通过这些我们也同样快速找到（如果没有混淆的话）。

## D3Mug

### 0x00 Daily Check Shell

IL2CPP逆向，无壳。

于是惯用套路查找 `libil2cpp.so` 和 `global-metadata.dat` 来恢复符号，metadata未被混淆，所以可以直接恢复。

接着可以配合 `Assembly-CSharp.dll` 或 `dump.cs` 来配合查看函数信息。

![](https://bbs.kanxue.com/upload/attach/202308/930234_Y5GY24UUXQGWV4B.png)

### 0x01 Track

尝试 hook IL2CPP 的函数，可是都不行，看别人博客都是模拟器跳这个，我真机跳这个是为什么，hook 未果

![](https://bbs.kanxue.com/upload/attach/202308/930234_9NZYTM7YFY86P39.png)

那么开始静态审计，既然是音游，通关调试肯定是全部按好即可，于是在 `Assembly-CSharp.dll` 很容易定位到 `notehit` `notemiss` 这些函数

```
void __fastcall GameManager__NoteHit(GameManager_o *this, float preciseTime, int32_t level, const MethodInfo *method)
{
    struct TMPro_TMP_Text_o *PerfectText; // x19
    System_String_o *v6; // x0
    System_String_o *v7; // x0
    const MethodInfo *v8; // x1
    uint32_t v9; // w0
    int v10; // [xsp+Ch] [xbp-14h] BYREF

    v10 = 0;
    if ( level >= 1 )
    {
        PerfectText = this->fields.PerfectText;
        if ( PerfectText )
            goto LABEL_3;
        LABEL_8:
        sub_560668();
    }
    PerfectText = this->fields.GoodText;
    if ( !PerfectText )
        goto LABEL_8;
    LABEL_3:
    v6 = (PerfectText->klass->vtable._65_get_text.method)(
            PerfectText,
            PerfectText->klass->vtable._66_set_text.methodPtr,
            method);
    v10 = System_Int32__Parse(v6, 0LL) + 1;
    v7 = System_Int32__ToString(&v10, 0LL);
    (PerfectText->klass->vtable._66_set_text.method)(
        PerfectText,
        v7,
        PerfectText->klass->vtable._67_get_fontSharedMaterial.methodPtr);
    if ( (preciseTime * 1000.0) >= 0.0 )
        v9 = (preciseTime * 1000.0);
    else
        v9 = (preciseTime * 1000.0);
    GameManager__update(v9, v8);
}
```

共同点就是算好分数，就直接调用 `GameManager__update`，而该函数往里查找可以发现调用了 `d3mug.so` 文件里的 `update` 函数

那么可以得出结论每次点击到 `preciseTime` 就会正确 `update`，而该 `update` 函数会把值更新到 `Server:instance` 里
