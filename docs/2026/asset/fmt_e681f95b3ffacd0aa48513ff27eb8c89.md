
### 3. 进入`oat_file_assistant.cc`的`GetBestOatFile`与`LoadDexFiles`

先看`GetBestOatFile`，当`OatFileAssistant`初始化`status`时`Status`构造函数通过`GetFile`调用`OatFile::Open`打开`oatfile`：

```cpp
std::unique_ptr<OatFile> OatFileAssistant::GetBestOatFile() {

  return GetBestInfo().ReleaseFileForUse();

}



OatFileAssistant::OatFileInfo& OatFileAssistant::GetBestInfo() {

  // TODO(calin): Document the side effects of class loading when

  // running dalvikvm command line.

  if (dex_parent_writable_) {

    // If the parent of the dex file is writable it means that we can

    // create the odex file. In this case we unconditionally pick the odex

    // as the best oat file. This corresponds to the regular use case when

    // apps gets installed or when they load private, secondary dex file.

    // For apps on the system partition the odex location will not be

    // writable and thus the oat location might be more up to date.

    return odex_;

  }



  // We cannot write to the odex location. This must be a system app.



  // If the oat location is usable take it.

  if (oat_.IsUseable()) {

    return oat_;

  }



  // The oat file is not usable but the odex file might be up to date.

  // This is an indication that we are dealing with an up to date prebuilt

  // (that doesn't need relocation).

  if (odex_.Status() == kOatUpToDate) {

    return odex_;

  }



  // The oat file is not usable and the odex file is not up to date.

  // However we have access to the original dex file which means we can make

  // the oat location up to date.

  if (HasOriginalDexFiles()) {

    return oat_;

  }



  // We got into the worst situation here:

  // - the oat location is not usable

  // - the prebuild odex location is not up to date

  // - and we don't have the original dex file anymore (stripped).

  // Pick the odex if it exists, or the oat if not.

  return (odex_.Status() == kOatCannotOpen) ? oat_ : odex_;

}



OatFileAssistant::OatStatus OatFileAssistant::OatFileInfo::Status() {

  if (!status_attempted_) {

    status_attempted_ = true;

    const OatFile* file = GetFile();

    if (file == nullptr) {

      // Check to see if there is a vdex file we can make use of.

      std::string error_msg;

      std::string vdex_filename = GetVdexFilename(filename_);

      std::unique_ptr<VdexFile> vdex = VdexFile::Open(vdex_filename,

                                                      /*writeable*/false,

                                                      /*low_4gb*/false,

                                                      /*unquicken*/false,

                                                      &error_msg);//这里在Status中先打开vdex，vdex也是版本新增的，可以加快启动速度

      if (vdex == nullptr) {

        status_ = kOatCannotOpen;

        VLOG(oat) << "unable to open vdex file " << vdex_filename << ": " << error_msg;

      } else {

        if (oat_file_assistant_->DexChecksumUpToDate(*vdex, &error_msg)) {

          // The vdex file does not contain enough information to determine

          // whether it is up to date with respect to the boot image, so we

          // assume it is out of date.

          VLOG(oat) << error_msg;

          status_ = kOatBootImageOutOfDate;

        } else {

          status_ = kOatDexOutOfDate;

        }

      }

    } else {

      status_ = oat_file_assistant_->GivenOatFileStatus(*file);

      VLOG(oat) << file->GetLocation() << " is " << status_

          << " with filter " << file->GetCompilerFilter();

    }

  }

  return status_;

}



const OatFile* OatFileAssistant::OatFileInfo::GetFile() {

  CHECK(!file_released_) << "GetFile called after oat file released.";

  if (!load_attempted_) {

    load_attempted_ = true;

    if (filename_provided_) {

      std::string error_msg;

      file_.reset(OatFile::Open(filename_.c_str(),

                                filename_.c_str(),

                                nullptr,

                                nullptr,

                                oat_file_assistant_->load_executable_,

                                /*low_4gb*/false,

                                oat_file_assistant_->dex_location_.c_str(),

                                &error_msg));

      if (file_.get() == nullptr) {

        VLOG(oat) << "OatFileAssistant test for existing oat file "

          << filename_ << ": " << error_msg;

      }

    }

  }

  return file_.get();

}
```

再看`LoadDexFiles`，先通过`GetOatDexFile`得到上面 open 的`OatFile`的`OatDexFile`，再通过`OpenDexFile`打开获得`OatDexFile`的`DexFile`。这里如果有多个 dex，会通过 for 循环获得多个`dex_file`，组装出`dex_files`。

```cpp
std::vector<std::unique_ptr<const DexFile>> OatFileAssistant::LoadDexFiles(

    const OatFile& oat_file, const char* dex_location) {

  std::vector<std::unique_ptr<const DexFile>> dex_files;



  // Load the main dex file.

  std::string error_msg;

  const OatFile::OatDexFile* oat_dex_file = oat_file.GetOatDexFile(

      dex_location, nullptr, &error_msg);//这句oat_file通过获得oat_dex_file

  if (oat_dex_file == nullptr) {

    LOG(WARNING) << error_msg;

    return std::vector<std::unique_ptr<const DexFile>>();

  }



  std::unique_ptr<const DexFile> dex_file = oat_dex_file->OpenDexFile(&error_msg);//通过oat_dex_file获得dex_file

  if (dex_file.get() == nullptr) {

    LOG(WARNING) << "Failed to open dex file from oat dex file: " << error_msg;

    return std::vector<std::unique_ptr<const DexFile>>();

  }

  dex_files.push_back(std::move(dex_file));//把dex_file放入dex_files



  // Load the rest of the multidex entries

  for (size_t i = 1; ; i++) {

    std::string multidex_dex_location = DexFile::GetMultiDexLocation(i, dex_location);

    oat_dex_file = oat_file.GetOatDexFile(multidex_dex_location.c_str(), nullptr);//如果不是一个dex，通过循环加载其他的oat_dex_file

    if (oat_dex_file == nullptr) {

      // There are no more multidex entries to load.

      break;

    }



    dex_file = oat_dex_file->OpenDexFile(&error_msg);//通过oat_dex_file的OpenDexFile获得其余dex_file

    if (dex_file.get() == nullptr) {

      LOG(WARNING) << "Failed to open dex file from oat dex file: " << error_msg;

      return std::vector<std::unique_ptr<const DexFile>>();

    }

    dex_files.push_back(std::move(dex_file));//把multidex放入dex_files

  }

  return dex_files;

}
```

### 4. 再往下

就是`OatFile::Open`、`GetOatDexFile`和`OpenDexFile`，这 3 个函数就进入了`oat_file.cc`中。`OatFile::Open`打开`oat_file`比较复杂，我们下一篇再分析，假设它很愉快完成了，得到了`oat_file`，后面通过`GetOatDexFile`操作它得到`oat_dex_file`，再通过`OpenDexFile`操作`OatDexFile`最终获得`dex_file`返回。这三个类互相关联，`OatFile`包含`OatDexFile`包含`DexFile`，阅读`oat_file.cc`源码可以搞明白它们的数据结构差异。

#### 4.1 先看`OatFile::Open`

先试图通过`OatFileBase::OpenOatFile<DlOpenOatFile>`打开`oat_file`。我们知道 art 分为 quick 和 portable 两种优化模式，这里先假设它是 portable 尝试打开 oat，失败就通过`OatFileBase::OpenOatFile<ElfOatFile>`打开`oat_file`，这就是 quick 模式了，和 5.0 以前版本的先判断 quick 还是 portable 模式再选择函数不太一样。oat 文件的加载就在这个`OpenOatFile`函数中，下一篇我们重点分析 oat 加载流程。
