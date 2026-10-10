

# mimisys pack v0.1

> 原文：[`www.pediy.com/kssd/pediy10/63403.html`](https://www.pediy.com/kssd/pediy10/63403.html)

mimisys pack 是一个驱动简单文件壳。  
驱动文件的处理比 r3 简单很多，许多不要关心，但还有几点比 r3 不同，注意一下。  
以前也做过一个 sys 加密壳，但是没有代码组织好，导致稳定性极差，这个版本是完全  
重写的版本，没有做很多复杂的东西，只做文件的压缩。  
  
附件是源码  
  

代码:

```
class mimisys\_loader
{
public:
  mimisys\_loader(mimisys\_ldr\_param\*,PDRIVER\_OBJECT, PUNICODE\_STRING);
  ~mimisys\_loader();

  ulong  init\_env();
  ulong  run();

private:
  ulong get\_base(const char\* );
  ulong get\_sys\_api(const char\* );

  static ulong get\_api(ulong, const char\* );
  static char\* make\_str(char\*, int, ...);
  static int strcmp(const char \*, const char \*);
  static int strnicmp(const wchar\_t\*, const char\*, size\_t);
  static void\* memcpy(void \*, const void \*, size\_t);
  static void\* memset(void\*, int, size\_t);
  static int tolower(int );

  static size\_t decompress(const ubyte\*, size\_t, ubyte\*, size\_t);

//   template <class T> T addr(T v)
//   {
//     return (T)((ulong)v + \_self\_diff);
//   }
//   
  ulong fixsec();
  ulong fixrel();
  ulong fiximp();
  ulong fixexp();
  ulong goto\_ep();

  ubyte\* get\_p1(ulong rva); // \_unpacked\_data
  ubyte\* get\_p2(ulong rva);

private:
  mimisys\_ldr\_param\*    \_ldr\_prm;
  ulong          \_self\_diff;
  
  PDRIVER\_OBJECT      \_drv\_obj;
  PUNICODE\_STRING      \_reg\_path;
  PLDR\_DATA\_TABLE\_ENTRY  \_self\_dte;
  
  union {
    PIMAGE\_DOS\_HEADER  \_dos\_hdr;
    ulong        \_img\_base;
  };
  PIMAGE\_NT\_HEADERS    \_nt\_hdr;
  
  PIMAGE\_SECTION\_HEADER  \_sect\_hdr;
  
  ubyte\*          \_unpacked\_data;
  char          \_sys\_fname\[5\]\[20\];

  PFN\_DbgPrint      \_DbgPrint;
  PFN\_ExAllocatePool    \_ExAllocatePool;
  PFN\_ExFreePool      \_ExFreePool;
};


class mimisys\_packer
{
public:
  mimisys\_packer();
  ~mimisys\_packer();

  bool  open(const char\* fpath);
  void  close();
  
  bool  is\_open() const;
  bool  pack(const char\* save\_path, ulong opt);

protected:
  bool  init\_ctx();

  bool  compress\_sections();
  bool  encrypt\_import();
  bool  save\_after\_add\_ldr(const char\* save\_path, ulong opt);

  bool  copyres\_to(ulong new\_rva);

  bool  check\_sect\_hdr(PIMAGE\_SECTION\_HEADER sect\_hdr);
  ulong  raw(ulong v);
  
private:
  char  \_fpath\[MAX\_PATH\];
  ubyte\*  \_fdata;
  size\_t  \_fsize;

  ulong  \_nsect;
  ulong  \_img\_size;

  ulong  \_unpacked\_size;
  ulong  \_packed\_size;
  ubyte\*  \_packed\_data;

  ulong  \_res\_rva;
  ulong  \_res\_size;

  ulong  \_rel\_rva;
  ulong  \_rel\_size;

  ulong  \_imp\_rva;
  ulong  \_imp\_size;

  bool  \_b\_page\_alig;
  mimisys\_ldr\_param \_ldr\_pm;

  PIMAGE\_DOS\_HEADER    \_dos\_hdr;
  PIMAGE\_NT\_HEADERS    \_nt\_hdr;
  PIMAGE\_SECTION\_HEADER  \_sect\_hdr;
  
  PIMAGE\_IMPORT\_DESCRIPTOR  \_imp;
  PIMAGE\_BASE\_RELOCATION    \_rel;
  PIMAGE\_RESOURCE\_DIRECTORY  \_res;
};

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || MimiSys.rar  
 |