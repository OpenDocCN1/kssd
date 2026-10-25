
# Dbgout

> 原文：[`www.pediy.com/kssd/pediy11/115533.html`](https://www.pediy.com/kssd/pediy11/115533.html)

直接上源码：`dbgout.h`

代码：

```
/**====================================================================
 *   DBGOUT (ver 1.3)   by triones   2010-03-26.
 *
 *   indentation：1 Tab == 2 Blank
 *
 *   ！！注意！！在主文件包含前，请声明  #define  _DBGOUT，以定义全局变量 errs
 *   其它只需要包含头文件即可
 *
 *   DEMO:
 *   errs<<"debugview 中输出一个 int 值"<<12345678<<endd;
 *   errs<<"debugview 中输出一个十六进制值"<<hex<<0x12345678<<endd;
 *   errs<<errTchar(TEXT("输出 Unicode 字符"))<<endd;
 *
 *   同样，你可以把 errTchar 应用于 cout（因为只有 wcout 能输出 Unicode）
 *   cout<<errTchar(TEXT("输出 Unicode 字符"))<<endl;
 *
 *   但你不能把 endd 应用在 cout,cerr 等中，否则将出错
 *
 *   当然使用全局 errs，在多线程下产生冲突。没有关系，你可以在需要时定义一个局部 ostringstream 用于输出。
 *
 *   2010-05-11  去除 errTchar 的容量限制，动态申请空间(ver 1.2)
 *   2010-06-08  发现 errTchar 对 Unicode 的自行计算会让编译器做出奇怪结果（EB FE 死循环），故调用标准库。(ver 1.3)
 *====================================================================*/
#pragma once

#include <windows.h>
#include <iostream>
#include <sstream>
#include <iomanip>
using namespace std;

#ifdef _DBGOUT
  ostringstream  errs;
#else
  extern ostringstream errs;
#endif
/**====================================================================
 *   用于确定在 DebugView 输出信息
 *====================================================================*/
template<class _Elem,class _Traits> inline  basic_ostream<_Elem, _Traits>&
__CLRCALL_OR_CDECL endd(basic_ostream<_Elem, _Traits>& _Ostr)
  {
  ostringstream& oo = (ostringstream&)_Ostr;
  OutputDebugStringA(oo.str().c_str());    //注意到 OutputDebugStringW 将最终调用 OutputDebugStringA，反而效率低下
  oo.clear();
  oo.str("");
  return (_Ostr);
  }
/**====================================================================
 *   用于转换 Unicode 为 ASCII
 *====================================================================*/
class errTchar
  {
  public:
    explicit  errTchar(wchar_t* lpstr):lpstr_(lpstr){}
  private:
    wchar_t* lpstr_;
    template<class _Elem,class _Traits>
    friend    basic_ostream<_Elem, _Traits>& operator<<(basic_ostream<_Elem, _Traits>& _Ostr,const errTchar& m)
      {
      int len = wcslen(m.lpstr_);
      ++len;
      len*=sizeof(wchar_t);
      char* buf = new char[len];
      WideCharToMultiByte(CP_ACP,0,m.lpstr_,-1,buf,len,0,0);
      _Ostr<<buf;
      delete[] buf;
      return (_Ostr);
      }
  };
```

用了很久了，也没发现网上有类似的利用。放出来分享一下。

这招是从《标准 C++ 输入输出流及本地化》这书中学的，书是好书，可惜买不到了。只能看电子版。

用过流处理，知道好处的兄弟们应该不用我多说什么了吧。
