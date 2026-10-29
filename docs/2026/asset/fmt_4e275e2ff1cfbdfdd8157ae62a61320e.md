
# [原创]Unity3D mono模式游戏保护之浅谈

前面两篇文章都是以破解为主，因为研究Unity3D的保护已经有段时间了，目前已迭代了几版了，因为某些原因，暂时不能公开，这里就只能先将以前用过的一种简单的保护方式分享出来，希望对研究Unity3D mono模式游戏保护的童鞋有所帮助。这种保护方式需要对CLR PE文件格式有所了解，特别是其中的metadata数据项。这边分析的mono源码版本为5.6。

这里的保护方式通过修改mono的源码，自己编译出`libmono.so`的方式来保护。`mono_image_open_from_data_with_name`这个关键的函数就不用说了，跟随函数的执行流程，通过过代码流程跟随到`do_mono_image_load`函数：

```
static MonoImage *
do_mono_image_load (MonoImage *image, MonoImageOpenStatus *status,
gboolean care_about_cli, gboolean care_about_pecoff)
{
MonoCLIImageInfo *iinfo;
MonoDotNetHeader *header;
mono_profiler_module_event (image, MONO_PROFILE_START_LOAD);
/* if MONO_SECURITY_MODE_CORE_CLR is set then determine if this image is platform code */
image->core_clr_platform_code = mono_security_core_clr_determine_platform_image (image);
mono_image_init (image);
iinfo = image->image_info;
header = &iinfo->cli_header;
if (status)
*status = MONO_IMAGE_IMAGE_INVALID;
if (care_about_pecoff == FALSE)
goto done;
if (!mono_verifier_verify_pe_data (image, NULL))
goto invalid_image;
if (!mono_image_load_pe_data (image))
goto invalid_image;
if (care_about_cli == FALSE) {
goto done;
}
if (!mono_verifier_verify_cli_data (image, NULL))
goto invalid_image;
if (!mono_image_load_cli_data (image))
goto invalid_image;
if (!mono_verifier_verify_table_data (image, NULL))
goto invalid_image;
#ifndef USE_COREE
/* if the last bit is not set, then the image is mixed mode with native code */
if (!(iinfo->cli_cli_header.ch_flags & 1))
goto invalid_image;
#endif
mono_image_load_names (image);
load_modules (image);
done:
mono_profiler_module_loaded (image, MONO_PROFILE_OK);
if (status)
*status = MONO_IMAGE_OK;
return image;
invalid_image:
mono_profiler_module_loaded (image, MONO_PROFILE_FAILED);
mono_image_close (image);
return NULL;
}
```

其中对`Assembly-CSharp.dll`和`Assembly-CSharp-firstpass.dll`加载的关键函数为`mono_image_load_pe_data`和`mono_image_load_cli_data`。可通过自定义`do_mono_image_load`、`mono_image_load_pe_data`、`mono_image_load_cli_data`以及后面相关的关键函数来对自定义的文件进行加载。

`mono_image_load_pe_data`函数内容如下：

```
gboolean
mono_image_load_pe_data (MonoImage *image)
{
MonoCLIImageInfo *iinfo;
MonoDotNetHeader *header;
MonoMSDOSHeader msdos;
gint32 offset = 0;
iinfo = image->image_info;
header = &iinfo->cli_header;
#ifdef USE_COREE
if (!image->is_module_handle)
#endif
if (offset + sizeof (msdos) > image->raw_data_len)
goto invalid_image;
memcpy (&msdos, image->raw_data + offset, sizeof (msdos));
if (!(msdos.msdos_sig [0] == 'M' && msdos.msdos_sig [1] == 'Z'))
goto invalid_image;
msdos.pe_offset = GUINT32_FROM_LE (msdos.pe_offset);
offset = msdos.pe_offset;
offset = do_load_header (image, header, offset);
if (offset < 0)
goto invalid_image;
/*
* this tests for a x86 machine type, but itanium, amd64 and others could be used, too.
* we skip this test.
if (header->coff.coff_machine != 0x14c)
goto invalid_image;
*/
#if 0
if (header->pe.pe_major != 6 || header->pe.pe_minor != 0)
goto invalid_image;
#endif
/*
* FIXME: byte swap all addresses here for header.
*/
if (!load_section_tables (image, iinfo, offset))
goto invalid_image;
return TRUE;
invalid_image:
return FALSE;
}
```

这里主要对文件的dos头进行判断，在这里可自行定义dos头，再往下继续跟，这里加载pe头的关键函数`do_load_header`。在这个函数中可以自行定义PE中的数据。函数原型如下：

```
static int
do_load_header (MonoImage *image, MonoDotNetHeader *header, int offset)
{
MonoDotNetHeader64 header64;
#ifdef USE_COREE
if (!image->is_module_handle)
#endif
if (offset + sizeof (MonoDotNetHeader32) > image->raw_data_len)
return -1;
memcpy (header, image->raw_data + offset, sizeof (MonoDotNetHeader));
if (header->pesig [0] != 'P' || header->pesig [1] != 'E')
return -1;
/* endian swap the fields common between PE and PE+ */
SWAP32 (header->coff.coff_time);
SWAP32 (header->coff.coff_symptr);
SWAP32 (header->coff.coff_symcount);
SWAP16 (header->coff.coff_machine);
SWAP16 (header->coff.coff_sections);
SWAP16 (header->coff.coff_opt_header_size);
SWAP16 (header->coff.coff_attributes);
/* MonoPEHeader */
SWAP32 (header->pe.pe_code_size);
SWAP32 (header->pe.pe_uninit_data_size);
SWAP32 (header->pe.pe_rva_entry_point);
SWAP32 (header->pe.pe_rva_code_base);
SWAP32 (header->pe.pe_rva_data_base);
SWAP16 (header->pe.pe_magic);
/* now we are ready for the basic tests */
if (header->pe.pe_magic == 0x10B) {
offset += sizeof (MonoDotNetHeader);
SWAP32 (header->pe.pe_data_size);
if (header->coff.coff_opt_header_size != (sizeof (MonoDotNetHeader) - sizeof (MonoCOFFHeader) - 4))
return -1;
SWAP32 (header->nt.pe_image_base); /* must be 0x400000 */
SWAP32 (header->nt.pe_stack_reserve);
SWAP32 (header->nt.pe_stack_commit);
SWAP32 (header->nt.pe_heap_reserve);
SWAP32 (header->nt.pe_heap_commit);
} else if (header->pe.pe_magic == 0x20B) {
/* PE32+ file format */
if (header->coff.coff_opt_header_size != (sizeof (MonoDotNetHeader64) - sizeof (MonoCOFFHeader) - 4))
return -1;
memcpy (&header64, image->raw_data + offset, sizeof (MonoDotNetHeader64));
offset += sizeof (MonoDotNetHeader64);
/* copy the fields already swapped. the last field, pe_data_size, is missing */
memcpy (&header64, header, sizeof (MonoDotNetHeader) - 4);
/* FIXME: we lose bits here, but we don't use this stuff internally, so we don't care much.
* will be fixed when we change MonoDotNetHeader to not match the 32 bit variant
*/
SWAP64 (header64.nt.pe_image_base);
header->nt.pe_image_base = header64.nt.pe_image_base;
SWAP64 (header64.nt.pe_stack_reserve);
header->nt.pe_stack_reserve = header64.nt.pe_stack_reserve;
SWAP64 (header64.nt.pe_stack_commit);
header->nt.pe_stack_commit = header64.nt.pe_stack_commit;
SWAP64 (header64.nt.pe_heap_reserve);
header->nt.pe_heap_reserve = header64.nt.pe_heap_reserve;
SWAP64 (header64.nt.pe_heap_commit);
header->nt.pe_heap_commit = header64.nt.pe_heap_commit;
header->nt.pe_section_align = header64.nt.pe_section_align;
header->nt.pe_file_alignment = header64.nt.pe_file_alignment;
header->nt.pe_os_major = header64.nt.pe_os_major;
header->nt.pe_os_minor = header64.nt.pe_os_minor;
header->nt.pe_user_major = header64.nt.pe_user_major;
header->nt.pe_user_minor = header64.nt.pe_user_minor;
header->nt.pe_subsys_major = header64.nt.pe_subsys_major;
header->nt.pe_subsys_minor = header64.nt.pe_subsys_minor;
header->nt.pe_reserved_1 = header64.nt.pe_reserved_1;
header->nt.pe_image_size = header64.nt.pe_image_size;
header->nt.pe_header_size = header64.nt.pe_header_size;
header->nt.pe_checksum = header64.nt.pe_checksum;
header->nt.pe_subsys_required = header64.nt.pe_subsys_required;
header->nt.pe_dll_flags = header64.nt.pe_dll_flags;
header->nt.pe_loader_flags = header64.nt.pe_loader_flags;
header->nt.pe_data_dir_count = header64.nt.pe_data_dir_count;
/* copy the datadir */
memcpy (&header->datadir, &header64.datadir, sizeof (MonoPEDatadir));
} else {
return -1;
}
/* MonoPEHeaderNT: not used yet */
SWAP32 (header->nt.pe_section_align); /* must be 8192 */
SWAP32 (header->nt.pe_file_alignment); /* must be 512 or 4096 */
SWAP16 (header->nt.pe_os_major); /* must be 4 */
SWAP16 (header->nt.pe_os_minor); /* must be 0 */
SWAP16 (header->nt.pe_user_major);
SWAP16 (header->nt.pe_user_minor);
SWAP16 (header->nt.pe_subsys_major);
SWAP16 (header->nt.pe_subsys_minor);
SWAP32 (header->nt.pe_reserved_1);
SWAP32 (header->nt.pe_image_size);
SWAP32 (header->nt.pe_header_size);
SWAP32 (header->nt.pe_checksum);
SWAP16 (header->nt.pe_subsys_required);
SWAP16 (header->nt.pe_dll_flags);
SWAP32 (header->nt.pe_loader_flags);
SWAP32 (header->nt.pe_data_dir_count);
/* MonoDotNetHeader: mostly unused */
```
