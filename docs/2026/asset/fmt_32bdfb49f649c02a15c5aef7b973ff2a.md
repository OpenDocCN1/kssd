
![](https://bbs.kanxue.com/upload/attach/201812/422740_8QEDWYUJ2X3A5G4.jpg)

规则`(C0 && C1) ? C2 : (C3 || C4)`和规则`(C0 && C1) ? (C3 || C4) : C2`

![](https://bbs.kanxue.com/upload/attach/201812/422740_YPVHKNCPRJ5PVB8.jpg)

### Type 7

规则`((C0 || C1) ? C2 : C3）|| C4`和规则`((C0 || C1) ? C2 : C3）&& C4`

![](https://bbs.kanxue.com/upload/attach/201812/422740_2CTHJQU7M6JNUMH.jpg)

规则`((C0 && C1) ? C2 : C3）|| C4`和规则`((C0 && C1) ? C2 : C3）&& C4`

![](https://bbs.kanxue.com/upload/attach/201812/422740_BWGV24HVVW8VCXQ.jpg)

规则`(C0 || C1) ? (C2 || C4) : (C3 && C4)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_CEMDSHEZ89JWA4Z.jpg)

规则`(C0 && C1) ? (C2 || C4) : (C3 && C4)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_BZWXNVMATD9GRSS.jpg)

### Type 8

规则`(C0 ? C1 : C2) ? C3 : C4`

![](https://bbs.kanxue.com/upload/attach/201812/422740_RM9VG5QYBKJJC36.jpg)

规则`C0 ? (C1 ? C2 : C3) : C4`

![](https://bbs.kanxue.com/upload/attach/201812/422740_82PD4BBSRD8FDV5.jpg)

规则`C0 ? C1 : (C2 ? C3 : C4)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_NF84NC2Y9JRXESE.jpg)

### Type 9

规则`(C0 ? C1 : C2) ? (C3 && C4) : C6`

![](https://bbs.kanxue.com/upload/attach/201812/422740_FUJCZFUHV7A3YU2.jpg)

规则`C0 ? (C1 ? C2 : C3) : (C5 && C6)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_5NJKQ6DXMF4WDQE.jpg)

规则`C0 ? (C1 && C2) : (C4 ? C5: C6)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_J62GXV8S67Y9V72.jpg)

### Type 10

规则`(C0 || C1) ? (C2 && C3）: (C4 ? C5: C6)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_KC3GB7J82CKBDVG.jpg)

规则`(C0 && C1) ? (C2 || C3）: (C4 ? C5: C6)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_2BWMS9CZJ28W2ZQ.jpg)

### Type 11

规则`(C0 ? C1 : C2) ? (C3 ? C4 : C5) :C6`

![](https://bbs.kanxue.com/upload/attach/201812/422740_ANM6RF3B9RJP7RX.jpg)

### Type 12

规则`(C0 ? C1 : C2) ? (C3 ? C4 : C5) :(C6 || C7)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_GJCF747S4XF3QQV.jpg)

### Type 13

规则`(C0 ? C1 : C2) ? (C3 ? C4 : C5) :(C6 ? C7 :C8)`

![](https://bbs.kanxue.com/upload/attach/201812/422740_BUB367NWQNHTDN8.jpg)

---

## 测试

测试规则`(C1 && C2) ? ((C3 ? C4 : C5) || (C8 || C9)) : ((C6 || C7) && (C8 || C9))`

![](https://bbs.kanxue.com/upload/attach/201812/422740_GYHS9Q6ABMD8R6K.jpg)

测试规则c代码：

```
int test ( int C1, int C2, int C3, int C4, int C5,

int C6, int C7, int C8, int C9,int C16 )

{

Entry:

int res = 0;

L1:
if ( C1 == 1 )
goto L2;
else
goto L6;

L2:
if ( C2 == 2 )
goto L3;
else
goto L6;

L3:
if ( C3 == 3 )
goto L4;
else
goto L5;

L4:
if ( C4 == 4 )
goto SeqT;
else
goto L8;

L5:
if ( C5 == 5 )
goto SeqT;
else
goto L8;

L6:
if ( C6 == 6 )
goto L8;
else
goto L7;

L7:
if ( C7 == 7 )
goto L8;
else
goto SeqF;

L8:
if ( C8 == 8 )
goto SeqT;
else
goto L9;

L9:
if ( C9 == 9 )
goto SeqT;
else
goto SeqF;

SeqT:
res = C16 + 1;
goto Exit;

SeqF:
res = C16 - 1;

Exit:
return res;
}
```

流程图:

![](https://bbs.kanxue.com/upload/attach/201812/422740_TNF9X4AAMGR8FGZ.jpg)

graphviz脚本dot表示:

```
digraph G{

Entry[shape=box,style=filled,color=white,label="res = 0"];

L1[label="C1 == 1"];
L2[label="C2 == 2"];
L3[label="C3 == 3"];
L4[label="C4 == 4"];
L5[label="C5 == 5"];
L6[label="C6 == 6"];
L7[label="C7 == 7"];
L8[label="C8 == 8"];
L9[label="C9 == 9"];
SeqT[label="res = C16 + 1"]
SeqF[label="res = C16 - 1"];

Exit[shape=box,style=filled,color=white,label="return res"];

subgraph cluster_1{

style=filled;

Entry:s -> L1:n[style=bold,color=yellow];

L1:w -> L2:n[style=bold,color=green,taillabel="T"];
L1:e -> L6:n[style=dashed,color=red,taillabel="F"];
L2:w -> L3:n[style=bold,color=green,taillabel="T"];
L2:e -> L6:n[style=dashed,color=red,taillabel="F"];
L3:w -> L4:n[style=bold,color=green,taillabel="T"];
L3:e -> L5:n[style=dashed,color=red,taillabel="F"];
L4:w -> SeqT:n[style=bold,color=green,taillabel="T"];
L4:e -> L8:n[style=dashed,color=red,taillabel="F"];
L5:w -> SeqT:n[style=bold,color=green,taillabel="T"];
L5:e -> L8:n[style=dashed,color=red,taillabel="F"];
L6:w -> L8:n[style=bold,color=green,taillabel="T"];
L6:e -> L7:n[style=dashed,color=red,taillabel="F"];
L7:w -> L8:n[style=bold,color=green,taillabel="T"];
L7:e -> SeqF:n[style=dashed,color=red,taillabel="F"];
L8:w -> SeqT:n[style=bold,color=green,taillabel="T"];
L8:e -> L9:n[style=dashed,color=red,taillabel="F"];
L9:w -> SeqT:n[style=bold,color=green,taillabel="T"];
L9:e -> SeqF:n[style=dashed,color=red,taillabel="F"];

SeqT:s -> Exit:n[style=bold,color=yellow];

SeqF:s -> Exit:n[style=bold,color=yellow];

}
}
```

可以生成未优化编译的x86汇编代码，然后用`IDA 7.0/7.1/7.2`反编译，查看`F5`后反编译代码中`goto`语句的数量

根据流图可知，归约合并后的反编译可能表示：

```
int test ( int C1, int C2, int C3, int C4, int C5,
int C6, int C7, int C8, int C9,int C16 )
{

int res = 0;

bool C10 = ( C1 ==1 ) && ( C2 == 2 );

bool C11 = ( C3 ==3 ) ? ( C4 == 4 ) : ( C5 == 5 );

bool C12 = ( C6 == 6 ) || ( C7 == 7 );

bool C13 = ( C8 == 8 ) || ( C9 == 9 );

bool C14 = C10 ? ( C11 || C13 ) : ( C12 && C13 );

if( C14 ) {

res = C16 + 1;

}
else {

res = C16 - 1;

}

return res;

}
```

---

## 总结

虽然可以根据规则进行模式匹配归约合并条件跳转语句，但该方法由于不可能覆盖所有的规则，因此归约后的流图的`goto`语句数量不可能最小化，还有，如果条件跳转语句在循环体内，情况会变得更加复杂，因此`Path-Sensitive Control Flow Analysis/Data Flow Analysis`应该是目前条件分支合并的最新方法，论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"就是采用了类似的方法。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-12-5 23:40

被`vasthao`编辑，原因： 错别字
