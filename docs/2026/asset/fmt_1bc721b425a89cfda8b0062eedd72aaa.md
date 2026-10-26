
```
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <string.h>

int main()
{
    int i;
    char name[255],buf[255],res[255]={0};
    printf("Please input your name: ");
    scanf("%s",name);

    for (i=0;i<strlen(name);i++) {
        itoa((int)(sqrt(name[i]*name[i]*name[i])+0.5),buf,10);
        strcat(res,buf);
    }

    for (i=0;i<10;i++) printf("%c",res[i]);

    printf("\nKeygen for ChangeWallpaper 1.04 by RoBa\nThanX\n");
    return 0;
}
```

算出一个可用的用户名：RoBa

注册码：7431169536

破这个东东不难，写这篇文章尤其是前面那几大段却花了我比破解多五六倍的时间，在这个OD大行其道的时代，希望我写的这些乱七八糟的东东能对在某些情况下不得不用SOFTICE和WIN98的兄弟有些启发，也算是不太成熟的经验之谈吧，抛砖引玉……

Finished. Quite Simple, isn't it? ![](https://bbs.kanxue.com/view/img/face/1.gif)

btw: 从联系方式看，作者也是学生，而且看界面的布局，应该是个newbie programmer，说不定比我还菜![](https://bbs.kanxue.com/view/img/face/1.gif) 各位如果有闲钱还是支持一下，不过看更新日期是02年了，作者应该也只是写着玩玩吧，就跟我写的那个破黑白棋似的，这也是我用它写破文的原因之一，当个CrackMe，呵呵。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
