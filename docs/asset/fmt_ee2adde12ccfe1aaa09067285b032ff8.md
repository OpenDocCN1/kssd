
## 汇编代码分析

| | |
|---|---|
| `0040B88D` | `037DA4` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`add` &nbsp;&nbsp;&nbsp;`edi`, `[ebp-$5C]` &nbsp;&nbsp;; 把上面几处的结果累加 |
| `0040B890` | `337DA0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`xor` &nbsp;&nbsp;&nbsp;`edi`, `[ebp-$60]` &nbsp;&nbsp;; 再与用户名的长度 `XOR` |
| `0040B893` | `897D9C` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`mov` &nbsp;&nbsp;&nbsp;`[ebp-$64]`, `edi` |
| `0040B896` | `8B459C` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`mov` &nbsp;&nbsp;&nbsp;`eax`, `[ebp-$64]` |
| `0040B899` | `99` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`cdq` |
| `0040B89A` | `33C2` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`xor` &nbsp;&nbsp;&nbsp;`eax`, `edx` |
| `0040B89C` | `2BC2` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`sub` &nbsp;&nbsp;&nbsp;`eax`, `edx` |
| `0040B89E` | `8BD0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`mov` &nbsp;&nbsp;&nbsp;`edx`, `eax` |
| `0040B8A0` | `8D45EC` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`lea` &nbsp;&nbsp;&nbsp;`eax`, `[ebp-$14]` |
| | |
| `0040B8A3` | `E8B8F70800` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`call` &nbsp;&nbsp;`0049B060` |
| `0040B8A8` | `8BD0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`mov` &nbsp;&nbsp;&nbsp;`edx`, `eax` |
| `0040B8AA` | `FF45D0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`inc` &nbsp;&nbsp;&nbsp;`dword ptr [ebp-$30]` |
| `0040B8AD` | `58` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`pop` &nbsp;&nbsp;&nbsp;`eax` |
| | |
| `0040B8AE` | `E835F90800` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`call` &nbsp;&nbsp;`0049B1E8` &nbsp;&nbsp;&nbsp;; 进行比较，不相等 `ZF=0` |
| `0040B8B3` | `85C0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`test` &nbsp;&nbsp;`eax`, `eax` |
| `0040B8B5` | `8D45EC` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`lea` &nbsp;&nbsp;&nbsp;`eax`, `[ebp-$14]` |
| `0040B8B8` | `0F94C1` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`setz` &nbsp;&nbsp;`cl` &nbsp;&nbsp;&nbsp;&nbsp;; 根据标志位设置 `CL` |
| `0040B8BB` | `83E101` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`and` &nbsp;&nbsp;&nbsp;`ecx`, `+$01` &nbsp;&nbsp;&nbsp;; `CL` 如果是 `0` 这里就成 `0` 了 |
| `0040B8BE` | `BA02000000` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`mov` &nbsp;&nbsp;&nbsp;`edx`, `$00000002` |
| `0040B8C3` | `51` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`push` &nbsp;&nbsp;`ecx` |
| `0040B8C4` | `FF4DD0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`dec` &nbsp;&nbsp;&nbsp;`dword ptr [ebp-$30]` |
| | |
| `0040B8C7` | `E81CF80800` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`call` &nbsp;&nbsp;`0049B0E8` |
| `0040B8CC` | `FF4DD0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`dec` &nbsp;&nbsp;&nbsp;`dword ptr [ebp-$30]` |
| `0040B8CF` | `8D45F0` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`lea` &nbsp;&nbsp;&nbsp;`eax`, `[ebp-$10]` |
| `0040B8D2` | `BA02000000` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`mov` &nbsp;&nbsp;&nbsp;`edx`, `$00000002` |
| | |
| `0040B8D7` | `E80CF80800` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`call` &nbsp;&nbsp;`0049B0E8` |
| `0040B8DC` | `59` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`pop` &nbsp;&nbsp;&nbsp;`ecx` |
| `0040B8DD` | `84C9` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`test` &nbsp;&nbsp;`cl`, `cl` |
| `0040B8DF` | `0F8458010000` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`jz` &nbsp;&nbsp;&nbsp;`0040BA3D` &nbsp;&nbsp;&nbsp;; `CL=0` 跳走就 `OVER` |

## 简易注册机：（VC++）

```cpp
#include <windows.h>  

#include <stdio.h>  

#include <iostream.h>  

void main()  

{  

  int eax,ebx,ecx,edx,esi,edi;  

  int ebp_54,ebp_58,ebp_5c,result;  

  char name[200]={0};  

  cout<<"Please input your name:";  

  cin>>name;  

  SYSTEM_INFO *pSI=new SYSTEM_INFO;  

  GetSystemInfo(pSI);  

  esi=pSI->wProcessorLevel;   


  ebx=pSI->dwProcessorType;  

  edi=pSI->dwNumberOfProcessors;  

  ecx=esi+ebx;  

  edx=pSI->wProcessorRevision;  

  ecx+=edi;  

  ebp_54=edx;  

  ecx+=ebp_54;  

  ebp_58=ecx;  

  eax=ebp_54;  

  eax+=esi;  

  eax^=ebx;  

  eax^=edi;  

  ebp_5c=eax;  

  eax=0;  

label1:  

  edx=esi+ebx+0x1234;  

  ecx=edi+esi+0x2345;  

  edx^=0x2345;  

  ecx^=0x3456;  

  ebx=edx;  

  edx=ebp_54;  

  edx+=ebp_58;  

  edi+=ebp_54;  

  edx+=0x4567;  

  esi=ecx;  

  edx^=0x5678;  

  edi+=0x3456;  

  ebp_54=edx;  

  edi^=0x4567;  

  ecx=ebp_58;  

  ecx+=ebp_5c;  

  ecx+=0x5678;  

  ecx^=0x6789;  

  ebp_58=ecx;  

  edx=ebp_5c;  

  edx+=ebx;  

  edx+=0x6789;  

  edx^=0x7891;  

  ebp_5c=edx;  

  eax++;  

  if (eax<0x25) goto label1;  

  result=(edi+esi+ebx+ebp_54+ebp_58+ebp_5c)^strlen(name);  

  cout<<"Your serial number is "<<result<<endl;  

  cout<<"KeyGen by RoBa  ThanQ!"<<endl;  

}  
```

呵呵，完全直译的，因为和硬件相关，无法确定它绝对正确，还请各位测试。

`4dd466d0e1e0cc123afbe504da2462ca`
