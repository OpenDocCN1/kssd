
# mimisys pack v0.1

> 原文：[`www.pediy.com/kssd/pediy10/63403.html`](https://www.pediy.com/kssd/pediy10/63403.html)

`mimisys pack` 是一个驱动简单文件壳。驱动文件的处理比 `r3` 简单很多，许多不要关心，但还有几点比 `r3` 不同，注意一下。

以前也做过一个 `sys` 加密壳，但是没有代码组织好，导致稳定性极差，这个版本是完全重写的版本，没有做很多复杂的东西，只做文件的压缩。

附件是源码

代码:

```
class mimisys_loader
{
public:
  mimisys_loader(mimisys_ldr_param*,PDRIVER_OBJECT, PUNICODE_STRING);
  ~mimisys_loader();

  ulong  init_env();
  ulong  run();

private:
  ulong get_base(const char* );
  ulong get_sys_api(const char* );

  static ulong get_api(ulong, const char* );
  static char* make_str(char*, int, ...);
  static int strcmp(const char *, const char *);
  static int strnicmp(const wchar_t*, const char*, size_t);
  static void* memcpy(void *, const void *, size_t);
  static void* memset(void*, int, size_t);
  static int tolower(int );

  static size_t decompress(const ubyte*, size_t, ubyte*, size_t);

//   template <class T> T addr(T v)
//   {
//     return (T)((ulong)v + _self_diff);
//   }
//    
  ulong fixsec();
  ulong fixrel();
  ulong fiximp();
  ulong fixexp();
  ulong goto_ep();

  ubyte* get_p1(ulong rva); // _unpacked_data
  ubyte* get_p2(ulong rva);

private:
  mimisys_ldr_param*    _ldr_prm;
  ulong          _self_diff;
  
  PDRIVER_OBJECT      _drv_obj;
  PUNICODE_STRING      _reg_path;
  PLDR_DATA_TABLE_ENTRY  _self_dte;
  
  union {
    PIMAGE_DOS_HEADER  _dos_hdr;
    ulong        _img_base;
  };
  PIMAGE_NT_HEADERS    _nt_hdr;
  
  PIMAGE_SECTION_HEADER  _sect_hdr;
  
  ubyte*          _unpacked_data;
  char          _sys_fname[5][20];

  PFN_DbgPrint      _DbgPrint;
  PFN_ExAllocatePool    _ExAllocatePool;
  PFN_ExFreePool      _ExFreePool;
};


class mimisys_packer
{
public:
  mimisys_packer();
  ~mimisys_packer();

  bool  open(const char* fpath);
  void  close();
  
  bool  is_open() const;
  bool  pack(const char* save_path, ulong opt);

protected:
  bool  init_ctx();

  bool  compress_sections();
  bool  encrypt_import();
  bool  save_after_add_ldr(const char* save_path, ulong opt);

  bool  copyres_to(ulong new_rva);

  bool  check_sect_hdr(PIMAGE_SECTION_HEADER sect_hdr);
  ulong  raw(ulong v);
  
private:
  char  _fpath[MAX_PATH];
  ubyte*  _fdata;
  size_t  _fsize;

  ulong  _nsect;
  ulong  _img_size;

  ulong  _unpacked_size;
  ulong  _packed_size;
  ubyte*  _packed_data;

  ulong  _res_rva;
  ulong  _res_size;

  ulong  _rel_rva;
  ulong  _rel_size;

  ulong  _imp_rva;
  ulong  _imp_size;

  bool  _b_page_alig;
  mimisys_ldr_param _ldr_pm;

  PIMAGE_DOS_HEADER    _dos_hdr;
  PIMAGE_NT_HEADERS    _nt_hdr;
  PIMAGE_SECTION_HEADER  _sect_hdr;
  
  PIMAGE_IMPORT_DESCRIPTOR  _imp;
  PIMAGE_BASE_RELOCATION    _rel;
  PIMAGE_RESOURCE_DIRECTORY  _res;
};
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || MimiSys.rar
|
