

# Dump 易格式原体

> 原文：[`www.pediy.com/kssd/pediy06/pediy6857.htm`](https://www.pediy.com/kssd/pediy06/pediy6857.htm)

///////////////////////////////////////////////////////////////////////////////  
//  
//  FileName    :   DumpECode.oms（把易格式原体从 PE 文件中分离出来）  
//  Author      :   monkeycz  
//  Date        :   2004-11-28 21:36  
//  Comment     :   搜索易格式所在的节，然后 dump 出易格式原体。由于没有经过装载器  
//                  重定位，dump 出来的原体也不需要修复重定位信息。  
//  
///////////////////////////////////////////////////////////////////////////////  
  
  
mov reg04,0x400000                        //ImageBase，需要自行修改  
invoke Search, reg04, "2E65636F64"        //查找易格式所在节  
cmp reg00,-1  
je nofind                                 //没有找到:(  
mov reg01,reg00  
  
add reg01,0x0C                            //到 VirtualAddress 处  
invoke ReadMemLong,reg01,0x04             //读取易格式的 VirtualAddress  
mov reg05,reg00  
//invoke PrintNum, reg05, 16  
add reg05,reg04                           //当前易格式所在的节内存中的偏移  
//invoke PrintNum, reg05, 16  
  
add reg01,0x04                            //到 SizeOfRawData 处  
invoke ReadMemLong,reg01,0x04             //取出易格式的尺寸  
mov reg03,reg00  
//invoke PrintNum, reg03, 16  
  
invoke InputText,"Please input the filename:"  
cmp reg00,0x00  
je cancel  
invoke DumpMem,reg05,reg03,FreeBufferReg  //dump  
je error  
invoke msg,"Dump the E code finish!"  
halt  
  
nofind:  
invoke msg,"No find the E code!"  
halt  
  
cancel:  
halt  
  
error:  
invoke msg,"Write file error!"  
halt 上非  4dd466d0e1e0cc123afbe504da2462ca