
```
#define DEFINE_USERAPIHOOK1(lib, ret, x, t1, p1) \
    ret WINAPI User_##x(t1 p1);\
    CAPIHook ghook_##x(lib, #x, (PROC)User_##x);\
    ret WINAPI CallBack_##x(t1 p1){return ((ret (WINAPI *)(t1 p1))(PROC)ghook_##x)(p1);}\
    ret WINAPI User_##x(t1 p1)

#define DEFINE_USERAPIHOOK2(lib, ret, x, t1, p1, t2, p2) \
    ret WINAPI User_##x(t1 p1, t2 p2);\
    CAPIHook ghook_##x(lib, #x, (PROC)User_##x);\
    ret WINAPI CallBack_##x(t1 p1, t2 p2){return ((ret (WINAPI *)(t1 p1, t2 p2))(PROC)ghook_##x)(p1, p2);}\
    ret WINAPI User_##x(t1 p1, t2 p2)

#define DEFINE_USERAPIHOOK3(lib, ret, x, t1, p1, t2, p2, t3, p3) \
    ret WINAPI User_##x(t1 p1, t2 p2, t3 p3);\
    CAPIHook ghook_##x(lib, #x, (PROC)User_##x);\
    ret WINAPI CallBack_##x(t1 p1, t2 p2, t3 p3){return ((ret (WINAPI *)(t1 p1, t2 p2, t3 p3))(PROC)ghook_##x)(p1, p2, p3);}\
    ret WINAPI User_##x(t1 p1, t2 p2, t3 p3)

#define DEFINE_USERAPIHOOK4(lib, ret, x, t1, p1, t2, p2, t3, p3, t4, p4) \
    ret WINAPI User_##x(t1 p1, t2 p2, t3 p3, t4 p4);\
    CAPIHook ghook_##x(lib, #x, (PROC)User_##x);\
    ret WINAPI CallBack_##x(t1 p1, t2 p2, t3 p3, t4 p4){return ((ret (WINAPI *)(t1 p1, t2 p2, t3 p3, t4 p4))(PROC)ghook_##x)(p1, p2, p3, p4);}\
    ret WINAPI User_##x(t1 p1, t2 p2, t3 p3, t4 p4)
```

实际代码中，我们最多提供了9个参数的操作宏定义，并且还可以根据需要进行扩展。

这样在扩展`APIHook.DLL`中就可以使用如下的代码来挂接一个`API`，简单吧。如下：

```
DEFINE_USERAPIHOOK4("user32.dll", int, MessageBoxA, HWND, hWnd, LPCSTR, lpText, LPCSTR, lpCaption, UINT, uType)
{
    return CallBack_MessageBoxA(hWnd, lpText, lpCaption, uType);
}
DEFINE_USERAPIHOOK4("user32.dll", int, MessageBoxW, HWND, hWnd, LPCWSTR, lpText, LPCWSTR, lpCaption, UINT, uType)
{
    return CallBack_MessageBoxW(hWnd, lpText, lpCaption, uType);
}
```

## 2）实现部分

帖子很长了，可能不方便看了，因此实现部分，在回复中继续写。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
