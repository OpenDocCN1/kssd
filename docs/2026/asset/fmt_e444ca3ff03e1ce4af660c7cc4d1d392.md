
写代码的时候只要计算一下`x`，然后把`x`填到`E8/E9`后面就可以了。

## 代码练习

### 练习1 内存偏移转文件偏移

```
int rva_to_foa(char* i,int n)
{
         Int  rva = 0;
         Int  rva_offset = 0;
         rva_offset = n;
         int sct_cnt = *(short*)((int)i + (*(int*)((int)i + 60))/*dos+*/ + 6/*节表个数*/);
         char* sord = (char*)((int)i + (*(int*)((int)i + 60))/*dos+*/ + 24);/*pe头*/
         sord += *(short*)( (int)i + (*(short*)((int)i + 60))/*可选头大小*/ + 20);
         sord += 12;//VirtualAddress;
         sord += 40 * (sct_cnt - 1);//最后一个节表的virtusladdress
         int s = sct_cnt;
         while (s > 0)
         {
                   if (rva_offset >= *(int*)sord)
                   {
                            break;
                   }
                   sord -= 40;
                   s--;
         }
         if (s == 0)//偏移在头部
         {
                   //printf("foa%x\n", rva_offset);
                   return rva_offset;
         }
         rva_offset -= *(int*)(sord);//减fva+foa
         rva_offset += *(int*)(sord + 8);
         //printf("foa%x\n", rva_offset);
         return rva_offset;
}
```

### 练习2 文件偏移转内存偏移

```
int foa_to_rva(char* i, int n)
{
         Int  rva = 0;
         Int  rva_offset = 0;
         rva_offset = n;
         int sct_cnt = *(short*)((int)i + (*(int*)((int)i + 60))/*dos+*/ + 6/*节表个数*/);
         char* sord = (char*)((int)i + (*(int*)((int)i + 60))/*dos+*/ + 24);/*pe头*/
         sord += *(short*)((int)i + (*(short*)((int)i + 60))/*可选头大小*/ + 20);
         sord += 20;//foa;
         sord += 40 * (sct_cnt - 1);//最后一个节表的virtusladdress
         int s = sct_cnt;
         while (s > 0)
         {
                   if (rva_offset >= *(int*)sord)
                   {
                            break;
                   }
                   sord -= 40;
                   s--;
         }
         if (s == 0)//偏移在头部
         {
                   //printf("foa%x\n", rva_offset);
                   return rva_offset;
         }
         rva_offset -= *(int*)(sord);//减foa+rva
         rva_offset += *(int*)(sord-8 );
         //printf("rva%x\n", rva_offset);
         return rva_offset;
}
```

### 练习3 在代码节空白区添加代码

```
//先调用文件打开函数-》文件buffer转映像buff-》添加代码函数（也就是这个）-》映像buff
//转文件buff函数存盘
char* inseart_code(char* imbuff)
{
         char* buff = imbuff;
         char* sod = (imbuff + (*(int*)(imbuff + 60) + 24));//pe头尾
         sod += *(short*)(imbuff + (*(int*)(imbuff + 60) + 20));
         while (strcmp(sod, ".text") != 0)
         {
                   sod += 40;
         }
         buff = (imbuff + (*(int*)(sod + 12))) + (*(int*)(sod + 8));//节的空白区
         char* oep = (imbuff + (*(int*)(imbuff + 60) + 40));
         //      oep += *(short*)(imbuff + (*(int*)(imbuff + 60) + 20));
         int roep = *(int*)oep;
         *(int*)oep = buff - imbuff;//代码入口
         char a[] = { 0x6A,0,0x6A,0,0x6A,0,0x6A,0,0xE8,0,0,0,0,0xE9,0,0,0,0 };
         int dest_addr = 0x75fc0047;//messageBox 的地址  这个地址会改变
         int x = dest_addr - (((int)(buff - imbuff) + 13) + 0x400000);
         int y = 0x400000 + roep - (((int)(buff - imbuff) + 18) + 0x400000);
         *(int*)(a + 9) = x;
         *(int*)(a + 14) = y;
         int s = 0;
         //75FC0026    8BEC            mov ebp, esp
         while (s <= (sizeof(a) / sizeof(a[0])))
         {
                   (*(char*)buff++) = a[s];
                   s++;
         }
         return imbuff;
}
```

# 0x4 新增节

![](https://bbs.kanxue.com/upload/attach/202003/813079_92G3TECKJ65PA47.jpg)

这个比较简单，多余文字我不写了。直接上代码。。

## 代码练习

### 练习1 新增节

```
char* inseart_jb(char* imbuff,int size)
{
         char* buff = imbuff;
         char* sh = (char*)((imbuff + (*(int*)(imbuff + 60))) + 24)/*pe*/;
         int sh_cnt = *(short*)(sh - 18);//节表个数
         int old_size = *(int*)(sh + 56);
         *(int*)(sh + 56) += size;//sizeofimg扩大
         flen += size;
         unsigned int nl = *(int*)(sh + 56);
         (*(short*)(sh - 18))++;//节表数+1
         sh += *(short*)(sh - 4);//节表
         int s = 0;
         while (s++ < sh_cnt)
         {
                   sh += 40;
         }
         char code[] = {
                   0x2E, 0x74, 0x65, 0x78, 0x74, 0x00, 0x00, 0x00, 0x40, 0xE9, 0x01,
                   0x00, 0x00, 0x10, 0x00, 0x00, 0x00, 0xF0, 0x01, 0x00, 0x00, 0x10,
                   0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
                   0x00, 0x00, 0x00, 0xe0, 0x00, 0x00, 0xf0
         };//节表
         strcpy(code, "tttt");
         unsigned int va = 0;
         int vs_rs = size;//尺寸
         if (*(int*)(sh - 32) >= *(int*)(sh - 24))
         {
                   va = (*(int*)(sh - 32)) + (*(int*)(sh - 28));
         }
         else
         {
                   va = (*(int*)(sh - 24)) + (*(int*)(sh - 28));
         }
         unsigned int ra = (*(int*)(sh - 20)) + (*(int*)(sh - 24));
         *(int*)(code + 8) = vs_rs;
         *(int*)(code + 12) = va;//p偏移
         *(int*)(code + 16) = vs_rs;
         *(int*)(code + 20) = ra;
         s = 0;
         while (s<40)
         {
                   *sh = code[s];
                   sh++;
                   s++;
         }
         //****************************新增节
         char* buff1 = (char*)realloc(imbuff,nl);
         memset((char*)(buff1 + old_size), 0, size);
         imbuff=buff1 ;
         return buff1;
}
```

# 0x5 扩大最后节和合并节

由于有的时候节表剩余空间不够80个字节，所以就不能增加节（当然可以也可以把dos sub给覆盖掉）但可以用扩大最后节/合并节

扩大最后节：

1. 拉伸到内存
2. 分配一块新的空间：`SizeOfImage + Ex`
3. 将最后一个节的`SizeOfRawData`和`VirtualSize`改成`N`

   `SizeOfRawData = VirtualSize = N`

   `N` = (`SizeOfRawData`或者`VirtualSize` 内存对齐后的值) + `Ex`
4. 修改`SizeOfImage`大小

   `SizeOfImage` = `SizeOfImage` + `Ex`

![](https://bbs.kanxue.com/upload/attach/202003/813079_MHUB9GB7YWF5PGC.jpg)

合并节

![](https://bbs.kanxue.com/upload/attach/202003/813079_MUJ5ZH47W444XNQ.jpg)

（字有点丑。。。）  最后把节的属性改到最大就ok。

## 代码练习

### 练习1 扩大最后节

```
char* expand_lastjb(char* imgbuff)
{
         char* si = (imgbuff + (*(unsigned int*)(imgbuff + 60)) + 80);
         (*(int*)si) += 0x1000;//扩大映像尺寸
         unsigned sl = (*(unsigned int*)si);
         char* buff = (char*)realloc(imgbuff,sl);
         if (buff==0)
         {
                   printf("内存分配失败--expand——lastjb");
                   getchar();
                   return 0;
         }
         memset(buff, 0, sl);
         imgbuff = buff;
         char* sh = (imgbuff + (*(unsigned int*)(imgbuff + 60)));//pe
         int sh_cnt = *(short*)(sh + 6);
         sh += ((*(short*)(sh + 20)) +24);//节表头
         int s = 0;
         while (s++ < (sh_cnt-1))
         {
                   sh += 40;
         }
         *(int*)(sh + 36) |= 0x60000020; //加上代码节的权限
         *(int*)(sh + 8) += 0x1000;
         *(int*)(sh + 16) += 0x1000;
         return imgbuff;
}
```

### 练习2 合并节
