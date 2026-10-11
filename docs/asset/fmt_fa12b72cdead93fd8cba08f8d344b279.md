
```
004CF727     |.  >|add esp,8                       ;  [esi].[esi+4]出栈，恢复指针  

004CF72A     |.  >|mov dword ptr ss:[ebp-10],eax  

004CF72D     |.  >|mov dword ptr ss:[ebp-C],edx  

004CF730     |.  >|push dword ptr ss:[ebp+C]       ;  0  

004CF733     |.  >|push dword ptr ss:[ebp+8]       ;  2  

004CF736     |.  >|mov eax,dword ptr ss:[ebp-10]   ;  临时变量保存数组数据  

004CF739     |.  >|mov edx,dword ptr ss:[ebp-C]    ;  [esi+4]  

004CF73C     |.  >|call dumped.004057C0            ;  eax 是 Nm/2 的商,如果[esi+4]!=0,还要计算上[esi+4]  

004CF741     |.  >|mov ecx,dword ptr ss:[ebp-4]  

004CF744     |.  >|mov ecx,dword ptr ds:[ecx+4]    ;  子字符串数组基地址  

004CF747     |.  >|mov dword ptr ds:[ecx+ebx*8],eax  ;  更新数据,即以 Nm/2 的商覆盖 Nm  

004CF74A     |.  >|mov dword ptr ds:[ecx+ebx*8+4],edx  

004CF74E     |.  >|push dword ptr ss:[ebp+C]       ;  0  

004CF751     |.  >|push dword ptr ss:[ebp+8]       ;  2  

004CF754     |.  >|mov eax,dword ptr ss:[ebp-10]   ;  覆盖前的 Nm  

004CF757     |.  >|mov edx,dword ptr ss:[ebp-C]  

004CF75A     |.  >|call dumped.00405888  

004CF75F     |.  >|mov dword ptr ds:[esi],eax      ;  覆盖前的 Nm mod 2  

004CF761     |.  >|mov dword ptr ds:[esi+4],edx    ;  0  

004CF764     |.  >|dec ebx  

004CF765     |.  >|test ebx,ebx  

004CF767     |.^ >\jnz short dumped.004CF6F6  

004CF769     |.  >jmp short dumped.004CF76C  

004CF76B     |>  >/dec edi  

004CF76C     |>  >  mov eax,dword ptr ss:[ebp-4]  

004CF76F     |.  >|mov eax,dword ptr ds:[eax+4]    ;  数组首地址  

004CF772     |.  >|cmp dword ptr ds:[eax+edi*8+4],0  

004CF777     |.  >|jnz short dumped.004CF784  

004CF779     |.  >|cmp dword ptr ds:[eax+edi*8],0  

004CF77D     |.  >|jnz short dumped.004CF784  

004CF77F     |.  >|cmp edi,1  

004CF782     |.^ >\jg short dumped.004CF76B  

004CF784     |>  >mov eax,edi  

004CF786     |.  >cdq  

004CF787     |.  >mov ecx,dword ptr ss:[ebp-4]  

004CF78A     |.  >mov ecx,dword ptr ds:[ecx+4]  

004CF78D     |.  >cmp edx,dword ptr ds:[ecx+4]  

004CF790     |.  >jnz short dumped.004CF794  

004CF792     |.  >cmp eax,dword ptr ds:[ecx]  

004CF794     |>  >je short dumped.004CF7C1  

004CF796     |.  >lea eax,dword ptr ds:[edi+1]  

004CF799     |.  >push eax  

004CF79A     |.  >mov eax,dword ptr ss:[ebp-4]  

004CF79D     |.  >add eax,4  

004CF7A0     |.  >mov ecx,1  

004CF7A5     |.  >mov edx,dword ptr ds:[4CF018]    ;  dumped.004CF01C  

004CF7AB     |.  >call dumped.00405D74  

004CF7B0     |.  >add esp,4  

004CF7B3     |.  >mov eax,edi  

004CF7B5     |.  >cdq  

004CF7B6     |.  >mov ecx,dword ptr ss:[ebp-4]  

004CF7B9     |.  >mov ecx,dword ptr ds:[ecx+4]  

004CF7BC     |.  >mov dword ptr ds:[ecx],eax  

004CF7BE     |.  >mov dword ptr ds:[ecx+4],edx  

004CF7C1     |>  >pop edi  

004CF7C2     |.  >pop esi  

004CF7C3     |.  >pop ebx  

004CF7C4     |.  >mov esp,ebp  

004CF7C6     |.  >pop ebp  

004CF7C7     \.  >retn 8
```

上面可能说的不清楚,以前面的具体数来说,`123-456789987-654321000` 被转为 `0000007B-1B3A0FE3-27002568`。这里即是 `N3=0000007B`,`N2=1B3A0FE3`,`N1=27002568`

先是 `N3=N3/2=3D`,余数=`1`,继续 `N2=(N2+1*3B9ACA00)/2=2B6A6CF1`,余=`1`,`N3=(N3+1*3B9ACA00)/2=314D77B4`,余数=`0`。看看结果,`0x3D=61`,`0x2B6A6CF1=728394993`,`0x314D77B4=827160500`,连在一起就是 `61728394993827160500=123456789987654321000/2`。

前面这个`[esi+4]`变量的作用是,在 `Nm+3B9ACA00` 时候可能会>`0xFFFFFFFF`。产生进位,那么在 `Nm+余数*3B9ACA00/2` 的时候要把进位考虑在内,这是在 `004CF73C     |.  >|call dumped.004057C0` 里面处理的,这里就不说了。

所以这个 `CALL` 是对整个长十进制数/2，并把余数返回到外面，然后外面对余数倒序排列就 OK。

哎，其他什么都没跟出来，懊恼，看能不能来骗篇精华，找点安慰。

`4dd466d0e1e0cc123afbe504da2462ca`
