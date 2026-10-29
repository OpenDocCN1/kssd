
# [原创]菜鸟学8.1版本dex加载流程笔记--第三篇: `OatFile::Open`流程与`OatDexFile`的获得

菜鸟最近破事比较多，磕磕绊绊总算把`oat_file.cc`大致流程看完了，论坛记录笔记方便以后查询。

这个函数重点就是如何打开`oat_file`文件，然后通过解析oat文件构建出`oat_dex_file`数据结构，这个`oat_dex_file`存储了完整的dex信息，如果走通过`oat_file`获得`dex_file`这条路，从`OpenDexFilesFromOat`直到`DexFile::Open`，就是主要通过解析`oat_dex_file`的数据结构获得`dex_file`的。

依然先贴一下流程，一些不重要的函数就省略了

```
 OatFile::Open

{

 GetVdexFilename

 OatFileBase::OpenOatFile<DlOpenOatFile> or OatFileBase::OpenOatFile<ElfOatFile>

 {

  PreLoad

  LoadVdex

   {

    VdexFile::Open

   }

  Load

   {

    Dlopen

   }

  ComputeFields

   {

    FindDynamicSymbolAddress&oatdata，oatlastword，oatbss，oatbsslastword，oatbssmethods，oatbssroots

   }

  PreSetup

   {

    dl_iterate_phdr(dl_iterate_context::callback,&context)//遍历所有elf获得信息后调用callback函数map映射oat文件

   }

  Setup

   {

    GetOatHeader

    GetInstructionSetPointerSize

    GetOatDexFilesOffset//这里达到了OatDexFile的起始

    GetDexFileCount

	 ReadOatDexFileData&dex_file_location_size//

	 ResolveRelativeEncodedDexLocation

    ReadOatDexFileData&dex_file_checksum

    ReadOatDexFileData&dex_file_offset

    ReadOatDexFileData&class_offsets_offset

    ReadOatDexFileData&lookup_table_offset//加快类查找速度

    ReadOatDexFileData&dex_layout_sections_offset

	 ReadOatDexFileData&method_bss_mapping_offset

	 FindDexFileMapItem&call_sites_item//调用站点标识符 

    new OatDexFile

   }

 }

}
```

1. 不管`DlOpenOatFile`还是`ElfOatFile`，都进入`OpenOatFile`，依次调用了，用于后面获得dex和调用等，流程比较清楚

```
OatFileBase* OatFileBase::OpenOatFile(const std::string& vdex_filename,

                                      const std::string& elf_filename,

                                      const std::string& location,

                                      uint8_t* requested_base,

                                      uint8_t* oat_file_begin,

                                      bool writable,

                                      bool executable,

                                      bool low_4gb,

                                      const char* abs_dex_location,

                                      std::string* error_msg) {

  std::unique_ptr<OatFileBase> ret(new kOatFileBaseSubType(location, executable));//不管是DlOpenOatFile还是ElfOatFile都先转换成OatFileBase指针



  ret->PreLoad();



  if (kIsVdexEnabled && !ret->LoadVdex(vdex_filename, writable, low_4gb, error_msg)) {

    return nullptr;

  }



  if (!ret->Load(elf_filename,

                 oat_file_begin,

                 writable,

                 executable,

                 low_4gb,

                 error_msg)) {

    return nullptr;

  }



  if (!ret->ComputeFields(requested_base, elf_filename, error_msg)) {

    return nullptr;

  }



  ret->PreSetup(elf_filename);



  if (!ret->Setup(abs_dex_location, error_msg)) {

    return nullptr;

  }



  return ret.release();

}
```

2. 先看`PreLoad`，通过`dl_iterate_phdr`遍历所有加载的elf对象获得它们的`dl_phdr_info`，每次循环`count+1`，然后把`count`存储在`shared_objects_before_`，下面`PreSetup`会使用`shared_objects_before_`这个变量。

这里重点关注一下结构`dl_phdr_info`，存储了elf的`address`、`name`、Pointer to array of ELF program headers等几个重要字段。

这里的`struct dl_iterate_context`只有一个`count`字段，用于存储计数遍历的elf对象，`callback`功能也比较简单，下面`PreSetup`还有一个`dl_iterate_context`结构，它的`callback`函数就比较复杂了，遍历并且映射了`oat_file`的program segments。

```
void DlOpenOatFile::PreLoad() {

#ifdef __APPLE__

  UNUSED(shared_objects_before_);

  LOG(FATAL) << "Should not reach here.";

  UNREACHABLE();

#else

  // Count the entries in dl_iterate_phdr we get at this point in time.//遍历所有elf的phdr

  struct dl_iterate_context {

    static int callback(struct dl_phdr_info *info ATTRIBUTE_UNUSED,

                        size_t size ATTRIBUTE_UNUSED,

                        void *data) {



// struct dl_phdr_info {

    // ElfW(Addr)        dlpi_addr;  /* Base address of object */

    // const char       *dlpi_name;  /* (Null-terminated) name of

                                     // object */

    // const ElfW(Phdr) *dlpi_phdr;  /* Pointer to array of

                                     // ELF program headers

                                     // for this object */

    // ElfW(Half)        dlpi_phnum; /* # of items in dlpi_phdr */

// }							





      reinterpret_cast<dl_iterate_context*>(data)->count++;//每次循环count自增

      return 0;  // Continue iteration.

    }

    size_t count = 0;

  } context;



  dl_iterate_phdr(dl_iterate_context::callback, &context);//遍历所有elf对象获得dl_phdr_info并调用callback，这里的callback就是count自增1

  shared_objects_before_ = context.count; //把count最终值存储到shared_objects_before_ 

#endif

}
```

3. 然后`LoadVdex`，最终调用了`VdexFile::Open`，这里的`vdex`是8.0以后的新变化，原先存储在oat里的`dexfile`现在似乎被quickene后放在`vdex`里，组合`oat_file`和`vdex_`才能获得完整的`oat_dex_file`。

```
bool OatFileBase::LoadVdex(const std::string& vdex_filename,

                           bool writable,

                           bool low_4gb,

                           std::string* error_msg) {

  vdex_ = VdexFile::Open(vdex_filename, writable, low_4gb, /* unquicken*/ false, error_msg);//打开并获得vdex_

  if (vdex_.get() == nullptr) {

    *error_msg = StringPrintf("Failed to load vdex file '%s' %s",

                              vdex_filename.c_str(),

                              error_msg->c_str());

    return false;

  }

  return true;

}
```

`vdex`简单结构`vdex_file.h`，包含`dex_files`和`QuickeningInfo`：

```
// File format:

//   VdexFile::Header    fixed-length header

//

//   DEX[0]              array of the input DEX files

//   DEX[1]              the bytecode may have been quickened

//   ...

//   DEX[D]

//   QuickeningInfo

//     uint8[]                     quickening data

//     unaligned_uint32_t[2][]     table of offsets pair:

//                                    uint32_t[0] contains code_item_offset

//                                    uint32_t[1] contains quickening data offset from the start

//                                                of QuickeningInfo

//     unalgined_uint32_t[D]       start offsets (from the start of QuickeningInfo) in previous

//                                 table for each dex file
```

4. 下面是`Load`函数，最终调用了`Dlopen`加载oat，获得`dlopen_handle_`。

```
bool DlOpenOatFile::Load(const std::string& elf_filename,

                         uint8_t* oat_file_begin,

                         bool writable,

                         bool executable,

                         bool low_4gb,

                         std::string* error_msg) {

  // Use dlopen only when flagged to do so, and when it's OK to load things executable.

  // TODO: Also try when not executable? The issue here could be re-mapping as writable (as

  //       !executable is a sign that we may want to patch), which may not be allowed for

  //       various reasons.

  if (!kUseDlopen) {

    *error_msg = "DlOpen is disabled.";

    return false;

  }

  if (low_4gb) {

    *error_msg = "DlOpen does not support low 4gb loading.";

    return false;

  }

  if (writable) {

    *error_msg = "DlOpen does not support writable loading.";

    return false;

  }

  if (!executable) {

    *error_msg = "DlOpen does not support non-executable loading.";

    return false;

  }



  // dlopen always returns the same library if it is already opened on the host. For this reason

  // we only use dlopen if we are the target or we do not already have the dex file opened. Having

  // the same library loaded multiple times at different addresses is required for class unloading

  // and for having dex caches arrays in the .bss section.

  if (!kIsTargetBuild) {

    if (!kUseDlopenOnHost) {

      *error_msg = "DlOpen disabled for host.";

      return false;

    }

  }



  bool success = Dlopen(elf_filename, oat_file_begin, error_msg);//调用Dlopen加载oat，获得dlopen_handle_

  DCHECK(dlopen_handle_ != nullptr || !success);



  return success;

}
```

看一下`Dlopen`，最终调用了`android_dlopen_ext`或者`dlopen`。
