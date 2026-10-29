
> art_dex_file_loader.cc

```cpp
static const DexFile* CreateDexFile(JNIEnv* env, std::unique_ptr<MemMap> dex_mem_map) {
  std::string location = StringPrintf("Anonymous-DexFile@%p-%p",
    dex_mem_map->Begin(),
    dex_mem_map->End());
  std::string error_message;
  const ArtDexFileLoader dex_file_loader;
  // 打开DexFile
  std::unique_ptr<const DexFile> dex_file(dex_file_loader.Open(location,
  // Anonymous-DexFile@0xaabbccdd-0xeeff0011
  0,
  std::move(dex_mem_map),
  /* verify */ true,
  /* verify_location */ true,
  &error_message));
  // ...省略，chinamima
  return dex_file.release();
}
```

```cpp
static constexpr OatDexFile* kNoOatDexFile = nullptr;

std::unique_ptr<const DexFile> ArtDexFileLoader::Open(const uint8_t* base,
  size_t size,
  const std::string& location,
  uint32_t location_checksum,
  const OatDexFile* oat_dex_file,
  bool verify,
  bool verify_checksum,
  std::string* error_msg) const {
  ScopedTrace trace(std::string("Open dex file from RAM ") + location);
  return OpenCommon(base,
    size,
    /*data_base*/ nullptr,
    /*data_size*/ 0u,
    location,
    location_checksum,
    oat_dex_file,
    // 有oat文件
    verify,
    verify_checksum,
    error_msg,
    /*container*/ nullptr,
    /*verify_result*/ nullptr);
}

std::unique_ptr<const DexFile> ArtDexFileLoader::Open(const std::string& location,
  uint32_t location_checksum,
  std::unique_ptr<MemMap> map,
  bool verify,
  bool verify_checksum,
  std::string* error_msg) const {
  // ...省略，chinamima
  std::unique_ptr<DexFile> dex_file = OpenCommon(map->Begin(),
    map->Size(),
    /*data_base*/ nullptr,
    /*data_size*/ 0u,
    location,
    location_checksum,
    kNoOatDexFile,
    // 无oat文件！
    verify,
    verify_checksum,
    error_msg,
    std::make_unique<MemMapContainer>(std::move(map)),
    /*verify_result*/ nullptr);
  // ...省略，chinamima
  return dex_file;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-4-21 10:59

被chinamima编辑，原因： 调整格式，提示关键步奏

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
