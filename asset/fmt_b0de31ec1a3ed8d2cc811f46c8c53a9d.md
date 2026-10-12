
# 102 : `VM_RtlLeaveCriticalSection`

3:

```
1  : MOV  VM_VAR1 ,  VM_EFL
2  : MOV dword  VM_Stack[0] ,VM_EBX
3  : MOV dword  VM_Stack[1] , 0x08
4  : ADD dword  VM_Stack[0] , [VM_Stack[1]]
5  : MOV dword VM_EBX , VM_Stack[0]
6  : MOV dword  VM_Stack[0] ,VM_ECX
7  : MOV dword  VM_Stack[1] , 0x0C
8  : ADD VM_Stack[0] , VM_Stack[0]
9  : MOV dword  VM_Stack[1] , [VM_Stack[0]]
10 : MOV dword  VM_Stack[0] ,VM_EDI
11 : SUB dword  VM_Stack[0] , [VM_Stack[1]]
12 : MOVZFSF VM_VAR2 , VM_VAR1
13 : TEST VM_Stack[9]<>0 , JMP 5
14 : MOV dword  VM_Stack[0] ,VM_EDX
15 : MOV dword  VM_Stack[1] , 0x02C
16 : SUB VM_Stack[0] , VM_Stack[0]
17 : MOV dword  VM_Stack[1] , 0x01
18 : MOV dword  [VM_Stack[0]] , VM_Stack[1]
19 : MOV dword  VM_Stack[0] ,VM_EBX
20 : MOV dword  VM_Stack[1] , [VM_Stack[0]]
21 : MOV dword VM_ECX , VM_Stack[1]
22 : MOV dword  VM_Stack[0] ,VM_EBX
23 : MOV dword  VM_Stack[1] , 0x04
24 : ADD VM_Stack[0] , VM_Stack[0]
25 : MOV dword VM_EBX , VM_Stack[0]
26 : MOV VM_EFL, VM_VAR1
```

出错的地方自己分析吧。

附带分析源代码 `DELPHI` 的。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `TX2-1.rar` |
