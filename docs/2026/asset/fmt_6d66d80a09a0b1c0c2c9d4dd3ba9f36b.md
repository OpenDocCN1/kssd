
![image.png](https://bbs.kanxue.com/upload/attach/202104/830671_6WMV6NYSP6P97PK.jpg)

## 4. stickystacks

> I made a [program](https://bbs.kanxue.com/elink@20fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9h3I4W2M7#2)9J5k6h3q4U0N6r3k6Q4x3X3g2U0L8#2)9J5c8X3p5$3k6h3j5K6j5$3f1K6k6U0f1I4k6U0p5#2x3X3f1&6x3o6k6W2x3e0R3&6y4h3c8T1y4$3y4X3x3o6R3K6y4r3f1H3x3o6g2W2j5X3b7I4z5o6V1H3k6h3j5H3k6U0l9@1y4X3c8T1x3U0R3&6z5r3f1H3x3r3c8V1k6U0y4Q4x3V1k6K6N6r3W2U0K9%4W2K6N6r3q4U0K9%4x3`.) that holds a lot of secrets... maybe even a flag!

> [Source](https://bbs.kanxue.com/elink@0a8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9h3I4W2M7#2)9J5k6h3q4U0N6r3k6Q4x3X3g2U0L8#2)9J5c8U0M7J5x3o6x3@1k6e0p5^5x3$3j5%4k6o6p5%4j5$3k6X3z5h3f1J5y4X3t1@1k6r3t1#2y4o6t1J5y4e0S2X3x3X3j5H3y4e0l9#2z5h3y4U0j5X3p5%4y4U0V1H3j5U0p5J5x3h3k6X3y4e0c8U0y4U0m8W2x3K6W2U0j5U0N6Q4x3V1k6K6N6r3W2U0K9%4W2K6N6r3q4U0K9%4y4Q4x3X3g2U0)

> Connect with `nc shell.actf.co 21820`, or visit `/problems/2021/stickystacks` on the shell server.

> Author: JoshDaBosh

### 步骤一：查看源码

将`flag`读取到了结构体中，结构体在栈里，但是没有打印它。能让我们输入6个字节的数据，没有栈溢出漏洞，但是存在 `printf(name)` 这个格式化字符串漏洞。所以，可以利用这个漏洞将`flag`泄漏出来。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
typedef struct Secrets {
char secret1[50];
char password[50];
char birthday[50];
char ssn[50];
char flag[128];
} Secrets;

int vuln(){
char name[7];
Secrets boshsecrets = {
.secret1 = "CTFs are fun!",
.password= "password123",
.birthday = "1/1/1970",
.ssn = "123-456-7890",
};

FILE *f = fopen("flag.txt","r");
if (!f) {
printf("Missing flag.txt. Contact an admin if you see this on remote.");
exit(1);
}
fgets(&(boshsecrets.flag), 128, f);

puts("Name: ");
fgets(name, 6, stdin);

printf("Welcome, ");
printf(name);
printf("\n");

return 0;
}

int main(){
setbuf(stdout, NULL);
setbuf(stderr, NULL);

vuln();

return 0;
}
```

### 步骤二：编写exp获取flag
