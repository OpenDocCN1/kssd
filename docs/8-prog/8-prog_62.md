

# 用 C 语言实现 MFC 库函数 GetPrivteProfileString

> 原文：[`www.pediy.com/kssd/pediy12/133033.html`](https://www.pediy.com/kssd/pediy12/133033.html)

这几天一直在整反汇编，经常看见函数 GetPrivteProfileString，正好闲来无事，写点程序，目的主要在于熟悉 Win32 API 中文件操作和 C 语言对字符串的处理过程。  
MFC 中此函数 GetPrivteProfileString 可以用来读取配置文件.ini 的函数，它在 VS2008 的 msdn 中的定义是这样的：  
GetPrivateProfileString Function  
Retrieves a string from the specified section in an initialization file.  
翻译过来就是返回初始化文件中特定区域的字符串。参数如下：  
DWORD WINAPI GetPrivateProfileString(  
  \_\_in          LPCTSTR lpAppName,  
  \_\_in          LPCTSTR lpKeyName,  
  \_\_in          LPCTSTR lpDefault,  
  \_\_out         LPTSTR lpReturnedString,  
  \_\_in          DWORD nSize,  
  \_\_in          LPCTSTR lpFileName  
);  
第一个 LPCTSTR 类型的 lpAppName 是定值区间，第二个是要读的字符串的名，第三个默认值为 NULL，第四个为返回值，返回读到的字符串，第五个设置第四个参数(字符串)的大小，第六个为初始化文件路径。  
这里假设初始化文件如下:  
\[function\]  
func1 = init  
func2 = paint  
func3 = read  
func4 = write  
func5 = exit  
\[parameter\]  
para1 = void  
para2 = GDI  
para3 = file  
para4 = string  
para5 = handle  
我们要读取 func2 的值，那么读取方法就是  
CString strVal;  
DWORD size = 1000;  
GetPraviteProfileString(\_T("function"),\_T("func2"),\_T("Not Found"),strVal,GetBuffer(size),size,path);  
\_T("Not Found")为默认字符，即没找到时返回的值，path 为文件路径。  
然而当我们在移植程序到嵌入式系统时，这样的 MFC 函数是要不得的，我所使用的方法就是重写这个函数，用 C 语言写说到底就是字符串匹配的过程了，涉及到文件的就变成了 FILE 或 fstream 了，这里选用 FILE，看代码吧  
#define MAX\_STRLEN 65 //规定最长字符串长度，分配内存、拷贝用  
#define MAX\_NUM 100   //规定最多可存储的单元数  
typedef struct  
{  
  char func\[MAX\_STRLEN\];  
  char \*data;  
}FUNC;  
  
typedef struct  
{  
  char para\[MAX\_STRLEN\];  
  char \*data;  
}PARA;  
//分别用于存储 function 和 parameter  
FUNC \*func;  
PARA \*para;  
  
bool Mine\_Init(char\* cfg\_filename)  
{    
  func = (FUNC\*)malloc(sizeof(FUNC)\*MAX\_NUM);  
  para = (PARA\*)malloc(sizeof(PARA)\*MAX\_NUM);  
  FILE \*in\_fp;  
  char in\_buf\[MAX\_STRLEN\] = {0};  /\* 从文件中读出的每一行的缓冲区  \*/  
  FUNC \*ptrf = NULL;  
  PARA \*ptrp = NULL;  
  char \*str = NULL;  
  char substr\[\] = "\\n";  
  bool is\_func = false;  
  bool is\_para = false;  
  int index\_func = 0;  
  int index\_para = 0;  
  
  in\_fp = fopen (cfg\_filename, "r");  
  if (in\_fp == NULL)  
  {  
    printf("\\nError, %s not found!",cfg\_filename);  
    return false;  
  }  
  
  while (fgets(in\_buf,sizeof(in\_buf),in\_fp) != NULL)  
  {  
    str = strpbrk(in\_buf,substr);  
    \*str = '\\0';  
    if (!strcmp(in\_buf,"\[function\]"))  
    {  
      //存 function  
      ptrf = (FUNC\*)func;  
      is\_func = true;  
    }  
    else if(!strcmp(in\_buf,"\[parameter\]"))  
    {  
      //存 parameter  
      ptrf = NULL;//读完 function 的值，要把指针清掉，否则又会跳进 Write\_FUNC  
      ptrp = (PARA\*)para;  
      is\_para = true;  
    }  
    else if (ptrf != NULL)  
    {  
      if ( Write\_FUNC(ptrf, in\_buf, in\_fp, index\_func, is\_func) == -1)//写   
入 function 值  
      {  
        return false;  
      }  
    }  
    else if (ptrp != NULL)  
    {  
      if ( Write\_PARA(ptrp, in\_buf, in\_fp, index\_para, is\_para) == -1)//写   
入 parameter 值  
      {  
        return false;  
      }  
    }  
    else  
    {  
      fclose(in\_fp);  
      return false;  
    }  
  }  
  fclose(in\_fp);  
  return true;   
}  
下面是两个写入数据结构的函数，我的程序不能随机的读某个节点，只能一次性将节点全部读入数据结构   
里存储，有更好的方法还望指教!!  
int Write\_FUNC(FUNC\* ptr, char\* in\_buf, FILE \*in\_fp, int& index\_func, bool is\_func)  
{  
  assert(ptr && in\_fp);  
  memcpy(ptr, in\_buf, MAX\_STRLEN);  
  
  if (is\_func)  
  {  
    ZeroMemory(in\_buf, MAX\_STRLEN);  
    fgets(in\_buf, MAX\_STRLEN, in\_fp);  
    (func + index\_func)->data = (char\*)malloc(MAX\_STRLEN);//这里要初始化，否则会   
出错  
    memcpy((func + index\_func)->data,in\_buf,MAX\_STRLEN);  
    index\_func++;  
  }  
  return 0;   
}  
  
int Write\_PARA(PARA\* ptr, char\* in\_buf, FILE \*in\_fp, int& index\_para, bool is\_para)  
{  
  assert(ptr && in\_fp);  
  memcpy(ptr, in\_buf, MAX\_STRLEN);  
  
  if (is\_para)  
  {  
    ZeroMemory(in\_buf, MAX\_STRLEN);  
    fgets(in\_buf, MAX\_STRLEN, in\_fp);  
    (para + index\_para)->data = (char\*)malloc(MAX\_STRLEN);  
    memcpy((para + index\_para)->data,in\_buf,MAX\_STRLEN);  
    index\_para++;  
  }  
  return 0;   
}  
从函数中可以看到，in\_buf 是一行一行的读取到数据的，所以对应的配置文件格式也要有所变化  
\[function\]  
func1  
init  
func2  
paint  
func3  
read  
func4  
write  
func5  
exit  
\[parameter\]  
para1  
void  
para2  
GDI  
para3  
file  
para4  
string  
para5  
handle  
  
声明：是从本人 CSDN 上的同名文章转载过来的，纯属原创，转载请注明