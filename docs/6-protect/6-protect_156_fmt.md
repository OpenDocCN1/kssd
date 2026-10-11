
# 如何 diy 加密狗 hasp4

> 原文：[`www.pediy.com/kssd/pediy12/134467.html`](https://www.pediy.com/kssd/pediy12/134467.html)

## 1 芯片说明

我是用 `moto ib8` 20 引脚的 `flash` 芯片。

## 2 主要代码

```c
int  Chiper(unsigned short ax1, int bufferr, int *  k1, int* k2)         
{                                                        
 // int bufferr; // esi@1                                  
  char key; // dl@3                                      
  char i; // edi@3                                  
  char tempkey; // dl@4                                  
unsigned short length1; // [sp+4h] [bp-4h]@2                                  
    //*(WORD *) get value                                          
    length1 = ax1;                                      
    do                                                          
    {                                                           
      key = 0;                                                   
      i = 4;                                                    
      do                                                       
      {                                                          
        ax1= *(WORD *)k1;   
        tempkey = 2 * key;                                      
        if ( *(WORD *)k1 & 1 )                                  
        {                                                        
          *(WORD *)k1 = ((ax1 ^ *(WORD *)k2) >> 1) | 0x8000;       
          tempkey |= 1u;                                         
        }                                                       
        else                                                     
        {                                                        
          ax1 = ax1 >> 1;   
       *(WORD *)k1 = ax1;                                  
        }                                                       
        key = 2 * tempkey;                                      
        if ( *(BYTE *)k1 & 0x80 )                                  
          key |= 1u;                                           
        --i;                                                    
      }                                                       
      while ( i );                                              
      *(BYTE *)bufferr++ ^= key;                                  
    }                                                           
    while ( length1-- != 1 );                                  
  return ax1;                                                   
}                    
```

这段代码是单片机里面运行。

我把 pc 端反汇编。

问题 在 单片机里面！

## 3 其他发现

另外我发现在单片里面 稍稍做点手脚， `dump` 工具 就没作用。

## 4 硬件参考

可以参考的硬件设计为 `http://hc08web.de/usb08`。

## 5 软件参考

参考没混淆前的 `multikey.sys`。
