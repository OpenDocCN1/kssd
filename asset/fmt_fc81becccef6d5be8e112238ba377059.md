
```c
int b = 0 ,  nx = 0, ny = 0;  

DWORD s = 0;  
::ReadProcessMemory(Process, (LPCVOID)x, &nx, 1, NULL);  //获取横向方格长度  
::ReadProcessMemory(Process, (LPCVOID)y, &ny, 1, NULL);  //获取纵向方格长度  

DWORD xuanzong = 0x10037E1;        //选方格的函数地址  
DWORD x1 = 1;  
DWORD y1 = 1;  

int (*Rec)[900] = new int[nx][900];  

for(int i = 0; i < nx * 32; i += 32)  
{  
  for(int j = 0; j < ny; j++)  
  {  
    ::ReadProcessMemory(Process, (LPCVOID)(addr + i+j), &b, 1, NULL);  
    if (b == 0x8E || b == 0x8F)     //把内存布局写入自定义的数组  
     {  
      Rec[i/32][j] = 1;     
     }  
     else  
       Rec[i/32][j] = 0;  
  }  
}  

for(i = 0; i < nx; i ++)  
{     
  for(int j = 0; j < ny; j++)  
  {  
    x1 = i + 1;  
    y1 = j + 1;  
    if(Rec[i][j] != TRUE)           //选择没有雷的方格  
    {  
      _asm  
      {  
        push x1        //要选方格的 X  
        push y1        //要选方格的 Y  
        call xuanzong  //调用选方格函数  
      }  
    }  
  }  
}  
```

哈哈,这样一个扫雷的秒杀外挂就出来了！由于我水平有限,如有错误之处请跟贴指正！(附件暂时还不能用 5555555 有时间找个地址我会贴出来

---
**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

**引用**: 最初由 `CARY`发布
**【文章标题】**: 打造扫雷终极外挂
**【文章作者】**: `CARY`
**【作者邮箱】**: `caryzyu@hotmail.com`
**【软件名称】**: 扫雷 5.1.2600.0
**【下载地址】**: XP 自带
**【编写语言】**: `VC++ 6.0`

我把正确的代码贴出来，如下：

```c
void __stdcall saolei()  
{  
   DWORD addr = 0x1005361;  
   DWORD x    = 0x10056A8;  
   DWORD y    = 0x10056AC;  

  HWND hwnd = ::FindWindow(NULL, "扫雷");  
  DWORD hProcessId;  
  ::GetWindowThreadProcessId(hwnd, &hProcessId);  
  HANDLE Process = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, hProcessId);  

  int b = 0 ,  nx = 0, ny = 0;  

  DWORD s = 0;  
  ::ReadProcessMemory(Process, (LPCVOID)x, &nx, 1, NULL);  //获取横向方格长度  
  ::ReadProcessMemory(Process, (LPCVOID)y, &ny, 1, NULL);  //获取纵向方格长度  

  DWORD xuanzong = /*0x10037E1*/0x01003512;        //选方格的函数地址  
  DWORD x1 = 1;  
  DWORD y1 = 1;  

  int (*Rec)[900] = new int[nx][900];  

  for(int i = 0; i < nx * 32; i += 32)  
  {  
  for(int j = 0; j < ny; j++)  
  {  
    ::ReadProcessMemory(Process, (LPCVOID)(addr + i+j), &b, 1, NULL);  
    if (b == 0x8E || b == 0x8F)     //把内存布局写入自定义的数组  
    {  
      Rec[i/32][j] = 1;     
    }  
    else  
       Rec[i/32][j] = 0;  
  }  
  }  

  for(i = 0; i < nx; i ++)  
  {     
  for(int j = 0; j < ny; j++)  
  {  
      x1 = i + 1;  
      y1 = j + 1;  
      if(Rec[i][j] != 1)           //选择没有雷的方格  
      {  
        _asm  
        {  
           push x1        //要选方格的 X  
           push y1        //要选方格的 Y  
           call xuanzong  //调用选方格函数  
        }  
      }  
  }  
  }  
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
