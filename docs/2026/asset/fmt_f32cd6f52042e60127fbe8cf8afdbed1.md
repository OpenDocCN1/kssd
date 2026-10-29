
`xref:` `/art/runtime/native/dalvik_system_DexFile.cc`
`std::unique_ptr<std::vector<const DexFile*>> dex_files(new std::vector<const DexFile*>());`

### DexFile_openDexFileNative

```
static jlong DexFile_openDexFileNative(JNIEnv* env, jclass, jstring javaSourceName, jstring javaOutputName, jint) {
  ...
  ClassLinker* linker = Runtime::Current()->GetClassLinker();
  std::unique_ptr<std::vector<const DexFile*>> dex_files(new std::vector<const DexFile*>());
  bool success = linker->OpenDexFilesFromOat(sourceName.c_str(), outputName.c_str(), &error_msgs, dex_files.get());
  if (success || !dex_files->empty()) {
    // In the case of non-success, we have not found or could not generate the oat file.
    // But we may still have found a dex file that we can use.
    return static_cast<jlong>(reinterpret_cast<uintptr_t>(dex_files.release()));
  }
}
```

## class_linker

#### open_oat_file

`xref:` `/art/runtime/class_linker.cc`

### OpenDexFilesFromOat

```
bool ClassLinker::OpenDexFilesFromOat(const char* dex_location, const char* oat_location,
  std::vector<std::string>* error_msgs,
  std::vector<const DexFile*>* dex_files) {
  if (!DexFile::GetChecksum(dex_location, dex_location_checksum_pointer, &checksum_error_msg)) {
    ...
  }
  // 检查是否我们有一个已经打开的oatfile(内存中)
  const OatFile::OatDexFile* oat_dex_file = FindOpenedOatDexFile(oat_location, dex_location, dex_location_checksum_pointer);
  std::unique_ptr<const OatFile> open_oat_file(
    oat_dex_file != nullptr ? oat_dex_file->GetOatFile() : nullptr);

  // 检查是否我们有一个可以使用的的oatfile(磁盘上)，通过oat_location字符串
  if (open_oat_file.get() == nullptr) {
    open_oat_file.reset(FindOatFileInOatLocationForDexFile(dex_location, dex_location_checksum, oat_location, &error_msg));
    ...
    needs_registering = true;
  }

  // 如果我们有一个oat文件，检查所有包含的multidex文件以确定我们的dex位置
  bool success = LoadMultiDexFilesFromOatFile(open_oat_file.get(), dex_location,
    dex_location_checksum_pointer,
    false, error_msgs, dex_files);
  if (success) {
    const OatFile* oat_file = open_oat_file.release();
    // Avoid deleting it.
    if (needs_registering) {
      // We opened the oat file, so we must register it.
      RegisterOatFile(oat_file);
    }
    return oat_file->IsExecutable();
  }

  // 如果不是这样(要么没有oat文件，要么不匹配)，重新生成并加载oat，通过dex_location字符串
  if (Runtime::Current()->IsDex2OatEnabled() && has_flock && scoped_flock.HasFile()) {
    // Create the oat file.
    open_oat_file.reset(CreateOatFileForDexLocation(dex_location, scoped_flock.GetFile()->Fd(), oat_location, error_msgs));
  }

  // Failed, bail.art模式下的解析模式，以上都是编译成oat的编译模式
  if (open_oat_file.get() == nullptr) {
    std::string error_msg;
    // dex2oat was disabled or crashed. Add the dex file in the list of dex_files to make progress.
    DexFile::Open(dex_location, dex_location, &error_msg, dex_files);
    error_msgs->push_back(error_msg);
    return false;
  }

  // Try to load again, but stronger checks.
  success = LoadMultiDexFilesFromOatFile(open_oat_file.get(), dex_location,
    dex_location_checksum_pointer,
    true, error_msgs, dex_files);
  if (success) {
    RegisterOatFile(open_oat_file.release());
    return true;
  }
}
```

### CreateOatFileForDexLocation

```
const OatFile* ClassLinker::CreateOatFileForDexLocation(const char* dex_location, int fd, const char* oat_location, std::vector<std::string>* error_msgs) {
  GenerateOatFile(dex_location, fd, oat_location, &error_msg)
  std::unique_ptr<OatFile> oat_file(OatFile::Open(oat_location, oat_location, nullptr, !Runtime::Current()->IsCompiler(), &error_msg));
  return oat_file.release();
}
```

### GenerateOatFile

```
bool ClassLinker::GenerateOatFile(const char* dex_filename,
  int oat_fd,
  const char* oat_cache_filename,
  std::string* error_msg) {
  std::string dex2oat(Runtime::Current()->GetCompilerExecutable());
  // 执行dex2oat
  ...
  return Exec(argv, error_msg);
}
```

## dex_file

#### dex_file

`xref:` `/art/runtime/dex_file.cc`

### DexFile::Open

```
bool DexFile::Open(const char* filename, const char* location, std::string* error_msg, std::vector<const DexFile*>* dex_files) {
  std::unique_ptr<const DexFile> dex_file(DexFile::OpenFile(fd.release(), location, true, error_msg));
  if (dex_file.get() != nullptr) {
    dex_files->push_back(dex_file.release());
  }
}
```

### DexFile::OpenFile

```
const DexFile* DexFile::OpenFile(int fd, const char* location, bool verify, std::string* error_msg) {
  std::unique_ptr<MemMap> map;
  map.reset(MemMap::MapFile(length, PROT_READ, MAP_PRIVATE, fd, 0, location, error_msg));
  const Header* dex_header = reinterpret_cast<const Header*>(map->Begin());
  const DexFile* dex_file = OpenMemory(location, dex_header->checksum_, map.release(), error_msg);
  return dex_file;
}
```

### DexFile::OpenMemory

```
const DexFile* DexFile::OpenMemory(const std::string& location,
  uint32_t location_checksum,
  MemMap* mem_map,
  std::string* error_msg) {
  return OpenMemory(mem_map->Begin(),
    mem_map->Size(),
    location,
    location_checksum,
    mem_map,
    error_msg);
}

const DexFile* DexFile::OpenMemory(const byte* base,
  size_t size,
  const std::string& location,
  uint32_t location_checksum,
  MemMap* mem_map, std::string* error_msg) {
  CHECK_ALIGNED(base, 4);
  // various dex file structures must be word aligned
  std::unique_ptr<DexFile> dex_file(new DexFile(base, size, location, location_checksum, mem_map));
  if (!dex_file->Init(error_msg)) {
    return nullptr;
  } else {
    return dex_file.release();
  }
}
```

### DexFile::DexFile

```
DexFile::DexFile(const byte* base, size_t size,
  const std::string& location,
  uint32_t location_checksum,
  MemMap* mem_map)
  : begin_(base),
    size_(size),
    location_(location),
    location_checksum_(location_checksum),
    mem_map_(mem_map),
    header_(reinterpret_cast<const Header*>(base)),
    string_ids_(reinterpret_cast<const StringId*>(base + header_->string_ids_off_)),
    type_ids_(reinterpret_cast<const TypeId*>(base + header_->type_ids_off_)),
    field_ids_(reinterpret_cast<const FieldId*>(base + header_->field_ids_off_)),
    method_ids_(reinterpret_cast<const MethodId*>(base + header_->method_ids_off_)),
    proto_ids_(reinterpret_cast<const ProtoId*>(base + header_->proto_ids_off_)),
    class_defs_(reinterpret_cast<const ClassDef*>(base + header_->class_defs_off_)),
    find_class_def_misses_(0),
    class_def_index_(nullptr),
    build_class_def_index_mutex_("DexFile index creation mutex") {
  CHECK(begin_ != NULL) << GetLocation();
  CHECK_GT(size_, 0U) << GetLocation();
}
```

# so层小结：
