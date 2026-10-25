
01030894  |.  FF15 2CCE1601 |call dword ptr ds:[0x116CE2C]           ;  nvcuda.`cuFuncSetBlockShape`

0103089A  |.  85C0          |test eax,eax

0103089C  |.  75 59         |jnz short AccentOP.010308F7

03CCF930  03F34008

03CCF934  00000040

03CCF938  00000001

03CCF93C  00000001

### `cuFuncSetBlockShape`

**名称**

`cuFuncSetBlockShape` – 为函数设置块维度

**概要**

`CUresult cuFuncSetBlockShape(CUfunction func, int x, int y, int z);`

**说明**

指定在`func`给定的内核启动时所创建的线程块的X、Y和Z维度。

0103089E  |.  8B46 1C       |mov eax,dword ptr ds:[esi+0x1C]

010308A1  |.  33D2          |xor edx,edx

010308A3  |.  F776 2C       |div dword ptr ds:[esi+0x2C]

010308A6  |.  6A 01         |push 0x1

010308A8  |.  50            |push eax

010308A9  |.  8B46 0C       |mov eax,dword ptr ds:[esi+0xC]

010308AC  |.  50            |push eax

010308AD  |.  FF15 48CE1601 |call dword ptr ds:[0x116CE48]           ;  nvcuda.`cuLaunchGrid`

010308B3  |.  85C0          |test eax,eax

010308B5  |.  75 4D         |jnz short AccentOP.01030904

03CCF934  03F34008

03CCF938  00000090

03CCF93C  00000001

### `cuLaunchGrid`

**名称**

`cuLaunchGrid` – 启动CUDA函数

**概要**

`CUresult cuLaunchGrid(CUfunction func, int grid_width, int grid_height);`

`CUresult cuLaunchGridAsync(CUfunction func, int grid_width, int grid_height, CUstream stream);`

**说明**

在大小为`grid_width x grid_height`的块网格上调用内核。每个块包含此前通过调用`cuFuncSetBlockShape()`指定的多个线程。

`cuLaunchGridAsync()`是可选的，可通过传入非零`stream`参数关联到一个流。

010308B7  |.  FF15 FCCD1601 |call dword ptr ds:[0x116CDFC]           ;  nvcuda.`cuCtxSynchronize`

010308BD  |.  85C0          |test eax,eax

010308BF  |.  75 50         |jnz short AccentOP.01030911

### `cuCtxSynchronize`

**名称**

`cuCtxSynchronize` – 阻塞操作，直至上下文的任务完成

**概要**

`CUresult cuCtxSynchronize(void);`

**说明**

在设备完成之前请求的所有任务之前阻塞操作。如果之前的某项任务失败，`cuCtxSynchronize()`将返回一个错误。

## 目前要处理的问题：

1. 分析问题
   1. 分析出准备数据的结构，写出相应数据处理代码  
      暂时是由 16字节数据+尝试密码组成
   2. 有几个函数功能暂时不明，先尝试写代码进行测试
   3. 末端有几个循环来设置下次的核函数 参数

2. 去测试`cubin`模块

### GPU执行之前数据

```
04E90020  F55CD693  撝\
04E90024  56CB92AE  畳薞
04E90028  CB0D6548  He.
04E9002C  F5FF81DA  趤
04E90030  41008000  .€.A
04E90034  00000000  ....
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
04E90058  00000000  ....
04E9005C  00000090  ?. .
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
04E9009C  00000000  ....
```

### GPU执行后的结果

```
04E90020  F9714353  SCq
04E90024  A0888200  .倛
04E90028  B6C36237  7b枚
04E9002C  5A3655C7  荱6Z
04E90030  92C90A4B  K.蓲
04E90034  00000000  ....
04E90038  00000000  ....
04E9003C  00000000  ....
04E90040  F18DD142  B褝
04E90044  8CA7AE31  1
04E90048  3A1ABB7C  |?:
04E9004C  E0CEE303  阄
04E90050  87B3FCFC  硣
04E90054  00000000  ....
04E90058  00000000  ....
04E9005C  00000090  ?. .
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
04E9009C  00000000  ....
```

`memcpy(buf2,inputD+8+32*i,20);`

返回运算数据置换后作为`buf2`执行CPU下的运算，不分析了

```
AESInit(buf2, 0x80u, &aesContext);
AESEat(pEncryptInf->m368_key, buf4, &aesContext);
AESEat(pEncryptInf->m3A8_key, buf1, &aesContext);
AESEat(pEncryptInf->m3B8_key, &buf1[16], &aesContext);
o7MD5Init(&mdContext);
o7MD5Update(&mdContext, buf4, pEncryptInf->m320_nKeySize);
o7MD5GetFinalData(buf2, &mdContext);
return memcmp(buf1, buf2, 20) == 0;
```

### 说下前面准备数据构成

`int *inputD(int*)malloc(4*0x48000);` // 4* 48000=0x00120000

// ------------------内部循环准备数据-------------------------------------

`// 0x2400 * 0x80 =0x120000`

每个密码的所占数据区 `0x80` 总数据区`0x120000`

循环次数为`0x2400`

```
for(int inside=0;inside<0x120000;inside+=0x80)
```

01031006   > \8B5C24 1C     mov ebx,dword ptr ss:[esp+0x1C]                  ;  密码位数; Cases 2,4 of switch 01030FF0

0103100A   .  C1E0 07       shl eax,0x7                                      ;  0    1>>>>0x80

0103100D   .  0386 40030000 add eax,dword ptr ds:[esi+0x340]                 ;  +偏移

01031013   .  8B7C24 18     mov edi,dword ptr ss:[esp+0x18]                  ;  密码ASCII码

01031017   .  8BE8          mov ebp,eax                                      ;  数据开始位置

01031019   .  8B46 10       mov eax,dword ptr ds:[esi+0x10]                  ;  salt F55CD693  依次放入

2位密码

0103103E   .  8D55 10       lea edx,dword ptr ss:[ebp+0x10]                  ;  转置

01031041   >  8B1C8F        mov ebx,dword ptr ds:[edi+ecx*4]                 ;  00410041

01031044   .  0FCB          bswap ebx                                        ;  41004100

01031046   .  891A          mov dword ptr ds:[edx],ebx

01031048   .  41            inc ecx

01031049   .  83C2 04       add edx,0x4

0103104C   .  3BC8          cmp ecx,eax

0103104E   .^ 7C F1         jl short AccentOP.01031041

01031050   .  8B5C24 1C     mov ebx,dword ptr ss:[esp+0x1C]

01031054   >  F6C3 01       test bl,0x1

01031057   .  74 1D         je short AccentOP.01031076

中间有注意两个判断

一个位数判断

按4字节分配

一个奇偶判断

确定0x80分配

ELF文件的确认

设备上的变量确认

循环传递核函数参数确认

```
int HostPre2=0x000007D7;
int HostPre3=0x00000080;
CUdevice cuDevice=0;
CUdeviceptr device_input1 =0;
CUdeviceptr device_input2 =0;
CUdeviceptr device_result =0;
CUdeviceptr device_M1=0;//05500000
CUdeviceptr device_M2=0;
CUdeviceptr device_M3=0;
CUdeviceptr device_M4=0;
cuDeviceGet(&cuDevice,0);
CUcontext cuContext;
cuCtxCreate(&cuContext,0,cuDevice);
CUmodule cuModule;
//  cuModuleLoad(&cuModule,"od.cubin");
rc=cuModuleLoadData(&cuModule, codemem);                   //00B5F478  &cuModule---->  cuModule 03CCFC38
if(rc!=CUDA_SUCCESS)
{
  fprintf(stderr, "There is error on cuModuleLoadData(%ld)./n", rc);
  /* free all resource */
  cuCtxDestroy(cuContext);
  return 0;
}
CUfunction kernel_o;
cuModuleGetFunction(&kernel_o,cuModule,"kernel_o");        //00B5F47C &kernel_o---------->  kernel_o      04C8B1D8   ,   cuModule 03CCFC38
cuModuleGetGlobal(&device_input1,0,cuModule,"_we7742i");   //00B5F484 &device_input1----->  device_input1 05400000   ,0, cuModule 03CCFC38
cuModuleGetGlobal(&device_input2,0,cuModule,"_we7742x");   //00B5F488 &device_input2----->  device_input2 05400004   ,0, cuModule 03CCFC38
cuMemAlloc(&device_M1,0x00120000);                         //00B5F480 &device_M1--------->  device_M1     05500000
//4* 48000=0x00120000
cuMemcpyHtoD(device_M1,inputD,0x00120000);
cuMemcpyHtoD(device_input1,&HostPre2,4);
cuMemcpyHtoD(device_input2,&HostPre3,4);
for(int i=0x164 ;i<0xC350+0x164;i+=0x164)
{
  cuParamSeti(kernel_o,0,0x05500000);
  cuParamSeti(kernel_o,4,i-0x164);  //上次的 0       0x164
  if (i+0x46>0xC350)
    cuParamSeti(kernel_o,8,0xC350); //
  else
  {
    cuParamSeti(kernel_o,8,i); //当次的 0x164  0x2c8
  }
  cuParamSeti(kernel_o,0xC,0xC350);//总数
  cuParamSetSize(kernel_o,0x10);
  cuFuncSetBlockShape(kernel_o,0x40,1,1);
  cuLaunchGrid(kernel_o,0x90,1);
  cuCtxSynchronize();
}
cuMemcpyDtoH(inputD,device_M1,0x00120000);
//  free(codemem);
cuCtxDestroy(cuContext);
cuMemFree(device_result);
```

程序是使用CUDA驱动API方式处理，高级API方式分析也类似，可以参考`CUDA 参考手册.doc`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [CUDA 参考手册.doc](javascript:void(0);)

  （2.90MB，110次下载）
