

# 获得重定位表和调试表的函数，喜欢的就用了！

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-310.htm`](https://www.pediy.com/kssd/pediy09/pediy09-310.htm)

> 和前面一个类一样，这两个函数是对我前面那个类的更新！  
> 这次不和那个贴放一块了，我觉得重定位好像还是有点不足的地方，但是已经完全正常，就是不完美！![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")   
>   
> 喜欢的就拿去用吧！  
> 这两类函数可是网络上的稀罕货！  
> 简单先说一下重定位的结构：  
>   
> 地址:DWORD 类型  
> 大小:DWORD 类型  
> 内容:WORD 数组  
>   
> 主要来说内容;  
> 比如,内容的 WORD 值是 3 1 5 0   
>   
> 红色的就是它的类型,后面的"150"就是它要修正的量.  
>   
> 类型如下:  
> #define IMAGE\_REL\_BASED\_ABSOLUTE              0  
> #define IMAGE\_REL\_BASED\_HIGH                  1  
> #define IMAGE\_REL\_BASED\_LOW                   2  
> #define IMAGE\_REL\_BASED\_HIGHLOW               3  
> #define IMAGE\_REL\_BASED\_HIGHADJ               4  
> #define IMAGE\_REL\_BASED\_MIPS\_JMPADDR          5  
> #define IMAGE\_REL\_BASED\_SECTION               6  
> #define IMAGE\_REL\_BASED\_REL32                 7  
>   
> //下面的应该不常用的:  
> #define IMAGE\_REL\_BASED\_MIPS\_JMPADDR16        9  
> #define IMAGE\_REL\_BASED\_IA64\_IMM64            9  
> #define IMAGE\_REL\_BASED\_DIR64                 10  
> #define IMAGE\_REL\_BASED\_HIGH3ADJ              11  
>   
> //这次我只处理了 IMAGE\_REL\_BASED\_HIGHLOW 这个类型,也是最多最常见的.其它的我找了好久也没找到那个文件在用!也就没研究了,这也许是我这个函数的不完美的地方吧!  
>   
> 前面简要说了一下结构我们可以再简要说一下实现:  
> 获得类型的简单方法如下,假设 buf 是我们需要处理的值.  
> buf=(buf&0xF000)>>0x0C; //buf 就是类型号了.  
> buf=buf&0x0FFF //就是我们要的偏移量.  
>   
>   
> //------------------------------------------------------代码如下--------------------------------  
> PIMAGE\_BASE\_RELOCATION CPNExeInfo::GetRelocInfo(DWORD \*dwNum,DWORD \*dwAddr,DWORD \*dwSize,  
>                         DWORD \*dwNumItem,DWORD \*dwSectionNum,LPCTSTR \*szSectionName)  
> {  
>   PIMAGE\_BASE\_RELOCATION \_reloc=NULL;  
>   \_reloc=(PIMAGE\_BASE\_RELOCATION)\_ntHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress;  
>     
>   //\*dwSize=(DWORD)\_ntHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].Size;  
>   if(\_reloc==NULL){return NULL;}  
>     
>   \_reloc=(PIMAGE\_BASE\_RELOCATION)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\_reloc,NULL);  
>   
>   DWORD \*pdwBuf=(DWORD\*)\_reloc;  
>   
>   
>   for(int i=0;\_reloc->VirtualAddress!=NULL;i++)  
>   {  
>     if(dwAddr!=NULL){dwAddr\[i\]   =\_reloc->VirtualAddress;}  
>     if(dwSize!=NULL){dwSize\[i\]   =\_reloc->SizeOfBlock;}  
>     if(dwNumItem!=NULL){dwNumItem\[i\]=(\_reloc->SizeOfBlock-8)/2;}  
>     
>     PIMAGE\_SECTION\_HEADER \_section=GetSection();  
>     for(int n=0;n<\_ntHeader->FileHeader.NumberOfSections;n++)  
>     {  
>       if(\_section->VirtualAddress<=\_reloc->VirtualAddress&&  
>         (\_section->VirtualAddress+\_section->Misc.VirtualSize)>\_reloc->VirtualAddress)  
>       {  
>         if(dwSectionNum!=NULL){dwSectionNum\[i\]=n;}  
>         if(szSectionName!=NULL){szSectionName\[i\]=(PSTR)\_section->Name;}  
>         break;  
>       }  
>       \_section++;  
>     }  
>     \_reloc=(PIMAGE\_BASE\_RELOCATION)((DWORD)\_reloc+\_reloc->SizeOfBlock);  
>   }  
>   if(dwNum!=NULL){\*dwNum=i;}  
>   \_reloc=(PIMAGE\_BASE\_RELOCATION)pdwBuf;  
>   return \_reloc;  
> }  
>   
> DWORD  CPNExeInfo::GetRelocItem(PIMAGE\_BASE\_RELOCATION \_pbaseReloc,DWORD \*dwRva,DWORD \*dwOffset,  
>                  LPCTSTR \*szType,DWORD \*dwFarAddr)  
> {  
>   if(\_pbaseReloc==NULL){return NULL;}  
>   
>   DWORD \*pdwbuf=(PDWORD)\_pbaseReloc;  
>   pdwbuf=pdwbuf+2;  
>   WORD  \*pwdbuf=(PWORD)(pdwbuf);  
>   
>   int num=(\_pbaseReloc->SizeOfBlock-8)/2;  
>   for(int i=0;i<num;i++)  
>   {  
>     WORD wdbuf=\*pwdbuf;  
>     WORD wdtype=wdbuf&0xF000;  
>     if(dwRva!=NULL){dwRva\[i\]=\_pbaseReloc->VirtualAddress+(DWORD)(wdbuf&0x0FFF);}  
>     if(dwOffset!=NULL){  
>       dwOffset\[i\]=(DWORD)ImageRvaToVa(\_ntHeader,  
>         \_dosHeader,(DWORD)((DWORD)(wdbuf&0x0FFF)+\_pbaseReloc->VirtualAddress),NULL);  
>       dwOffset\[i\]=dwOffset\[i\]-(DWORD)\_dosHeader;  
>     }  
>     if(dwFarAddr!=NULL){  
>       DWORD dwbuf=(DWORD)(wdbuf&0x0FFF)+\_pbaseReloc->VirtualAddress;  
>       DWORD \*pdw=(PDWORD)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)dwbuf,NULL);  
>       dwFarAddr\[i\]=\*pdw;  
>     }  
>     if(szType!=NULL){  
>       wdtype=wdtype>>12;  
>       switch(wdtype)  
>       {  
>       case IMAGE\_REL\_BASED\_ABSOLUTE: {  
>         szType\[i\]="\[00\]ABOLUTE";  
>         if(dwRva!=NULL)    {dwRva\[i\]=NULL;}  
>         if(dwOffset!=NULL) {dwOffset\[i\]=NULL;}  
>         if(dwFarAddr!=NULL){dwFarAddr\[i\]=NULL;}  
>         break;  
>         }  
>       case IMAGE\_REL\_BASED\_HIGH:     {szType\[i\]="\[01\]HIGH";break;}  
>       case IMAGE\_REL\_BASED\_LOW:      {szType\[i\]="\[02\]LOW";break;}             
>       case IMAGE\_REL\_BASED\_HIGHLOW:  {szType\[i\]="\[03\]HIGHLOW";break;}            
>       case IMAGE\_REL\_BASED\_HIGHADJ:  {szType\[i\]="\[04\]HIGHADJ";break;}         
>       case IMAGE\_REL\_BASED\_MIPS\_JMPADDR:  {szType\[i\]="\[05\]MIPS\_JMPADDR";break;}         
>       case IMAGE\_REL\_BASED\_SECTION:       {szType\[i\]="\[06\]SECTION";break;}         
>       case IMAGE\_REL\_BASED\_REL32:         {szType\[i\]="\[07\]REL32";break;}      
>       case IMAGE\_REL\_BASED\_IA64\_IMM64:    {szType\[i\]="\[09\]IA64\_IMM64";break;}         
>       case IMAGE\_REL\_BASED\_DIR64:         {szType\[i\]="\[10\]DIR64";break;}         
>       case IMAGE\_REL\_BASED\_HIGH3ADJ:      {szType\[i\]="\[11\]HIGH3ADJ";break;}    
>       default:{szType\[i\]="\[-1\]UNKNOW";break;}  
>       }  
>     }  
>     pwdbuf++;  
>   }  
>   return num;  
> }  
>   
> PIMAGE\_DEBUG\_DIRECTORY CPNExeInfo::GetDebugInfo(DWORD \*dwSize,LPCTSTR \*szType)  
> {  
>   PIMAGE\_DEBUG\_DIRECTORY  \_debug=NULL;  
>   \_debug=(PIMAGE\_DEBUG\_DIRECTORY)\_ntHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].VirtualAddress;  
>   DWORD size=(DWORD)\_ntHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].Size;  
>   
>   if(\_debug==NULL){return NULL;}  
>   \_debug=(PIMAGE\_DEBUG\_DIRECTORY)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\_debug,NULL);  
>   
>   DWORD \*pdwBuf=(DWORD\*)\_debug;  
>   for(DWORD i=0;i<(size/sizeof(IMAGE\_DEBUG\_DIRECTORY));i++,\_debug++)  
>   {  
>     switch(\_debug->Type)  
>     {  
>     case IMAGE\_DEBUG\_TYPE\_UNKNOWN      :{szType\[i\]="\[00\]UNKNOW";break;}  
>     case IMAGE\_DEBUG\_TYPE\_COFF         :{szType\[i\]="\[01\]COFF";break;}  
>     case IMAGE\_DEBUG\_TYPE\_CODEVIEW     :{szType\[i\]="\[02\]CODEVIEW";break;}      
>     case IMAGE\_DEBUG\_TYPE\_FPO          :{szType\[i\]="\[03\]FPO";break;}  
>     case IMAGE\_DEBUG\_TYPE\_MISC         :{szType\[i\]="\[04\]MISC";break;}  
>     case IMAGE\_DEBUG\_TYPE\_EXCEPTION    :{szType\[i\]="\[05\]EXCEPTION";break;}  
>     case IMAGE\_DEBUG\_TYPE\_FIXUP        :{szType\[i\]="\[06\]FIXUP";break;}  
>     case IMAGE\_DEBUG\_TYPE\_OMAP\_TO\_SRC  :{szType\[i\]="\[07\]OMAPTOSRC";break;}  
>     case IMAGE\_DEBUG\_TYPE\_OMAP\_FROM\_SRC:{szType\[i\]="\[08\]OMAPFROMSRC";break;}  
>     case IMAGE\_DEBUG\_TYPE\_BORLAND      :{szType\[i\]="\[09\]BORLAND";break;}  
>     case IMAGE\_DEBUG\_TYPE\_RESERVED10   :{szType\[i\]="\[10\]RESERVED10";break;}  
>     default:{szType\[i\]="\[-1\]UNKNOW";break;}  
>     }  
>   }  
>   
>   \*dwSize=i;  
>   \_debug=(PIMAGE\_DEBUG\_DIRECTORY)pdwBuf;  
>   return \_debug;  
> }  
>   
>   
> //---------------------------------------------------------------------------------  
>   
>    这次的代码纯粹是个人杰作!高手们指点一下!呵呵~~~,自我感觉良好!![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif")   
>   
>   
>    现在还差什么表没发了！我想除了资源了吧！  
>   
>   
>                                                               -By EasyStudy For PhantomNet 

 4dd466d0e1e0cc123afbe504da2462ca