
```
0041DE57   |.  8D8C08 78A46AD7    lea ecx,dword ptr ds:[eax+ecx+D76AA478]
0041DE5E   |.  8BC1               mov eax,ecx
0041DE60   |.  C1E0 48            shl eax,48                                 ;就是这里变形的，在标准的MD5算法中这里应该是
0041DE63   |.  C1E9 D8            shr ecx,0D8                                ;shl eax,7
0041DE66   |.  0BC1               or eax,ecx                                 ;shr ecx,19
0041DE68   |.  03C7               add eax,edi
0041DE6A   |.  8BC8               mov ecx,eax
0041DE6C   |.  23D0               and edx,eax
0041DE6E   |.  F7D1               not ecx
0041DE70   |.  23CB               and ecx,ebx
0041DE72   |.  0BCA               or ecx,edx
0041DE74   |.  8B5424 18          mov edx,dword ptr ss:[esp+18]
0041DE78   |.  03CA               add ecx,edx
0041DE7A   |.  8D9429 56B7C7E8    lea edx,dword ptr ds:[ecx+ebp+E8C7B756]
0041DE81   |.  8BCA               mov ecx,edx
0041DE83   |.  03D2               add edx,edx                                ;这里的add edx,edx就相当于shl edx,1
0041DE85   |.  C1E9 1F            shr ecx,1F                                 ;所以这里只有shr ecx,1F而没有shl
0041DE88   |.  0BCA               or ecx,edx
0041DE8A   |.  03C8               add ecx,eax
0041DE8C   |.  8BD1               mov edx,ecx
0041DE8E   |.  8BE9               mov ebp,ecx
0041DE90   |.  F7D2               not edx
0041DE92   |.  23D7               and edx,edi
0041DE94   |.  23E8               and ebp,eax
0041DE96   |.  0BD5               or edx,ebp
0041DE98   |.  8B6C24 1C          mov ebp,dword ptr ss:[esp+1C]
0041DE9C   |.  03D5               add edx,ebp
0041DE9E   |.  8BE9               mov ebp,ecx
0041DEA0   |.  8D9C13 DB702024    lea ebx,dword ptr ds:[ebx+edx+242070DB]
0041DEA7   |.  8BD3               mov edx,ebx
0041DEA9   |.  C1E2 27            shl edx,27                                 ;原来的应该是shl edx,0C
0041DEAC   |.  C1EB F9            shr ebx,0F9                                ;shr ebx,14
0041DEAF   |.  0BD3               or edx,ebx
0041DEB1   |.  03D1               add edx,ecx
0041DEB3   |.  8BDA               mov ebx,edx
0041DEB5   |.  23EA               and ebp,edx
0041DEB7   |.  F7D3               not ebx
0041DEB9   |.  23D8               and ebx,eax
0041DEBB   |.  0BDD               or ebx,ebp
0041DEBD   |.  8B6C24 20          mov ebp,dword ptr ss:[esp+20]
0041DEC1   |.  03DD               add ebx,ebp
0041DEC3   |.  8D9C1F EECEBDC1    lea ebx,dword ptr ds:[edi+ebx+C1BDCEEE]
0041DECA   |.  8BFB               mov edi,ebx
0041DECC   |.  C1EF 18            shr edi,18                                 ;原来的应该是shl ebx,11
0041DECF   |.  C1E3 08            shl ebx,8                                  ;shr edi,0F
0041DED2   |.  0BFB               or edi,ebx
0041DED4   |.  8BDA               mov ebx,edx
0041DED6   |.  03FA               add edi,edx
0041DED8   |.  897C24 58          mov dword ptr ss:[esp+58],edi
0041DEDC   |.  8B6C24 58          mov ebp,dword ptr ss:[esp+58]
0041DEE0   |.  F7D7               not edi
0041DEE2   |.  23F9               and edi,ecx
0041DEE4   |.  23DD               and ebx,ebp
0041DEE6   |.  0BFB               or edi,ebx
0041DEE8   |.  8B6C24 24          mov ebp,dword ptr ss:[esp+24]
0041DEEC   |.  8B5C24 58          mov ebx,dword ptr ss:[esp+58]
0041DEF0   |.  03FD               add edi,ebp
0041DEF2   |.  8BEB               mov ebp,ebx
0041DEF4   |.  8D8438 AF0F7CF5    lea eax,dword ptr ds:[eax+edi+F57C0FAF]
0041DEFB   |.  8BF8               mov edi,eax
0041DEFD   |.  C1E7 48            shl edi,48
0041DF00   |.  C1E8 D8            shr eax,0D8
0041DF03   |.  0BF8               or edi,eax
0041DF05   |.  03FB               add edi,ebx
0041DF07   |.  8BC7               mov eax,edi
0041DF09   |.  23EF               and ebp,edi
```

--------------------------------------------------------------------------------------------

例如前5位为`WUMIN`,那么经过变形MD5计算得到`7A C1 34 14 70 41 E9 0B 10 13 EA FD 13 98 24 0F`这个Hash结果,然后经过`HashtoInt`这个转换为`2320253169439265`这样一个整数,现在我们用`BigCalc`中的`RSATool`设置好`Private(D)`,`Modulu(n)`和`Encrypt&Decrypt Text:`为`2320253169439265`,然后按下`Decrypt`就可以得到一组31位的字符串，这就是注册码后面的31位，到此这个软件的注册算法大致清楚了

感谢 无名 兄弟的帮助和支持 ![](https://www.kanxue.com/images/smilies/biggrin.gif)![](https://www.kanxue.com/images/smilies/biggrin.gif)
