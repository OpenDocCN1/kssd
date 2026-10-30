
# [原创]CTF2017第三题分析

这个题目设计的不错，感谢题主

## 题目与坑点

坑是smc，一开始不知道！不知道！不知道！坑是三个加号改减号，变量`v94`、`v8c`存放位置颠倒。因为存在约束，题目给出的原算法不可行，没有实数解，明知道有坑，不知坑在哪。

题目是VB Pcode，好多年没接触了，因为做题又拣回来，pcode主要是读指令，在虚拟机中打转，变量围绕stack操作，浮点数的存取计算则用到st寄存器。

工具用到看雪主页提供的VB Decompiler Pro用于看算法，ljtt VBParser用于看指令的具体地址，OD用于跟踪。

smc是后来关注到的，一开始改了`EIP`为1090，后来因为算不出来怀疑自己是不是中招了，恢复原状，结果跟踪时发现代码中有的加法怎么变减法，于是对内存下写断点，才发现有SMC，其中有`rdtsc`运行时间检查，细节没太关注，通过内存写断点可知，就是异或了上面提到了五处，如果你去跟，会一直在里面打转。OD加载后直接运行就行，在点验证之前设内存断点就可以跟踪了。

## 注册码与初始化

`Regcode`应为16个字母，16进制形式，分为两个dword，设前半部分为`L`，后半部分为`R`：

```
X＝0x6606D1F5， A=373414231.36250， B=874402299.931726
```

初始化代码为：

```
Private Sub Form_Load() '408040

  'Data Table: 407BB0

  loc_408025: global_76 = CDbl(&H6606D1F5)

  loc_40802E: global_100 = "373414231.362502"

  loc_408038: global_104 = "874402299.931726"

  loc_40803C: Exit Sub

End Sub
```

## 约束条件与公式

约束条件是：

`L, R < 0x80000000; L > R; X + R > L; L + R > X`

用到的公式是

```
loc_4083BE:   If (((L > CDbl(0)) And (R > CDbl(0))) And (L > R)) Then

  

  loc_4083D7:     var_94 = ((L + R) / CDbl(&HF4240))

  loc_4084CA:     var_8C = ((0.5 * ((2 * X^2 * L^2 - R^4 - X^4 + 2 * X^2 * R^2 + 2 * L^2 * R^2 - L ^ 4 ) ^ 0.5)) /  (X  + L  + R ))

  

  loc_40857A:     var_94 = ((X * L  * R) / (((X + L + R) * (X  + L - R ) * (X + R - L ) * ( L + R - X)) ^ 0.5 ))

  

  loc_4085FC:     If CBool(1 And (Format(var_94, "0.000000") = CVar(global_104))) Then

  loc_40860D:       1(0).Enabled = (Format(var_8C, "0.000000") = CVar(global_100))

  loc_408623:       1(0).Enabled = True

  loc_408639:       var_94(&HFF).Visible = var_8C

  loc_408641:     End If

  loc_408643:   End If
```

其中`((2 * X^2 * L^2 - R^4 - X^4 + 2 * X^2 * R^2 + 2 * L^2 * R^2 - L ^ 4) ^ 0.5)` = `((X + L + R) * (X + L - R) * (X + R - L) * (L + R - X)) ^ 0.5`，我是查百度来的，公式知识已经忘光了。

设上式为`DLT`，则：

`0.5 * DLT / (X + L + R) = A ...(1)`

`X * L * R / DLT = B ...(2)`

约去`DLT`，将`R`用`L`表示：

`R = (X + L) * 2 * A * B / (L * X - 2 * A * B)`

## 解题思路

穷举，`L`作为循环变量，将`L`、`R`代入1、2式，并以`string`来验证，1式过了再验2式，结果一致则为答案（实际上1过了，2也就过了）。

这里面还可以改进一下，即直接验证浮点数，因为`string`转换、比较用时还是长。

没VB，反出来的代码没法用，试了VBA，根本不行，程序一跑就失去响应，硬着头皮用C代码，主要是对浮点数心里没底，后来才发现不管什么语言，浮点数就是浮点数，相同的公式出的结果是一样的。

最后用到的主要代码：

```
int main(int argc, char* argv[])

{

	//其中的多余变量是在优化前用到的

	long unsigned double A, B, RT, X, DLT, RB, tmp;

	long unsigned int L, R, RL, RH, LL, LH;

	char sa[32], sb[32];

	char aa[32] = "373414231.362502";

	char bb[32] = "874402299.931726";

	char ch1;

	

	A=373414231.362502;

	B=874402299.931726;

	X=1711722997.0;

	cout<<"input LL"<<endl;

	cin>>LL;

	//1284400000

_next:

	LH=LL+100000000;

	cout<<"start :"<<LL<<"  end:  "<<LH<<endl;

	for(L=LL; L<LH; L++)

	{

		RB=(X + L) * 2 * A * B / (L * X - 2 * A * B);

		R=RB;

			DLT=sqrt(long unsigned double( (X + L + R) * (X  + L - R ) * (X + R - L ) * ( L + R - X)));

			tmp=0.5*DLT/(X+L+R)-A;

			if((tmp<1.0 )&& (tmp>-1.0))

			{

				sprintf(sa, "%13.6f", 0.5*DLT/(X+L+R));

				cout<<sa<<endl;

				if(!strcmp(sa, aa))

				{

					sprintf(sa, "%13.6f", X*R*L/DLT );

					cout<<"GOOD  "<<L<<"   "<<R<<endl;

					cout<<sa<<"     ";

					if(!strcmp(sa, bb))

					{

						continue;

					}

				}

			}

	}

	cout<<"continue ?";

	cin>>ch1;

	if(ch1!='n')

	{

		LL+=100000000;

		goto _next;			//比较丑，就不改do while....

	}

	cout<<"end..."<<endl;

	return 0;

}
```

## 编程中的弯路

开始时对浮点数理解不深，不敢直接用得到的`R`来直接参与运算，而是在`R`的左右一定范围内取数，都试一遍，造成运算效率急剧降低，我用的上下限各1/1M，每次验证5M个值，需要10多分钟，加上中间一时疏忽，`A`值又赋值错误了，前前后后用了有10个小时。比赛完成后，去掉`R`的上下限，直接用，半分钟内就能出结果 `T_T`。

## 总结

解这个题费时较长，在坑里呆了一天吧，好在没放弃。收获是对pcode，浮点理解更多一些，知道VB也可以加smc。公式嘛，现在记不住，将来也还会是记不住。

PS: 看到用matlab解题的，深感自己学识浅薄，自叹弗如啊。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
