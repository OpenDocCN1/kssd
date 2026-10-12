
# 一道 C++ 测试题

> 原文：[`www.pediy.com/kssd/pediy12/130218.html`](https://www.pediy.com/kssd/pediy12/130218.html)

一道 C++ 测试题

在这个函数前面加代码，使能工作，并输出：

`<Hello> <world> <foo> <bar> <yow> <baz>`

代码：

```
void test_tokenizer()
{
  std::string str = ";;Hello|world||-foo--bar;yow;baz|";
  
  boko::tokenizer tokens(str, "-;|");

  for (boko::tokenizer::iterator it = tokens.begin();
       it != tokens.end(); ++it)
    std::cout << "<" << *it << "> ";
}
```

WinMount 继续招聘！请把简历发到 support@winmount.com

我先贴出我的解答。大家的做法，我有空的时候慢慢看，或点评。

声明，这个解答是我从 boost 化简得来的。不是我自己想出来的。

其中用到了 `std::string::const_iterator`，我还不知道如果不

用它怎么完成。

代码：

```
#include <windows.h>
#include <iostream>
#include <string>

namespace boko {  
  
  class token_iterator
  {
    typedef std::string::const_iterator Itor;   
    LPCSTR f_;
    Itor begin_;
    Itor end_;

  public:
    token_iterator(LPCSTR f, Itor begin, Itor e)
    : f_(f),begin_(begin),end_(e){ }

    bool operator != (const token_iterator & other) 
    { 
      skip();
      return begin_ != other.begin_;
    }
    void operator ++() 
    { 
      skip();
      Itor p = begin_;
      while (p != end_ && !IsSeperator(*p))
      {
        p++;
      }
      begin_ = p; 
    }
    std::string operator *() 
    { 
      std::string s;

      skip();
      Itor p = begin_;
      while (p != end_ && !IsSeperator(*p))
      {
        s += *p++;
      }
      return s;
    }
  private:
    bool IsSeperator(char c)
    {
      LPCSTR p = f_;

      while (*p != 0)
      {
        if (*p++ == c)
          return true;
      }
      return false;
    }
    void skip()
    {
      Itor p = begin_;
      while (p != end_ && IsSeperator(*p))
        p++;
      begin_ = p;
    }
  };
  
  class tokenizer
  {
    typedef std::string::const_iterator Itor;     
    Itor first_;
    Itor last_;
    LPCSTR f_;
  public:
    typedef token_iterator iterator;
    
    tokenizer(const std::string& s, LPCSTR f)
      : first_(s.begin()), last_(s.end()), f_(f) { }
        
    iterator begin() const { return iterator(f_,first_,last_); }
    iterator end() const { return iterator(f_,last_,last_); }         
  };

} // namespace boko

void test_tokenizer()
{
  std::string str = ";;Hello|world||-foo--bar;yow;baz|";
  
  boko::tokenizer tokens(str, "-;|");

  for (boko::tokenizer::iterator it = tokens.begin();
       it != tokens.end(); ++it)
    std::cout << "<" << *it << "> ";

  // 要求输出 <Hello> <world> <foo> <bar> <yow> <baz>
}
```
