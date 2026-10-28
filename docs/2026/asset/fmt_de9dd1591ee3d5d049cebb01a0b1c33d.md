
```
// atexit.c
#include "minicrt.h"

typedef struct __func_node{
    atexit_func_t func;
    void *arg;
    int is_cxa;
    struct __func_node *next;
} func_node;

static func_node *atexit_list = 0;

int register_atexit(atexit_func_t func, void *arg, int is_cxa){
    func_node *node;
    if (!func)
    {
        return -1;
    }
    node = (func_node*)malloc(sizeof(func_node));
    if (node == 0)
    {
        return -1;
    }
    node->func = func;
    node->arg = arg;
    node->is_cxa = is_cxa;
    node->next = atexit_list;
    atexit_list = node;
    return 0;
}

#ifndef WIN32
int __cxa_atexit(cxa_func_t func, void *arg, void *unused){
    return register_atexit((atexit_func_t)func, arg, 1);
}
#endif

int atexit(atexit_func_t func){
    return register_atexit(func, 0, 0);
}

void mini_crt_call_exit_routine(){
    func_node *p = atexit_list;
    for(; p != 0; p = p->next){
#ifdef WIN32
        p->func();
#else
        if (p->is_cxa)
        {
            ((cxa_func_t)p->func)(p->arg);
        } else {
            p->func();
        }
#endif
        free(p);
    }
    atexit_list = 0;
}
```

值得一提的是，在注册函数时，被注册的函数是插入到列表头部的，而最后`mini_crt_call_exit_routine()`是从头部开始遍历的，于是由`atexit()`或`__cxa_atexit()`注册的函数是按照先注册后调用的顺序，这符合析构函数的规则，因为先构造的全局对象应该后析构。

#### 5.3.4 入口函数修改

由于增加了全局构造和析构的支持,那么需要对Mini CRT的入口函数和`exit()`函数进行修改，把对`do_global_ctors()`和`mini_crt_call_exit_routine()`的调用加入到`entry()`和`exit()`函数中去。修改后的`entry.c`如下(省略一部分未修改的内容):

```
// entry.c
...

void mini_crt_entry(void){
    ...
    if (mini_crt_heap_init() == -1)
    {
        crt_fatal_error("heap initialize failed");
    }
    if (mini_crt_io_init() == -1)
    {
        crt_fatal_error("IO initialize failed");
    }
    do_global_ctors();
    ret = main(argc,argv);
    exit(ret);
}

void exit(int exitCode){
    mini_crt_call_exit_routine();
#ifdef WIN32
    ExitProcess(exitCode);
#else
    asm("movl %0,%%ebx \n"
        "movl $1,%%eax \n"
        "int $0x80     \n"
        "hlt           \n"
        :
        : "m" (exitCode)
        : "%ebx");
#endif
}
```

#### 5.3.5 `stream`与`string`

C++的Hello World里面一般都会用到`cout`和`string`，以展示C++的特性。流和字符串是C++ STL的最基本的两个部分，我们在这一节中为MiniCRT增加`string`和`stream`的实现，在有了流和字符串之后,Mini CRT将最终宣告完成,可以考虑将它重命名为Mini CRT++。

当然，在真正的STL实现中，`string`和`stream`的实现十分复杂，不仅有强大的模板定制功能、缓冲，庞大的继承体系及一系列辅助类。我们在实现时还是以展示和剖析为最基本的目的，简化一切能够简化的内容。`string`和`stream`的实现将遵循下列原则。

- 不支持模板定制，即这两个类仅支持`char`字符串类型，不支持自定义分配器等，没有`basic_string`模板类。

- 流对象仅实现`ofstream`，且没有继承体系，即没有`ios_base`、`stream`、`ostream`、`fstream`等类似的相关类。

- 流对象没有内置的缓冲功能，即没有`stream_buffer`类支持。

- `cout`作为`ofstream`的一个实例，它的输出文件是标准输出。

`stream`和`string`类的实现用到了不少C++语言的特性，已经一定程度上偏离了本文的意义，因此在此仅将它们的实现源代码列出，而不做更多的详细分析。有兴趣的读者可以参考C++STL的相关实现的资料，如果对C++语言本身不熟悉，也可以跳过这一节，这并不影响对Mini CRT 整体实现的理解。`string`和`iostream`的实现如下：

```
// string.cpp
#include "minicrt.h"

namespace std{

class string{
    unsigned len;
    char *pbuf;
public:
    explicit string(const char *str);
    string(const string&);
    ~string();
    string& operator=(const string&);
    string& operator=(const char *s);
    const char& operator[](unsigned idx) const;
    char& operator[](unsigned idx);
    const char *c_str() const;
    unsigned length() const;
    unsigned size() const;
};

string::string(const char *str) :
    len(0), pbuf(0){
    *this = str;
}
string::string(const string& s) :
    len(0), pbuf(0){
    *this = s;
}
string::~string(){
    if (pbuf != 0)
    {
        delete[] pbuf;
        pbuf = 0;
    }
}
string& string::operator=(const string& s){
    if (&s == this)
    {
        return *this;
    }
    this->~string();
    len = s.len;
    pbuf = strcpy(new char[len + 1], s.pbuf);
    return *this;
}
string& string::operator=(const char *s){
    this->~string();
    len = strlen(s);
    pbuf = strcpy(new char[len + 1], s);
    return *this;
}
const char& string::operator[](unsigned idx) const{
    return pbuf[idx];
}
char& string::operator[](unsigned idx){
    return pbuf[idx];
}
const char *string::c_str() const{
    return pbuf;
}
unsigned string::length() const{
    return len;
}
unsigned string::size() const{
    return len;
}
ofstream& operator<<(ofstream o, const string& s){
    return o << s.c_str();
}

}
```

```
// iostream
#include "minicrt.h"

namespace std{

class ofstream{
public:
    enum openmode{in = 1, out = 2, binary = 4, trunc = 8};
protected:
    const char *filename;
    ofstream::openmode md;
    FILE *fp;

public:
    ofstream(const ofstream& of);
    ofstream();
    explicit ofstream(const char *filename, ofstream::openmode md = ofstream::out);
    ~ofstream();
    ofstream& operator<<(char c);
    ofstream& operator<<(int n);
    ofstream& operator<<(const char *str);
    ofstream& operator<<(ofstream& (*)(ofstream&));
    void open(const char *filename, ofstream::openmode md = ofstream::out);
    void close();
    ofstream& write(const char *buf, unsigned size);
};

inline ofstream& endl(ofstream& o){
    return o << '\n';
}

class stdout_stream : public ofstream{
public:
    stdout_stream();
    stdout_stream(const stdout_stream& std);
};
extern stdout_stream cout;

}
```
