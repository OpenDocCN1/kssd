
## `multiply3` 函数源码

```
// need this to return 0

// need to see how this affect the cc1 and cc2

signed int __userpurge multiply3@<eax>(CriticalClass *cc1@<eax>, CriticalClass *cc2@<ecx>, CriticalClass *cc1_)

{

  void *v3; // esp@1

  CriticalClass *cc2_1; // esi@1

  int i; // edi@1

  int cc2Rec; // ecx@1

  int cc1Rec; // eax@1

  DWORD *cc1_arr2; // edx@4

  signed int v10; // ebp@5

  int v11; // edx@7

  DWORD *i_cc1_arr2; // [sp+0h] [bp-4030h]@4

  CriticalClass cc2_tmp; // [sp+4h] [bp-402Ch]@1

  CriticalClass sum; // [sp+2014h] [bp-201Ch]@1

  CriticalClass *cc1_1; // [sp+4024h] [bp-Ch]@1

  int (__cdecl *v16)(EXCEPTION_RECORD *, struct EHRegistrationNode *, struct _CONTEXT *, void *); // [sp+4028h] [bp-8h]@1

  int v17; // [sp+402Ch] [bp-4h]@1

  v17 = -1;

  v16 = sub_4075F6;

  cc1_1 = cc1;

  v3 = alloca(0x4024);

  cc2_1 = cc2;

  constructor(&cc2_tmp);

  i = 0;

  v17 = 0;

  constructor3(&sum, 0);

  cc2Rec = cc2_1->rec;

  LOBYTE(v17) = 1;

  cc1Rec = cc1_->rec;

  if ( cc1Rec + cc2Rec > 1024 )                 // cc1->rec + cc2->rec <= 1024

  {

LABEL_2:

    setVt(&sum);                                // 出问题，就返回1，但基本上不会有那么大的数。。。

    setVt(&cc2_tmp);

    return 1;

  }

  if ( cc1Rec > 0 )                             // i < ccRec

  {

    cc1_arr2 = cc1_->arr2;

    i_cc1_arr2 = cc1_->arr2;

    do

    {

      v10 = cc1_->arr1[*i_cc1_arr2];

                                                // 在循环中，i_cc1_arr2 总是 &cc1->arr2[i]

      cc_cpy(&cc2_tmp, (int)cc1_arr2, cc2_1);//第二个参数没用，IDA抽了

      lsr_critical_class(&cc2_tmp, i);

      multiply2(&cc2_tmp, v10);                 // cc1->arr1[arr2[i]]

      add(&sum, &cc2_tmp);

      if ( cc2_1->rec >= 1024 )

        goto LABEL_2;

      ++i;

      ++i_cc1_arr2;

    }

    while ( i < cc1_->rec );            // i从0到cc1->rec

    /*

    这个跟前面那个multiply2很像，只不过是乘上另外一个CriticalClass

    设this为x

    也是从最低位开始，x*10^i*cc1的第i位，加起来加到sum

    一样的，也是把x提出来，最后发现结果就是cc1*x

    */

  }

  init_class(cc2_1);

  cc_cpy(cc2_1, v11, &sum);//this*=cc1

  setVt(&sum);

  setVt(&cc2_tmp);

  return 0;

}
```

## 回到 `main` 看逻辑

设 `input` 输入为 `x`

`x` 先乘上 9, 然后再乘 `x`, 然后再乘 9, 就是 `81*x*x`

设这玩意为 `y`

1. `y` 一定是奇数位数
2. `y` 的（位数>>1）所存的数字一定要跟 `x` 的第 0 位的数字一样
3. 如果不一样，还有几次循环的机会，到目前为止就只能分析到这么多

## 分析 `critical_4013E0`

最后看这个 `critical_4013E0`，这个耗了我最久的时间。。。首先反编译效果很差，必须要用 `asm` 自己看，而且他分两种情况第一种是第一次调用第二种是第二次调用，我们要一次一次地看。其中 `this` 是 `81*x*x`，`input` 是 `x`

```
; unsigned int __thiscall critical_4013E0(CriticalClass *critical_class, CriticalClass *cc_input, int a3, int one, int j_, char a6)

critical_4013E0 proc near

var_4= dword ptr -4

cc_input= dword ptr  4

a3= dword ptr  8

one= dword ptr  0Ch

j_= dword ptr  10h

a6= byte ptr  14h

i = esi

j = edx

this = edi

input = ecx

push    input

mov     j, [esp+4+j_]   ; edx=input_rec

push    i

push    this

mov     this, input     ; edi=this

mov     input, [esp+0Ch+cc_input] ; ecx=input

mov     [esp+0Ch+var_4], 0 ; v4=0

test    j, j

jle     short loc_401477 ; if (rec > 0)，小于等于直接飞走return，这个不可能<=0

mov     eax, [esp+0Ch+one] ; eax=1

push    ebx

push    ebp

mov     ebp, [esp+14h+a3] ; ebp=a4

lea     ebx, [eax+j-1]  ; ebx=input_rec

mov     i, eax          ; esi = 1

mov     [esp+14h+j_], ebx ; 编译器抽风没用系列

sub     ebp, eax        ; ebp = a4 - 1
```

### 进入 `do while` 循环

先看看 `do while` 循环的变量范围

```
loc_401468:

mov     eax, [esp+14h+j_]

dec     eax

inc     i               ; i 从 1 到 input->rec - 1

dec     j               ; j 从 input->rec - 1 到 1

                        ; j == input->rec - i

mov     [esp+14h+j_], eax ; j_--

jnz     short DOWHILE   ; j == j_ always

                        ; 循环次数为 input->rec - 1 次
```

```
DOWHILE:                

mov     ebx, [this+4]   ; ebx=this->rec

lea     eax, [i+ebp]    ; eax=a4 - 1 + i

cmp     eax, ebx

jl      short loc_401426

mov     eax, [esp+14h+var_4] ; a4-1+i >= this->rec

                             ; 不可能到这里

                             ;因为i最大是input->rec-1，a4有两种情况0(第二次调用)或者this->rec-input->rec + 1(第一次)

                             ;所以a4-1+i最大是trec-irec+1-1+irec-1，等于trec-1，所以不可能大于等于trec

                             ;PS:待会为了方便起见this->Rec就是trec,input->rec就是irec

inc     eax

xor     ebx, ebx

mov     [esp+14h+var_4], eax ; ebx=0

                        ; v4++

jmp     short loc_401428

loc_401426:             ; 会永远到这里

                        ; a4-1+i < this->rec

mov     ebx, eax        ; ebx=a4-1+i

                        ; 第1次: ebx=cc-input+i

                        ; 第2次: ebx=i-1

                        ;ebx待会会用来作为下标，待会说

loc_401428:

call    getRec

cmp     i, eax

jl      short loc_40143E

mov     eax, [esp+14h+var_4] ; if (i >= input->rec)

                             ; 同样不可能到达，因为i最大为irec-1

inc     eax

mov     [esp+14h+var_4], eax ; ++v4

xor     eax, eax

jmp     short loc_40144C

loc_40143E:             ; if (i < input->rec),永远到这

mov     al, [esp+14h+a6];a6第一次是0，第二次是1

test    al, al

mov     eax, [esp+14h+j_] ; 2nd time第二次，eax会是j

jnz     short loc_40144C

mov     eax, i          ; 1st time第一次，eax会是i

loc_40144C:

mov     ebx, [this+ebx*4+1008h]

mov     eax, [input+eax*4+1008h]

mov     ebx, [this+ebx*4+8] ; this->arr1[arr2[ebx]]

cmp     ebx, [input+eax*4+8] ; input->arr1[arr2[eax]]

                        ; 现在分析这个就差不多了，之前说我想让两次调用返回相反数

                        ;因为这里只有加，而且不大可能溢出，所以只有让他们都返回0

                        ;所以我们不能让v4++，（v4是返回值的一部分，待会说）

                        ;所以一定要让他们总是相等

                        ; 第一次  :input->arr1[arr2[i]]

                        ;          this-> arr1[arr2[cc_Rec-input_rec + i]]

                        ;          i = [1..input->rec-1]

                        ;所以x的1位到irec-1位要 和 y(81x^2)的trec-irec+1位到trec-1位一致

                        ;

                        ; 第二次  :input->arr1[arr2[input_rec-i]]

                        ;          this-> arr1[arr2[i - 1]]

                        ;          i = [1..input->rec-1]

                        ;所以x的irec-1位到1位要 和 y的0位到irec-2位一致，注意这里是倒序一致

jz      short loc_401468

inc     [esp+14h+var_4] ; 爆破点，nop掉直接过，当然要满足最前面的那两条规则
```

### 最后看看返回值

```
loc_401477:

mov     eax, [input+200Ch];还记得那个GetTickCount不

mov     i, [this+200Ch]

mov     input, [esp+0Ch+var_4]

xor     eax, i    ;取出来xor右移7位

shr     eax, 7    ;基本上这个会是0，因为电脑很快的，但是main里面的while循环跑太多次就不一定了。。。所以上面我分析的要求最好一次达标，就是呢，在y=81x^2的时候达标

pop     this

add     eax, input;是0的话，就返回v4的值

pop     i

pop     input

retn    14h
```

好的，结合前面的两条标准，加上最后分析出的那条标准，我们写出 `Python` 爆破器。。。

```
def takeReverseOrder(num, x):

	ret = 0

	i = 0

	while (i <= x):

		ret = ret * 10

		ret = ret + (num % 10)

		i = i + 1

		num = num / 10

	return ret

def getNumDigit(x):

	ret = 0

	while (x != 0):

		ret = ret + 1

		x = x / 10

	return ret

def getDigit(x, n):

	return (x / pow(10,n)) % 10

def ifMatch(number, nextNum):

	irec = getNumDigit(number)

	trec = getNumDigit(nextNum)

	tmp = number / 10

	nnumD = getNumDigit(nextNum)

	if (tmp == nextNum / pow(10, trec - irec + 1)

		and tmp == takeReverseOrder(nextNum, irec - 2)

		and nnumD % 2 == 1

		and number % 10 == getDigit(nextNum, nnumD / 2)):

		return True

	return False

number = 10000000

while True:

	nextNum = number * number * 81

	if (ifMatch(number, nextNum)):

		print ("correct: %d" % number)

		break

	else:

		if (number % 100000 == 0):

			print ("wrong: %d" % number)

	number = number + 1
```

最后输出为

```
wrong: 11800000

wrong: 11900000

wrong: 12000000

wrong: 12100000

wrong: 12200000

wrong: 12300000

correct: 12345679
```

PS:汇编太菜，F5用的多，略脚本小子

而且这第三题一出我一看就知道不在我的能力范围内。。。所以直接give up。。。就写了这篇分析。。。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
