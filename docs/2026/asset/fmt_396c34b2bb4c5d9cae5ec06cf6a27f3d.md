
```
if(\*((uint32_t*)base)==0x0A786564)

{

int pid=getpid();

const char* proc_name=get_process_name(pid);

if(strstr(proc_name,g_szTargetName))

{

LOGI("openMemory found target dex");

char szDumpPath[256]={0};

sprintf(szDumpPath,"/data/data/%s/dump_marsha_%x.dex",g_szTargetName,size);

FILE* fd_dump=fopen(szDumpPath,"wb+");

if(fd_dump==NULL)

{

LOGI("fopen dumpfile error:%s",strerror(errno));

return;

}

fwrite(base,size,1,fd_dump);

fflush(fd_dump);

fclose(fd_dump);

LOGI("dex file save at %s size:%x",szDumpPath,size);

}

return old_openMemory(DexFile_thiz,base,size,location,location_checksum,mem_map,oat_dex_file,error_meessage);

}

else

{

return old_openMemory(DexFile_thiz,base,size,location,location_checksum,mem_map,oat_dex_file,error_meessage);

}

}
```

最后测试发现`libshella` 2.7-2.10版本的Dex都可以成功Dump出来

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [样本.zip](javascript:void(0);)

  （1.95MB，833次下载）
