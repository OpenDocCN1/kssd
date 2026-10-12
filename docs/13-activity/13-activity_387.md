
# 第一题 VM 人肉还原，附带还原工具源码

> 原文：[`www.pediy.com/kssd/pediy11/123973.html`](https://www.pediy.com/kssd/pediy11/123973.html)

![](img/962324edb48d57effae0c0571850a66b.jpg "酷")

共有三块 VM 代码：

## 1

```
1 : MOV VM\_VAR1 , VM\_EFL
2 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
3 : MOV dword  VM\_Stack\[1\] , 0x0FFFFFFFC
4 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
5 : MOV dword VM\_EBX , VM\_Stack\[0\]
6 : MOV dword  VM\_Stack\[1\] ,VM\_ECX
7 : MOV dword  \[VM\_Stack\[0\]\] , VM\_Stack\[1\]
8 : MOV dword  VM\_Stack\[0\] ,VM\_EBP
9 : MOV dword  VM\_Stack\[1\] ,VM\_EBP
10 : XOR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
11 : MOV VM\_EFL, VM\_VAR1
12 : MOV dword VM\_EBP , VM\_Stack\[0\]
13 : MOV dword  VM\_Stack\[0\] , 0x040ECE0
14 : MOV dword VM\_ESI , VM\_Stack\[0\]
15 : MOV dword  VM\_Stack\[0\] ,VM\_EBP
16 : MOV dword VM\_EDI , VM\_Stack\[0\]
17 : MOV dword  VM\_Stack\[0\] , 0x08
18 : MOV dword VM\_ECX , VM\_Stack\[0\]
19 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
20 : MOV dword  VM\_Stack\[1\] , 0x01
21 : TEST byte  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
22 : MOVZFSF VM\_VAR2 , VM\_VAR1
23 : TEST VM\_Stack\[9\]<>0 , JMP 10
24 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
25 : MOV dword  VM\_Stack\[1\] , 0x01
26 : SHR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
27 : MOV dword VM\_EDI , VM\_Stack\[0\]
28 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
29 : MOV dword  VM\_Stack\[1\] , 0x0EDB08320
30 : XOR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
31 : MOV dword VM\_EDI , VM\_Stack\[0\]
32 : MOVZFSF VM\_VAR2 , VM\_VAR1
33 : TEST VM\_Stack\[9\]<>0 , JMP 4
34 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
35 : MOV dword  VM\_Stack\[1\] , 0x01
36 : SHR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
37 : MOV dword VM\_EDI , VM\_Stack\[0\]
38 : MOV dword  VM\_Stack\[0\] ,VM\_ECX
39 : MOV dword  VM\_Stack\[1\] , 0x01
40 : SUB dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
41 : MOV dword VM\_ECX , VM\_Stack\[0\]
42 : MOVZFSF VM\_VAR2 , VM\_VAR1
43 : TEST VM\_Stack\[9\]<>0 , JMP 268435431
44 : MOV dword  VM\_Stack\[1\] ,VM\_EDI
45 : MOV dword  VM\_Stack\[0\] ,VM\_ESI
46 : MOV dword  \[VM\_Stack\[0\]\] , VM\_Stack\[1\]
47 : MOV dword  VM\_Stack\[0\] ,VM\_ESI
48 : MOV dword  VM\_Stack\[1\] , 0x04
49 : ADD dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
50 : MOV dword VM\_ESI , VM\_Stack\[0\]
51 : MOV dword  VM\_Stack\[0\] ,VM\_EBP
52 : MOV dword  VM\_Stack\[1\] , 0x01
53 : ADD dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
54 : MOV dword VM\_EBP , VM\_Stack\[0\]
55 : MOV dword  VM\_Stack\[0\] ,VM\_ESI
56 : MOV dword  VM\_Stack\[1\] , 0x040F0E0
57 : SUB dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
58 : MOVZFSF VM\_VAR2 , VM\_VAR1
59 : TEST VM\_Stack\[9\]<>0 , JMP 268435411
60 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
61 : MOV dword  VM\_Stack\[1\] , \[VM\_Stack\[0\]\]
62 : MOV dword VM\_ECX , VM\_Stack\[1\]
63 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
64 : MOV dword  VM\_Stack\[1\] , 0x04
65 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
66 : MOV dword VM\_EBX , VM\_Stack\[0\]
67 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
68 : MOV dword  VM\_Stack\[7\] , \[VM\_Stack\[0\]\]
69 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
70 : MOV dword  VM\_Stack\[1\] , 0x04
71 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
72 : MOV dword VM\_EBX , VM\_Stack\[0\]
73 : MOV VM\_EFL, VM\_VAR1
74 : VM\_RtlLeaveCriticalSection
```

## 2

```
1 : MOV VM\_VAR1 , VM\_EFL
2 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
3 : MOV dword  VM\_Stack\[1\] , 0x0FFFFFFFC
4 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
5 : MOV dword VM\_EBX , VM\_Stack\[0\]
6 : MOV dword  VM\_Stack\[1\] ,VM\_ECX
7 : MOV dword  \[VM\_Stack\[0\]\] , VM\_Stack\[1\]
8 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
9 : MOV dword  VM\_Stack\[1\] , 0x0C
10 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
11 : MOV dword  VM\_Stack\[0\] , \[VM\_Stack\[0\]\]
12 : MOV dword VM\_ECX , VM\_Stack\[0\]
13 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
14 : MOV dword  VM\_Stack\[1\] , 0x0FFFFFFFF
15 : OR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
16 : MOV dword VM\_EDI , VM\_Stack\[0\]
17 : MOV dword  VM\_Stack\[0\] ,VM\_ECX
18 : MOV dword  VM\_Stack\[1\] ,VM\_ECX
19 : TEST dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
20 : MOVZFSF VM\_VAR2 , VM\_VAR1
21 : TEST VM\_Stack\[9\]<>0 , JMP 63
22 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
23 : MOV dword  VM\_Stack\[1\] , 0x08
24 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
25 : MOV dword  VM\_Stack\[0\] , \[VM\_Stack\[0\]\]
26 : MOV dword VM\_ESI , VM\_Stack\[0\]
27 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
28 : MOV dword  VM\_Stack\[1\] , 0x0FFFFFFFC
29 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
30 : MOV dword VM\_EBX , VM\_Stack\[0\]
31 : MOV dword  VM\_Stack\[1\] ,VM\_ESP
32 : MOV dword  \[VM\_Stack\[0\]\] , VM\_Stack\[1\]
33 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
34 : MOV dword VM\_EBP , VM\_Stack\[0\]
35 : MOV dword  VM\_Stack\[0\] ,VM\_ESP
36 : MOV dword  VM\_Stack\[1\] ,VM\_ESP
37 : XOR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
38 : MOV VM\_EFL, VM\_VAR1
39 : MOV dword VM\_ESP , VM\_Stack\[0\]
40 : MOV dword  VM\_Stack\[0\] ,VM\_ESI
41 : MOV byte  VM\_Stack\[0\] , \[VM\_Stack\[0\]\]
42 : MOV byte VM\_ESP , VM\_Stack\[0\]
43 : MOV dword  VM\_Stack\[0\] ,VM\_EBP
44 : MOV dword  VM\_Stack\[1\] , 0x0FF
45 : AND dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
46 : MOV dword VM\_EBP , VM\_Stack\[0\]
47 : MOV dword  VM\_Stack\[0\] ,VM\_EBP
48 : MOV dword  VM\_Stack\[1\] ,VM\_ESP
49 : XOR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
50 : MOV VM\_EFL, VM\_VAR1
51 : MOV dword VM\_EBP , VM\_Stack\[0\]
52 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
53 : MOV dword  VM\_Stack\[1\] , 0x08
54 : SHR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
55 : MOV dword VM\_EDI , VM\_Stack\[0\]
56 : MOV dword  VM\_Stack\[0\] ,VM\_EBP
57 : MOV dword  VM\_Stack\[1\] , 0x04
58 : MUL VM\_Stack\[0\] , VM\_Stack\[0\]
59 : MOV dword  VM\_Stack\[1\] , 0x040ECE0
60 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
61 : MOV dword  VM\_Stack\[0\] , \[VM\_Stack\[0\]\]
62 : MOV dword VM\_EBP , VM\_Stack\[0\]
63 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
64 : MOV dword  VM\_Stack\[1\] ,VM\_EBP
65 : OR dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
66 : MOV VM\_EFL, VM\_VAR1
67 : MOV dword VM\_EDI , VM\_Stack\[0\]
68 : MOV dword  VM\_Stack\[0\] ,VM\_ESI
69 : MOV dword  VM\_Stack\[1\] , 0x01
70 : ADD dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
71 : MOV dword VM\_ESI , VM\_Stack\[0\]
72 : MOV dword  VM\_Stack\[0\] ,VM\_ECX
73 : MOV dword  VM\_Stack\[1\] , 0x01
74 : SUB dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
75 : MOV dword VM\_ECX , VM\_Stack\[0\]
76 : MOVZFSF VM\_VAR2 , VM\_VAR1
77 : TEST VM\_Stack\[9\]<>0 , JMP 268435411
78 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
79 : MOV dword  VM\_Stack\[1\] , \[VM\_Stack\[0\]\]
80 : MOV dword VM\_ESP , VM\_Stack\[1\]
81 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
82 : MOV dword  VM\_Stack\[1\] , 0x04
83 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
84 : MOV dword VM\_EBX , VM\_Stack\[0\]
85 : MOV dword  VM\_Stack\[0\] ,VM\_EDI
86 : NOT dword  VM\_Stack\[0\] , \[VM\_Stack\[1\]\]
87 : MOV dword VM\_EDI , VM\_Stack\[0\]
88 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
89 : MOV dword  VM\_Stack\[1\] , \[VM\_Stack\[0\]\]
90 : MOV dword VM\_ECX , VM\_Stack\[1\]
91 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
92 : MOV dword  VM\_Stack\[1\] , 0x04
93 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
94 : MOV dword VM\_EBX , VM\_Stack\[0\]
95 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
96 : MOV dword  VM\_Stack\[7\] , \[VM\_Stack\[0\]\]
97 : MOV dword  VM\_Stack\[0\] ,VM\_EBX
98 : MOV dword  VM\_Stack\[1\] , 0x04
99 : ADD VM\_Stack\[0\] , VM\_Stack\[0\]
100 : MOV dword VM\_EBX , VM\_Stack\[0\]
101 : MOV VM\_EFL, VM\_VAR1
```



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
