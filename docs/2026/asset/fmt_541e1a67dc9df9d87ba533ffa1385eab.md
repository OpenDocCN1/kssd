
```cpp
void load_modules() {
    DIR *dir;
    struct dirent *entry;
    char path[PATH_MAX];
    void *handle;
    const int riruApiVersion = RIRU_API_VERSION;

    if (!(dir = _opendir(MODULES_DIR))) return;

    // 遍历/data/adb/riru/modules目录下的文件夹
    while ((entry = _readdir(dir))) {
        if (entry->d_type != DT_DIR) continue;

        // 获取文件夹名称
        auto name = entry->d_name;
        if (name[0] == '.') continue;

        // 根据文件夹名称拼接so路径：/system/lib/libriru_%s.so，这一步操作也是在riru插件的customize.sh中完成
        snprintf(path, PATH_MAX, MODULE_PATH_FMT, name);

        if (access(path, F_OK) != 0) {
            PLOGE("access %s", path);
            continue;
        }

        handle = dlopen(path, 0);
        if (!handle) {
            LOGE("dlopen %s failed: %s", path, dlerror());
            continue;
        }

        // 找到so的init方法，这里也是为什么riru的官方文档中指出必须要有一个导出的init函数
        // 如果没有init方法，则将不会被认为是一个合法的riru module，后续get_modules也无法获取到
        auto init = (RiruInit_t *) dlsym(handle, "init");
        if (!init) {
            LOGW("%s does not export init", path);
            cleanup(handle, path);
            continue;
        }

        // 1. pass riru api version, return module's api version
        auto apiVersion = (int *) init((void *) &riruApiVersion);
        if (apiVersion == nullptr) {
            LOGE("%s returns null on step 1", path);
            cleanup(handle, path);
            continue;
        }

        if (*apiVersion < RIRU_MIN_API_VERSION || *apiVersion > RIRU_API_VERSION) {
            LOGW("unsupported API %s: %d", name, *apiVersion);
            cleanup(handle, path);
            continue;
        }

        // 2. create and pass Riru struct by module's api version
        auto module = new RiruModule(strdup(name));
        module->handle = handle;
        module->apiVersion = *apiVersion;

        if (*apiVersion == 9) {
            auto info = init_module_v9(module->token, init);
            if (info == nullptr) {
                LOGE("%s returns null on step 2", path);
                cleanup(handle, path);
                continue;
            }
            module->info(info);
        }

        // 3. let the module to do some cleanup jobs
        init(nullptr);

        // 缓存插件信息到一个vector中，后续通过get_modules()方法才能找到对应的插件信息
        get_modules()->push_back(module);

        LOGI("module loaded: %s (api %d)", module->name, module->apiVersion);
    }

    closedir(dir);

    status::getStatus()->hideEnabled = access(ENABLE_HIDE_FILE, F_OK) == 0;
    if (status::getStatus()->hideEnabled) {
        LOGI("hide is enabled");
        auto modules = get_modules();
        auto names = (const char **) malloc(sizeof(char *) * modules->size());
        int names_count = 0;
        for (auto module : *get_modules()) {
            if (strcmp(module->name, MODULE_NAME_CORE) == 0) continue;
            if (!module->supportHide) {
                LOGI("module %s does not support hide", module->name);
                continue;
            }
            names[names_count] = module->name;
            names_count += 1;
        }
        hide::hide_modules(names, names_count);
    } else {
        PLOGE("access " ENABLE_HIDE_FILE);
        LOGI("hide is not enabled");
    }

    for (auto module : *get_modules()) {
        if (module->hasOnModuleLoaded()) {
            LOGV("%s: onModuleLoaded", module->name);
            // 回调module的_onModuleLoaded方法
            module->onModuleLoaded();
        }
    }
}
```

# EdXposed如何依赖riru

EdXposed本身也是一个riru插件，但是如何证明呢。

我们可以知道如果作为一个riru插件，必然需要

1. 在`/data/adb/riru/modules`或者`/data/misc/riru/modules`下存在一个插件文件夹。
2. 存在与插件文件夹相同名称的`/system/lib/libriru_%s.so`。
3. so中可能有`init`或者`forkAndSpecializePre`等zygote进程相关的方法。

在`edxp-core`中，依然通过`build.gradle`为入口分析zip打包的逻辑，可以看到也是一个标准的magisk module installer，其中`customize.sh`中有一行关键代码创建了riru插件的相关目录：

```bash
# 创建riru的module目录，这里可以看出edxp同时是一个riru module，riru通过检测目录加载对应的libriru_xxx.so
[[ -d "${RIRU_TARGET}" ]] || mkdir -p "${RIRU_TARGET}" || abort "! Can't mkdir -p ${RIRU_TARGET}"

# RIRU_TARGET变量的赋值过程
RIRU_PATH="/data/misc/riru"
# 为libriru_edxp.so设置随机后缀, libriru_edxp.so -> libriru_{randomNum}.so
# libriru_edxp.so由edxp-core:src/main/cpp/main相关c文件编译得出
# getRandomNameExist方法作用为获取一个/proc/sys/kernel/random/uuid不存在的四位随机数
RIRU_EDXP="$(getRandomNameExist 4 "libriru_" ".so" /system/lib /system/lib64)"
RIRU_MODULES="${RIRU_PATH}/modules"
RIRU_TARGET="${RIRU_MODULES}/${RIRU_EDXP}"
```

通过这段脚本可以看出edxp插件在magisk加载时创建了一个riru插件目录，并且把`libriru_edxp.so`重命名成一个随机后缀的so文件由riru进行加载。

接着分析`libriru_edxp.so`，通过`CMakeLists.txt`可以知道是由`edxp-core`中编译得到，查看`edxp-core/src/main/cpp/main/src/main.cpp`中可以看到确实声明了riru中的一些方法模版钩子：

- `nativeForkAndSpecializePre`
- `nativeForkAndSpecializePost`
- `nativeForkSystemServerPre`
- `nativeForkSystemServerPost`
- `specializeAppProcessPre`
- `specializeAppProcessPost`
- `onModuleLoaded`
- `shouldSkipUid`

等方法，一切都可以想明白了。接下来就是edxp如何进行ART Hook。

# PS

1. [riru-docs](<https://bbs.kanxue.com/elink@281K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8r3W2W2L8Y4N6S2M7X3g2t1k6g2)9J5c8Y4u0A6M7Y4g2Q4x3X3c8V1L8$3y4K6):f7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8r3W2W2L8Y4N6S2M7X3g2t1k6g2)9J5c8Y4u0A6M7Y4g2Q4x3X3c8V1L8$3y4K6>)
2. 目前的riru v22中可以看到一个riru module必须包含一个`init`导出函数，但是在edxp里并没有发现有`init`函数，在v19的riru中是没有`init`函数限制，而是校验`module.prop`。
3. riru v22中提到切换到native bridge方式注入之后，会导致riru和模块的加载时机变晚，对于Xposed框架可能会有变化和影响。

Starting v22.0, Riru has switched to "native bridge" (`ro.dalvik.vm.native.bridge`) to inject zygote, this will lead Riru and modules be loaded later (`LoadNativeBridge` vs `__attribute__((constructor))`).

For most modules, this should have no problem, but modules like Xposed frameworks may have to make changes.

Magisk may provider Riru-like features in the far future, and of course, it will have more strict restrictions, module codes will not be run in zygote. Maybe Xposed framework modules should prepare for this?

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-10-27 17:37 被 alienhe 编辑，原因：
