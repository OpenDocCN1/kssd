

# ring3 层对抗劲舞 3.1 封取色的解决办法\[公布\]

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-539.htm`](https://www.pediy.com/kssd/pediy09/pediy09-539.htm)

> 废话不多说了  
> 实现步骤  
> 1：注入----注入都不会的话看看我 blog 里介绍的一些 windows 技术  
> 2：重写 getpixel 代码  有人问只要注入了不就可以 getpixel 了吗？  呵呵那是以前的事了,反外挂系统漏洞越来越少，注入后直接 getpixel 仍然没用，只好自己重新写了，本来想用 TBitmap 但是 TBitmap 使用结构参数过多 严重消耗 CPU  现实现采取方法如下  
> function GetPixelA(DC: HDC; X, Y: Integer): COLORREF; stdcall;  
> var  
> DC2:HDC;  
> Bhandle:Dword;  
> begin  
> Result:=0;  
> DC2:=CreateCompatibleDC(DC);  
> Bhandle:=CreateCompatibleBitmap(DC,1,1);  
> if Boolean(Bhandle) and Boolean(DC2) and Boolean(DC) then  
> begin  
>   SelectObject(DC2,Bhandle);  
>   StretchBlt(DC2,  
>       0,  
>       0,  
>       1,  
>       1,  
>       DC,  
>       X,  
>       Y,  
>       1,  
>       1,  
>       SRCCOPY  
>       );  
>  Result:=GetPixel(DC2,0,0);  
> end;  
> if Bhandle<>0 then  
> DeleteObject(Bhandle);  
> Sleep(1);  
> if DC2<>0 then  
> DeleteDC(DC2);  
> end;  
>   
> 该函数消耗 CPU 资源很少  与 windows 核心的 getpixel 速度差的不是太远！！！！  
> 以上函数由外挂吧的最恨 VC\[nproject\]公布,在 59v.org 首发 转载请注明出处！ 

> 怎么和我写的这么象啊：  
> function GetPixelA(DC: HDC; X, Y: Integer): COLORREF; stdcall;  
> var  
> DC2:HDC;  
> Bhandle:Dword;  
> begin  
> Result:=0;  
> DC2:=CreateCompatibleDC(DC);  
> Bhandle:=CreateCompatibleBitmap(DC,1,1);  
> if Boolean(Bhandle) and Boolean(DC2) and Boolean(DC) then  
> begin  
>   SelectObject(DC2,Bhandle);  
>   StretchBlt(DC2, 0, 0, 1, 1, DC, X, Y, 1, 1, SRCCOPY);  
>  Result:=GetPixel(DC2,0,0);  
> end;  
> if Bhandle<>0 then  
> DeleteObject(Bhandle);  
> Sleep(1);  
> if DC2<>0 then  
> DeleteDC(DC2);  
> end; 

 4dd466d0e1e0cc123afbe504da2462ca