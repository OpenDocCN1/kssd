
# 反劲舞 3.1 HShield 的问题

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-531.htm`](https://www.pediy.com/kssd/pediy09/pediy09-531.htm)

> 郁闷了,以前的方法都不能用了!
>
> 就 `GetPixel` 办法来说
>
> 1. 修改自己程序不让 `EhSvc.dll` 等 `DLL` 挂钩自己的程序
>
> 可是现在 `HShield` 都在驱动级拦截服务 `SDT`,所以应用层对抗没意思了
>
> 2. 用汇编自己写 `GetPixel`
>
> ```pascal
> function xiaocall(x1,x2,x3:DWORD):DWORD;
> asm
> mov      eax, $10BF
> mov      edx, $7FFE0300
> call     dword ptr [edx]
> ret      $0C
> end;
>
> function GetPixelA(DC: HDC; X, Y: Integer): COLORREF;stdcall
> asm
> push     ebx
> mov      ebx, dword ptr [ebp+$8]
> mov      eax, ebx
> push     edi
> and      eax, $0FFFF
> or       edi, $FFFFFFFF
> cmp      eax, $10000
> jnb      @@2
> shl      eax, $4
> add      eax, dword ptr [$77F33000]
> cmp      byte ptr [eax+$A], 1
> jnz      @@2
> mov      ecx, ebx
> shr      ecx, $10
> cmp      word ptr [eax+$8], cx
> jnz      @@2
> mov      ecx, dword ptr [eax+$4]
> and      ecx, $FFFFFFFE
> cmp      ecx, dword ptr [$77F33004]
> jnz      @@2
> push     esi
> mov      esi, dword ptr [eax+$C]
> test     esi, esi
> je       @@1
> push     dword ptr [ebp+$10]
> push     dword ptr [ebp+$C]
> push     ebx
> call     xiaocall
> mov      edi, eax
> mov      eax, dword ptr [esi+$5C]
> @@1:
> pop      esi
> @@2:
> mov      eax, edi
> pop      edi
> pop      ebx
> end;
> ```
>
> 可是现在 `HShield` 都在驱动级拦截服务 `SDT`,所以应用层对抗没意思了
>
> 3. 用 `BITMAP` 实现
>
> ```pascal
> function GetPixelEx(DC: HDC; X, Y: Integer): COLORREF;stdcall
> label
> js;
> var
>  p:pRGBArray;
>  yansebmp:TBitmap;
> begin
> try
>  Result:=$FFFFFFFF;
>  yansebmp:=TBitmap.Create;
>  IF not Assigned(yansebmp) then Exit;
>  yansebmp.PixelFormat:=pf24bit;
>
>  yansebmp.Height:=1;
>  yansebmp.Width:=Screen.Width;
>
>  BitBlt(yansebmp.Canvas.Handle,             //目标 dc，也就是 Image1 的 dc
>                  0,                                     //复制到 Image1 的 x   =   0 处
>                  0,                                     //复制到 Image1 的 y   =   0 处
>                  1,                            //复制 Image1->Width 这么宽
>                  1,                          //复制 Image1->Height 这么宽
>                  DC,                              //源 dc
>                  x,                                     //从桌面的 x   =   0 处开始复制
>                  y,                                     //从桌面的 y   =   0 处开始复制
>                  SRCCOPY                     //指定为直接复制过去，不做处理
>                  );
>  p:=yansebmp.ScanLine[0];
>  Result:=RGB(p[0].rgbtRed,p[0].rgbtGreen,p[0].rgbtBlue);
>  yansebmp.free;
>  Form1.Panel1.Color:=Result;
>  exit;
> except
>
> end;
> Result:=$FFFFFFFF;
> yansebmp.free;
> end;
> ```
>
> 消耗 `CPU` 太厉害不说,现在只对 `GETDC(0)` 的场景有效 其他也被驱动层拦截了
>
> 4. `HOOK` 程序 `劲舞团` 本身的程序里实现 `GETPIXEL` 因为劲舞 `3.0` 版本以及以前版本,对自己进程是放行的,现在 `3.1` 版本也不能用此漏洞了!
>
> 5. `Rootkit` 方法 我把自己的进程链从物理内存里删掉 以前老版本可以用 现在新版本也不成了
>
> 6. 驱动对抗 驱动不怎么精通 以前写个修复 `SDT` 的 但是没用!现在想不出什么好的办法来实现 `GETPIXEL` 跪求点子
>
> `4dd466d0e1e0cc123afbe504da2462ca`
