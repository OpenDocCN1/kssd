
```c++
std::vector<std::unique_ptr<const DexFile>> ClassLinker::OpenDexFilesFromOat(
const char* dex_location, const char* oat_location,
std::vector<std::string>* error_msgs) {
CHECK(error_msgs != nullptr);
// Verify we aren't holding the mutator lock, which could starve GC if we
// have to generate or relocate an oat file.
Locks::mutator_lock_->AssertNotHeld(Thread::Current());
OatFileAssistant oat_file_assistant(dex_location, oat_location, kRuntimeISA,
!Runtime::Current()->IsAotCompiler());
// Lock the target oat location to avoid races generating and loading the
// oat file.
std::string error_msg;
if (!oat_file_assistant.Lock(&error_msg)) {
// Don't worry too much if this fails. If it does fail, it's unlikely we
// can generate an oat file anyway.
VLOG(class_linker) << "OatFileAssistant::Lock: " << error_msg;
}
// Check if we already have an up-to-date oat file open.
const OatFile* source_oat_file = nullptr;
{
ReaderMutexLock mu(Thread::Current(), dex_lock_);
for (const OatFile* oat_file : oat_files_) {
CHECK(oat_file != nullptr);
if (oat_file_assistant.GivenOatFileIsUpToDate(*oat_file)) {
source_oat_file = oat_file;
break;
}
}
}
// If we didn't have an up-to-date oat file open, try to load one from disk.
if (source_oat_file == nullptr) {
// Update the oat file on disk if we can. This may fail, but that's okay.
// Best effort is all that matters here.
if (!oat_file_assistant.MakeUpToDate(&error_msg)) {
LOG(WARNING) << error_msg;
}
// Get the oat file on disk.
std::unique_ptr<OatFile> oat_file = oat_file_assistant.GetBestOatFile();
if (oat_file.get() != nullptr) {
// Take the file only if it has no collisions, or we must take it because of preopting.
bool accept_oat_file = !HasCollisions(oat_file.get(), &error_msg);
if (!accept_oat_file) {
// Failed the collision check. Print warning.
if (Runtime::Current()->IsDexFileFallbackEnabled()) {
LOG(WARNING) << "Found duplicate classes, falling back to interpreter mode for "
<< dex_location;
} else {
LOG(WARNING) << "Found duplicate classes, dex-file-fallback disabled, will be failing to "
" load classes for " << dex_location;
}
LOG(WARNING) << error_msg;
// However, if the app was part of /system and preopted, there is no original dex file
// available. In that case grudgingly accept the oat file.
if (!DexFile::MaybeDex(dex_location)) {
accept_oat_file = true;
LOG(WARNING) << "Dex location " << dex_location << " does not seem to include dex file. "
<< "Allow oat file use. This is potentially dangerous.";
}
}
if (accept_oat_file) {
source_oat_file = oat_file.release();
RegisterOatFile(source_oat_file);
}
}
}
std::vector<std::unique_ptr<const DexFile>> dex_files;
// Load the dex files from the oat file.
if (source_oat_file != nullptr) {
dex_files = oat_file_assistant.LoadDexFiles(*source_oat_file, dex_location);
if (dex_files.empty()) {
error_msgs->push_back("Failed to open dex files from "
+ source_oat_file->GetLocation());
}
}
// Fall back to running out of the original dex file if we couldn't load any
// dex_files from the oat file.
if (dex_files.empty()) {
if (oat_file_assistant.HasOriginalDexFiles()) {
if (Runtime::Current()->IsDexFileFallbackEnabled()) {
if (!DexFile::Open(dex_location, dex_location, &error_msg, &dex_files)) {
LOG(WARNING) << error_msg;
error_msgs->push_back("Failed to open dex files from " + std::string(dex_location));
}
} else {
error_msgs->push_back("Fallback mode disabled, skipping dex files.");
}
} else {
error_msgs->push_back("No original dex files found for dex location "
+ std::string(dex_location));
}
}
return dex_files;
}
```

还好是从后往前看的，最后一行注释，如果没有从`oat`文件中找到`dex`文件，我们就执行下面的代码！！`ohohohoh！！`

下面做了什么呢？

翻译翻译？

如果`dex`文件数组为空，调用`DexFile::Open(dex_location, dex_location, &error_msg, &dex_files)`，

如果返回值是`true`，则啥也不干，返回`dex_files`数组，程序正常结束，如果返回值是`false`，输出错误日志，往`error_msgs`中存入错误信息。

### 8. `DexFile::Open`

`xref`: `/art/runtime/dex_file.cc`

```c++
bool DexFile::Open(const char* filename, const char* location, std::string* error_msg,
std::vector<std::unique_ptr<const DexFile>>* dex_files) {
DCHECK(dex_files != nullptr) << "DexFile::Open: out-param is nullptr";
uint32_t magic;
ScopedFd fd(OpenAndReadMagic(filename, &magic, error_msg));
if (fd.get() == -1) {
DCHECK(!error_msg->empty());
return false;
}
if (IsZipMagic(magic)) {
return DexFile::OpenZip(fd.release(), location, error_msg, dex_files);
}
if (IsDexMagic(magic)) {
std::unique_ptr<const DexFile> dex_file(DexFile::OpenFile(fd.release(), location, true,error_msg));
if (dex_file.get() != nullptr) {
dex_files->push_back(std::move(dex_file));
return true;
} else {
return false;
}
}
*error_msg = StringPrintf("Expected valid zip or dex file: '%s'", filename);
return false;
}
```

`ok`，这段代码就是判断一下当前是什么文件，若是`dex`文件，则调用`DexFile::OpenFile(fd.release(), location, true,error_msg)`，构造`dex`文件，并将其放入文件数组中，返回值为文件数组。

### 9. `DexFile::OpenFile`

`xref`: `/art/runtime/dex_file.cc`
