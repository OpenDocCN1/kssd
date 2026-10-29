
分析完`ZipArchive::OpenFromFd`后，继续回到`dex2oat`中，21 行调用`DexFile::Open`函数，该函数定义在`/art/runtime/dex_file.cc`源文件中，注意有两个`DexFile::Open`函数，一个是`const DexFile* DexFile::Open(const std::string& filename, const std::string& location)`，另一个是`const DexFile* DexFile::Open(const ZipArchive& zip_archive, const std::string& location)`，这里显然调用的是后者，部分代码如下：

``` 
1. const DexFile* DexFile::Open(const ZipArchive& zip_archive, const std::string&
location) {
2. CHECK(!location.empty());
3. UniquePtr<ZipEntry> zip_entry(zip_archive.Find(kClassesDex));
4. ......
5. UniquePtr<MemMap> map(zip_entry->ExtractToMemMap(kClassesDex));
6. ......
7. UniquePtr<const DexFile> dex_file(OpenMemory(location, zip_entry->GetCrc32(),
map.release()));
8. ......
9. if (!DexFileVerifier::Verify(dex_file.get(), dex_file->Begin(),
dex_file->Size())) {
10. LOG(ERROR) << "Failed to verify dex file '" << location << "'";
11. return NULL;
12. }
13. if (!dex_file->DisableWrite()) {
14. LOG(ERROR) << "Failed to make dex file read only '" << location << "'";
15. return NULL;
16. }
17. CHECK(dex_file->IsReadOnly()) << location;
18. return dex_file.release();
19. }
```

第 3 行调用`ZipArchive`类的`Find`函数，以文件名为 Key 找到对应 ZIP 文件中的文件并返回`ZipEntry`实例，该`ZipEntry`实例对应的内容为 Central Directory Header，其中`kClasssesDex`定义在`dex_file.cc`源码文件的第 208 行，值为`classes.dex`，也就是从`/system/framework/javax.obex.jar`(JAR 文件实际上也是一个 ZIP 文件)文件中找到`classes.dex`的文件。可以简单看下从 Nexus4 上 pull 下来的`javax.obex.jar`，将该文件解压缩，得到的文件列表如图 12 所示，确实存在一个`classes.dex`的文件。第 5 行调用`ZipEntry`的`ExtractToMemMap`函数，该函数部分代

![](https://bbs.kanxue.com/upload/attach/201401/444052_hh2o7xxfhjvsosf.png)

**图 12 javax.obex.jar 解压后文件**

码如下：

``` 
1. MemMap* ZipEntry::ExtractToMemMap(const char* entry_filename) {
2. std::string name(entry_filename);
3. name += " extracted in memory from ";
4. name += entry_filename;
5. UniquePtr<MemMap> map(MemMap::MapAnonymous(name.c_str(),NULL,
6. GetUncompressedLength(), PROT_READ | PROT_WRITE));
7. ......
8. bool success = ExtractToMemory(map->Begin(), map->Size());
9. ......
10. return map.release();
11. }
```

2-4 行做字符串拼接，最终字符串值为`classes.dex extracted in memory from classes.dex`；第 5 行调用`MemMap`类的`MapAnonymous`(`/art/runtime/mem_map.cc`)创建`MemMap`的实例，其中`GetUncompressedLength`的函数代码如下，很简单，就是从 Central Directory Header 中获得

``` 
1. uint32_t ZipEntry::GetUncompressedLength() {
2. return Le32ToHost(ptr_ + ZipArchive::kCDEUncompLen);
3. }
```

uncompressed size 的值。回到`ExtractToMemMap`函数，第 8 行调用`ExtractToMemory`，该函数代码如下：

``` 
1. bool ZipEntry::ExtractToMemory(uint8_t* begin, size_t size) {
2. ......
3. off64_t data_offset = GetDataOffset();
4. ......
5. if (lseek64(zip_archive_->fd_, data_offset, SEEK_SET) != data_offset) {
6. PLOG(WARNING) << "Zip: lseek to data at " << data_offset << " failed";
7. return false;
8. }
9. switch (GetCompressionMethod()) {
10. case kCompressStored:
11. return CopyFdToMemory(begin, size, zip_archive_->fd_,
GetUncompressedLength());
12. case kCompressDeflated:
13. return InflateToMemory(begin, size, zip_archive_->fd_,
14. GetUncompressedLength(), GetCompressedLength());
15. default:
16. LOG(WARNING) << "Zip: unknown compression method " << std::hex <<
GetCompressionMethod();
17. return false;
18. }
19. }
```

第 3 行调用`GetDataOffset`函数，该函数的作用就是从 Central Directory Header 中找到 Local File Header 的偏移量，然后从 Local File Header 中定位到 File Data 的位置，该函数部分代码如下(已忽略所有文件格式合法性检查)：

``` 
1. off64_t ZipEntry::GetDataOffset() {
2. ......
3. int64_t lfh_offset = Le32ToHost(ptr_ + ZipArchive::kCDELocalOffset);
4. ......
5. if (lseek64(zip_archive_->fd_, lfh_offset, SEEK_SET) != lfh_offset) {
6. PLOG(WARNING) << "Zip: failed seeking to LFH at offset " << lfh_offset;
7. return -1;
8. }
9. uint8_t lfh_buf[ZipArchive::kLFHLen];
10. ssize_t actual = TEMP_FAILURE_RETRY(read(zip_archive_->fd_, lfh_buf,
sizeof(lfh_buf)));
11. ......
12. off64_t data_offset = (lfh_offset + ZipArchive::kLFHLen
13. + Le16ToHost(lfh_buf + ZipArchive::kLFHNameLen)
14. + Le16ToHost(lfh_buf + ZipArchive::kLFHExtraLen));
15. ......
16. return data_offset;
17. }
```

第 3 行获得该 Central Directory Header 对应的 File Local Header 的偏移量；第 5 行将`zip_archive_`的`fd_`定位到该偏移处；第 10 行从偏移处开始读取内容到`lfh_buf`中；第 12 行从 File Local Header 中获得 File Data 的偏移，可以参考 Local File Header 的结构来看第 12 行的代码；最后返回`data_offset`值。

回到`ExtractToMemory`，第 5 行根据 File Data 的偏移量，将 ZIP 文件的文件描述符定位到 File Data 起始位置；第 9 行进入 switch 语句，`GetCompressionMethod`从 Central Directory Header 中获得 compression method 字段值，那么`javax.obex.jar`中`classes.dex`文件该值是多少呢？使用 010Editor 可以发现该值为 0x0008，如图 13 所示。而`kCompressStored`表示 File Data 没有压缩，仅仅是存储，其值为 0，`kCompressDeflated`表示 File Data 经过压缩，其值为 8(两者都定义在`/art/runtime/zip_archive.h`中)，因此 13 行的`InflateToMemory`函数会被调用，该函数主

![](https://bbs.kanxue.com/upload/attach/201401/444052_ith2z2o85eg0ipy.png)

**图 13 javax.obex.jar 中 classes.dex 的 compression method 值**

要完成 File Data 的解压工作，大家可以自行分析。

回到`DexFile::Open`函数中，至此已经完成了`classes.dex`文件的解压并映射到了内存中。第 7 行调用`OpenMemory`函数，该函数代码如下：

``` 
1. const DexFile* DexFile::OpenMemory(const byte* base,size_t size,
2. const std::string& location,uint32_t location_checksum, MemMap* mem_map) {
3. CHECK_ALIGNED(base, 4);
4. UniquePtr<DexFile> dex_file(new DexFile(base, size, location, location_checksum,
mem_map));
5. if (!dex_file->Init()) {
6. return NULL;
7. } else {
8. return dex_file.release();
9. }
10. }
```

第 4 行创建`DexFile`类的实例，完成部分成员变量的初始化，`DexFile`的构造函数位于`/art/runtime/dex_file.h`头文件中；第 5 行调用`Init`函数完成初始化，如 DEX 文件的`StringId`，`TypeId`的初始化，以及 Dex 文件的一些合法性检查工作。

`DexFile::Open`函数第 9 行调用`DexFileVerifier::Verify`函数对文件进行 DEX 合法性验证；13 行设置文件禁止写；17 行检查文件是否只读，也就是进一步判断设置禁止写是否成功。

回到`dex2oat`中，在 26 行调用`dex_files.push_back(dex_file)`将创建的`DexFile`实例存入`vector`中；第 30-31 行将 DEX 文件设置为可写。

总结下该部分的工作，主要完成了从`javax.obex.jar`文件中提取`classes.dex`并映射到内存，同时完成了 DEX 文件合法性的验证工作。

## 创建 OAT 文件

继续分析`dex2oat`的代码，接着会创建 OAT 文件，代码如下：

``` 
1. if (!image && (Runtime::Current()->GetCompilerFilter() !=
Runtime::kInterpretOnly)) {
2. size_t num_methods = 0;
3. for (size_t i = 0; i != dex_files.size(); ++i) {
4. const DexFile* dex_file = dex_files[i];
5. CHECK(dex_file != NULL);
6. num_methods += dex_file->NumMethodIds();
7. }
8. if (num_methods <= Runtime::Current()->GetNumDexMethodsThreshold()) {
9. Runtime::Current()->SetCompilerFilter(Runtime::kSpeed);
10. VLOG(compiler) << "Below method threshold, compiling anyways";
11. }
12. }
13. UniquePtr<const CompilerDriver>
compiler(dex2oat->CreateOatFile(boot_image_option,host_prefix.get(),android_roo
t,is_host, dex_files,oat_file.get(), bitcode_filename, image, image_classes,
dump_stats,timings));
```

第 2-10 行主要从 DEX 文件中获得函数个数并判断是否达到阈值；13 行调用`CreateOatFile`函数创建 OAT 文件，创建 OAT 文件的过程过于繁琐，就不再在此一一罗列分析(主要是我不想再往坑里跳了~~)，其实 OAT 部分的内容是嵌在 ELF 格式的文件中的，Linux 下使用`file`命令来检验下`/system/framework/javax.obex.jar`转换后的文件`/data/dalvik-cache/system@framework@javax.objex.jar@classes.dex`，如图 14 所示。如果顺着代码逻辑走下去的话，在`/art/compiler/elf_writer_quick.cc`文件中(当未定义`ART_USE_PORTABLE_COMPILER`宏时会进入`ElfWriterQuick::Create`函数)的`ElfWriteQuick::Write`函数中有一个文件格式的说明，参考

[2a5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4y4h3k6J5x3g2)9J5c8Y4S2J5k6h3k6Q4x3V1k6S2M7Y4c8Q4x3V1k6U0L8$3#2H3K9h3I4W2M7W2)9J5c8X3g2D9k6W2)9#2k6Y4N6J5K9i4c8W2M7W2)9#2k6Y4q4#2K9h3y4C8i4K6u0W2j5$3y4Q4x3U0x3@1y4R3`.`.](https://bbs.kanxue.com/elink@e46K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4y4h3k6J5x3g2)9J5c8Y4S2J5k6h3k6Q4x3V1k6S2M7Y4c8Q4x3V1k6U0L8$3#2H3K9h3I4W2M7W2)9J5c8X3g2D9k6W2)9#2k6Y4N6J5K9i4c8W2M7W2)9#2k6Y4q4#2K9h3y4C8i4K6u0W2j5$3y4Q4x3U0x3@1y4R3`.`.)。而关于 OAT 部分的内容
