
# NdisWrapper 框架分析

> 原文：[`www.pediy.com/kssd/pediy12/144780.html`](https://www.pediy.com/kssd/pediy12/144780.html)

boywhp@gmail.com  
www.opscn.com  

## ndiswrapper 简介

一些 usb 无线网卡厂商只提供 windows 下的驱动，那么你可以直接使用 ndiswrapper 来执行 windows 上的网卡驱动，是不是很 cool 呢？其实也算是 linux 不如 windows 流行的一个证据。

ndiswrapper 项目主页：

[`sourceforge.net/apps/mediawik...itle=Main\_Page`](http://sourceforge.net/apps/mediawiki/ndiswrapper/index.php?title=Main_Page)  

## 入口

入口：`wrapper.c`

```
module_init(wrapper_init);
  if (wrapmem_init() || ntoskernel_init() || ndis_init() ||
      wrapndis_init() || usb_init() || wrap_procfs_init() ||
      loader_init())
```

完成各个子模块初始化工作。

## 重点分析：loader.c

重点在 `loader.c` `int loader_init(void)`

该函数首先 `misc_register` 创建一个杂项设备，然后调用 `register_devices` 注册设备

`static void register_devices(void)`，该函数会依次注册 PCI 以及 USB 驱动

```
pci_register_driver(&wrap_pci_driver);
usb_register(&wrap_usb_driver);
```

### USB 驱动注册

以 USB 为例：

```
static struct usb_driver wrap_usb_driver = {
  .name = DRIVER_NAME,
  .id_table = wrap_usb_id_table,
  .probe = wrap_pnp_start_usb_device,
  .disconnect = __devexit_p(wrap_pnp_remove_usb_device),
  .suspend = wrap_pnp_suspend_usb_device,
  .resume = wrap_pnp_resume_usb_device,
};
```

首先看 usb 设备探测处理，`pnp.c` `wrap_pnp_start_usb_device`

```
load_device.bus = WRAP_USB_BUS;
load_device.vendor = le16_to_cpu(udev->descriptor.idVendor);
load_device.device = le16_to_cpu(udev->descriptor.idProduct);
load_device.subvendor = 0;
load_device.subdevice = 0;
wd = load_wrap_device(&load_device);
```

成功后调用 `ret = wrap_pnp_start_device(wd);` 启动设备

### 加载设备

`loader.c`

```
struct wrap_device *load_wrap_device(struct load_device *load_device)
char *argv[] = {"loadndisdriver", WRAP_CMD_LOAD_DEVICE,
#if DEBUG >= 1
    "1",
#else
    "0",
#endif
  UTILS_VERSION, vendor, device,
  subvendor, subdevice, bus, NULL};
  char *env[] = {NULL};
ret = call_usermodehelper("/sbin/loadndisdriver", argv, env, 1);
```

可见 ndiswrapper 会通过 `call_usermodehelper` 来处理驱动的加载，主要是需要在 ring3 下做一些参数配置之类的东西，`loadndisdriver` 通过 `ioctl` 接口同内核模块通信。

### ioctl 接口

具体实现在 `loader.c` `wrapper_ioctl` 函数

```
static int wrapper_ioctl(struct inode *inode, struct file *file,
      unsigned int cmd, unsigned long arg)
```

以 `WRAP_IOCTL_LOAD_DRIVER` 为例，我们看 ndiswrapper 是如何加载 windows 驱动的。

```
case WRAP_IOCTL_LOAD_DRIVER:
  load_driver = vmalloc(sizeof(*load_driver));
  if (copy_from_user(load_driver, addr, sizeof(*load_driver)))
    ret = -EFAULT;
  else
    ret = load_user_space_driver(load_driver);
```

首先从 R3 程序复制参数，然后调用 `load_user_space_driver`

### 加载用户空间驱动

`loader.c` `load_user_space_driver`

```
static int load_user_space_driver(struct load_driver *load_driver)
  drv_obj = allocate_object(sizeof(*drv_obj), OBJECT_TYPE_DRIVER, NULL);
  drv_obj->drv_ext = kzalloc(sizeof(*(drv_obj->drv_ext)), GFP_KERNEL);
  InitializeListHead(&drv_obj->drv_ext->custom_ext);
  IoAllocateDriverObjectExtension(drv_obj,
            (void *)WRAP_DRIVER_CLIENT_ID,
            sizeof(*wrap_driver),
            (void **)&wrap_driver);
  memset(wrap_driver, 0, sizeof(*wrap_driver));
  InitializeListHead(&wrap_driver->list);
  InitializeListHead(&wrap_driver->settings);
  wrap_driver->drv_obj = drv_obj;
  RtlInitAnsiString(&ansi_reg, "/tmp");
  RtlAnsiStringToUnicodeString(&drv_obj->name, &ansi_reg, TRUE);
  strncpy(wrap_driver->name, load_driver->name, sizeof(wrap_driver->name));
  wrap_driver->name[sizeof(wrap_driver->name)-1] = 0;
  if (load_sys_files(wrap_driver, load_driver) ||
      load_bin_files_info(wrap_driver, load_driver) ||
      load_settings(wrap_driver, load_driver) ||
      start_wrap_driver(wrap_driver) ||
      add_wrap_driver(wrap_driver)) {
```

该函数完成 windows 的 ndis 驱动实际加载以及启动。

### 加载系统文件

`loader.c` `load_sys_files`

```
static int load_sys_files(struct wrap_driver *driver, struct load_driver *load_driver)
```

该函数首先申请内存，将待加载的 sys pe 文件复制到申请的内存。

然后 `link_pe_images(driver->pe_images, driver->num_pe_images)`

`loader.c` `link_pe_images`

```
int link_pe_images(struct pe_image *pe_image, unsigned short n)
```

将依次执行

*   `check_nt_hdr` 执行一些检查
*   `fix_pe_image` 执行内存节对齐操作
*   `read_exports` 如果出函数，添加到全局函数表
*   `fixup_reloc` 处理重定位数据修改
*   `fixup_imports` 处理导入表
*   `fix_user_shared_data_addr` X64 的一些处理

至此 windows 的 ndis 二进制镜像正式加载到 linux 内核。

### 处理导入表

以 `fixup_imports` 为例，看一下 ndiswrapper 是如何处理 NDIS 接口的。

`pe_linker.c` `fixup_imports`

```
static int fixup_imports(void *image, IMAGE_NT_HEADERS *nt_hdr)
  opt_hdr = &nt_hdr->OptionalHeader;
  import_data_dir =
    &opt_hdr->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT];
  dirent = RVA2VA(image, import_data_dir->VirtualAddress,
      IMAGE_IMPORT_DESCRIPTOR *);
  for (i = 0; dirent[i].Name; i++) {
    name = RVA2VA(image, dirent[i].Name, char*);
    ret += import(image, &dirent[i], name);
  }

static int import(void *image, IMAGE_IMPORT_DESCRIPTOR *dirent, char *dll)
  lookup_tbl = RVA2VA(image, dirent->u.OriginalFirstThunk, ULONG_PTR *);
  address_tbl = RVA2VA(image, dirent->FirstThunk, ULONG_PTR *);
  for (i = 0; lookup_tbl[i]; i++) {
    symname = RVA2VA(image,
        ((lookup_tbl[i] &
      ~IMAGE_ORDINAL_FLAG) + 2), char *);
    get_export(symname, &adr);
    address_tbl[i] = (ULONG_PTR)adr;
  }
```

定位到 pe 的导入函数表，然后依次通过 `get_export` 函数查询函数的地址，然后修改地址表。

```
static int get_export(char *name, generic_func *func)
{
  int i, j;
  struct wrap_export *exports[] = {
    ntoskernel_exports,
    ntoskernel_io_exports,
    ndis_exports,
    crt_exports,
    hal_exports,
    rtl_exports,
#ifdef ENABLE_USB
    usb_exports,
#endif
  };
  for (j = 0; j < ARRAY_SIZE(exports); j++)
    for (i = 0; exports[j][i].name != NULL; i++)
      if (strcmp(exports[j][i].name, name) == 0) {
        *func = exports[j][i].func;
        return 0;
      }
  for (i = 0; i < num_pe_exports; i++)
    if (strcmp(pe_exports[i].name, name) == 0) {
      *func = pe_exports[i].addr;
      return 0;
    }
  return -1;
}
```

这个函数依次检查所有的导出函数表，以及 `pe_exports`【sys pe 镜像导出函数表】返回函数地址。

### 导出表实现

以 `ndis_exports` 表为例；

```
#include "ndis_exports.h"
```

该文件通过 `mkexport.sh` 脚本自动生成 `./mkexport.sh ndis.c ndis_exports.h`

内容如下：

```
extern struct wrap_export ndis_export[];
struct wrap_export ndis_export[] = {
  WIN_SYMBOL(NdisAcquireReadWriteLock, 3),
  WIN_SYMBOL(NdisAcquireSpinLock, 1),
```

其中 `wrap_export` 数据结构以及 `WIN_SYMBOL` X86 宏定义如下：

```
struct wrap_export {
  const char *name;
  generic_func func;
};

#define WIN_SYMBOL(name, argc) {#name, (generic_func)name}
```

`WIN_SYMBOL(NdisAcquireReadWriteLock, 3),` 即对应`{"NdisAcquireReadWriteLock", NdisAcquireReadWriteLock}`

至此整个 ndiswrapper 的框架大致出来了，最关键的部分 NDIS API 实现见 `ndis.c`，实现了 NDIS 导出的 133 个函数。

在此向作者表示敬意，全世界的 linux 用户感谢你们的辛勤劳动！

2011-12-27
