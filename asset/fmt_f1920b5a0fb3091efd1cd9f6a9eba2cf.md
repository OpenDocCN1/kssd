
`a1:=a1-$004800D1-5+1;`

`writeProcessMemory(pHandle,Pointer($004800D1),@ee9,sizeof(ee9),lpNumberOfBytesWritten);`

`writeProcessMemory(pHandle,Pointer($004800D1+1),@a1,sizeof(a1),lpNumberOfBytesWritten);`

```
asm
push eax
lea eax,get_sfz
mov a1,eax
pop eax
end;
```

`a1:=a1-$0042CCF8-5;`

`writeProcessMemory(pHandle,Pointer($0042CCF8),@ee9,sizeof(ee9),lpNumberOfBytesWritten);`

`writeProcessMemory(pHandle,Pointer($0042CCF8+1),@a1,sizeof(a1),lpNumberOfBytesWritten);`

```
end;
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

里面没什么技术含量，而且汇编很粗糙。。。。。。但是我想阐述的就是一个思路

这篇帖也就到此结束了。。。。。。。。。。。。。。。。。。。。。。。。。。。
