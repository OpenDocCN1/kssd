

# 请问如何能找到 WndProc 的地址？

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-251.htm`](https://www.pediy.com/kssd/pediy09/pediy09-251.htm)

> int WINAPI WinMain(...){  
>   ......  
>   while ( GetMessage(&msg, NULL, 0, 0) ){  
>     TranslateMessage(&msg) ;  
>     DispatchMessage(&msg) ;  
>   }  
>   return msg.wParam ;  
> }  
>   
> LRESULT CALLBACK WndProc(...) {  
>   switch ( message ){  
>     ...  
>   }  
>   return DefWindowProc(...) ;  
> }  
> 一般程序都是这个结构，DispatchMessage 负责把 message 提交给 WndProc 处理，但我找不到 WndProc 的地址，只能找到 DispatchMessage，按 F8 之后这个消息就已经处理完了，怎么能跟进去找到 WndProc 的起始地址呢？ 

> GetWindowLong 

> 1. 已知窗口句柄求窗口过程地址，用 GetWindowLong/GetClassLong 函数  
> lpfnWndProc = (WNDPROC)GetWindowLong(hWnd, GWL\_WNDPROC)  
> lpfnWndProc = (WNDPROC)GetClassLong(hWnd, GCL\_WNDPROC)  
>   
> 2. 下断 RegisterClassA(W)/RegisterClassExA(W)查看其参数所指向的 WNDCLASS(EX)结构体的 lpfnWndProc 成员  
>   
> 总之方法很多 

> 对 DefWindowProc 下断。 

 4dd466d0e1e0cc123afbe504da2462ca