
```c
#include<stdio.h>
int main()
{
    FILE *fp;
    char ch;
    // 如果文件不存在，给出提示并退出
    if((fp=fopen("D:\\demo.txt","rt")) == NULL)
    {
        puts("Fail to open file!");
        exit(0);
    }
    // 每次读取一个字节，直到读取完毕
    while((ch=fgetc(fp)) != EOF)
    {
        putchar(ch);
    }
    // 输出换行符
    putchar('\n');
    if(ferror(fp))
    {
        puts("读取出错");
    }
    else
    {
        puts("读取成功");
    }
    fclose(fp);
    return 0;
}
```

`EOF`是end of file的缩写，表示文件末尾，是在`stdio.h`中定义的宏，它的值是一个负数，往往是-1。`EOF`不绝对是-1，也可以是其他负数，这要看编译器的实现。`EOF`本来表示文件末尾，意味着读取结束，但是很多函数在读取出错时也返回`EOF`，我们可以借助`stdio.h`中的两个函数来判断，分别是`feof()`和`ferror()`。

```c
/*判断文件操作是否出错，出错时返回非零值，否则返回零值。*/
int ferror (FILE* fp);
/*判断文件内部指针是否指向了文件末尾，
当指向文件末尾时返回非零值，否则返回零值*/
int feof (FILE* fp);
```

## 以字符串形式读写文件

`fgetc()`和`fputc()`函数每次只能读写一个字符，速度较慢，实际开发中往往是每次读写一个字符串或者一个数据块，这样能明显提高效率。`fgets()`函数用来从指定的文件中读取一个字符串，并保存到字符数组中；`fputs()`函数用来向指定的文件写入一个字符串。

```c
#include <stdio.h>
#include <stdlib.h>
#define N 100
int main()
{
    FILE *fp;
    char str[N+1];
    if((fp=fopen("d:\\demo.txt","rt")) == NULL)
    {
        puts("Fail to open file!");
        exit(0);
    }
    while(fgets(str, N, fp) != NULL)
    {
        printf("%s", str);
    }
    fclose(fp);
    return 0;
}
```

```c
#include<stdio.h>
int main()
{
    FILE *fp;
    char str[102] = {0}, strTemp[100];
    if((fp=fopen("D:\\demo.txt", "at+")) == NULL)
    {
        puts("Fail to open file!");
        exit(0);
    }
    printf("Input a string:");
    gets(strTemp);
    strcat(str, "\n");
    strcat(str, strTemp);
    fputs(str, fp);
    fclose(fp);
    return 0;
}
```

需要注意的是，读取到的字符串会在末尾自动添加`'\0'`，`n`个字符也包括`'\0'`。也就是说，实际只读取到了`n-1`个字符，如果希望读取100个字符，`n`的值应该为101。另外，在读取到`n-1`个字符之前如果出现了换行，或者读到了文件末尾，则读取结束。这就意味着，不管`n`的值多大，`fgets()`最多只能读取一行数据，不能跨行。在C语言中，没有按行读取文件的函数，我们可以借助`fgets()`，将`n`的值设置的足够大，每次就可以读取到一行数据。

## 以数据块形式读写文件

如果希望读取多行内容，需要使用`fread()`函数，相应地写入函数为`fwrite()`。对于Windows系统，使用`fread()`和`fwrite()`时应该以二进制的形式打开文件。

```c
//从指定文件中读取块数据
size_t fread(void* ptr,size_t size,size_t count,FILE *fp);
//向文件中写入块数据
size_t fwrite( void* ptr,size_t size,size_t count,FILE *fp);
//所谓块数据，也就是若干个字节的数据，可以是一个字符，可以是一个字符串，可以是多行数据，并没有什么限制。
```

### 参数说明

- `ptr`：为内存区块的指针，它可以是数组、变量、结构体等。`fread()`中的`ptr`用来存放读取到的数据，`fwrite()`中的`ptr`用来存放要写入的数据。
- `size`：表示每个数据块的字节数。
- `count`：表示要读写的数据块的块数。
- `fp`：表示文件指针。
- `size_t`是在`stdio.h`和`stdlib.h`头文件中使用`typedef`定义的数据类型，表示无符号整数，也即非负数，常用来表示数量。

### 返回值说明

返回成功读写的块数`count`，如果返回值小于`count`：

- 对于`fwrite()`来说，肯定发生了写入错误，可以用`ferror()`函数检测。
- 对于`fread()`来说，可能读到了文件末尾，可能发生了错误，可以用`ferror()`或`feof()`检测。

示例：从键盘输入两个学生数据，写入一个文件中，再读出这两个学生的数据显示在屏幕上。

```c
#include<stdio.h>
#define N 2
struct stu
{
    char name[10]; //姓名
    int num; //学号
    int age; //年龄
    float score; //成绩
}boya[N], boyb[N], *pa, *pb;
int main()
{
    FILE *fp;
    int i;
    pa = boya;
    pb = boyb;
    if((fp=fopen("d:\\demo.txt", "wb+")) == NULL)
    {
        puts("Fail to open file!");
        exit(0);
    }
    //从键盘输入数据
    printf("Input data:\n");
    for(i=0; i<N; i++,pa++)
    {
        scanf("%s %d %d %f",pa->name,&pa->num,&pa->age,&pa->score);
    }
    //将数组 boya 的数据写入文件
    fwrite(boya, sizeof(struct stu), N, fp);
    //将文件指针重置到文件开头
    rewind(fp);
    //从文件读取数据并保存到数据 boyb
    fread(boyb, sizeof(struct stu), N, fp);
    //输出数组 boyb 中的数据
    for(i=0; i<N; i++,pb++)
    {
        printf("%s %d %d %f\n",pb->name,pb->num,pb->age,pb->score);
    }
    fclose(fp);
    return 0;
}
```

## 格式化读写文件

`fscanf()`和`fprintf()`函数与前面使用的`scanf()`和`printf()`功能相似，都是格式化读写函数，两者的区别在于`fscanf()`和`fprintf()`的读写对象不是键盘和显示器，而是磁盘文件。

示例：用`fscanf()`和`fprintf()`函数来完成对学生信息的读写。

```c
#include<stdio.h>
#define N 2
struct stu
{
    char name[10];
    int num;
    int age;
    float score;
} boya[N], boyb[N], *pa, *pb;
int main()
{
    FILE *fp;
    int i;
    pa=boya;
    pb=boyb;
    if( (fp=fopen("D:\\demo.txt","wt+")) == NULL )
    {
        puts("Fail to open file!");
        exit(0);
    }
    //从键盘读入数据，保存到boya
    printf("Input data:\n");
    for(i=0; i<N; i++,pa++)
    {
        scanf("%s %d %d %f", pa->name, &pa->num, &pa->age, &pa->score);
    }
    pa = boya;
    //将boya中的数据写入到文件
    for(i=0; i<N; i++,pa++)
    {
        fprintf(fp,"%s %d %d %f\n", pa->name, pa->num, pa->age, pa->score);
    }
    //重置文件指针
    rewind(fp);
    //从文件中读取数据，保存到boyb
    for(i=0; i<N; i++,pb++)
    {
        fscanf(fp, "%s %d %d %f\n", pb->name, &pb->num, &pb->age, &pb->score);
    }
    pb=boyb;
    //将boyb中的数据输出到显示器
    for(i=0; i<N; i++,pb++)
    {
        printf("%s  %d  %d  %f\n", pb->name, pb->num, pb->age, pb->score);
    }
    fclose(fp);
    return 0;
}
```

## 随机读写文件

实际开发中经常需要读写文件的中间部分，要解决这个问题，就得先移动文件内部的位置指针，再进行读写。这种读写方式称为随机读写，实现随机读写的关键是要按要求移动位置指针，这称为文件的定位。

```c
//将位置指针移动到文件开头
void rewind ( FILE *fp );
//将位置指针移动到任意位置
int fseek ( FILE *fp, long offset, int origin );
```

参数说明：

- `fp` 为文件指针，也就是被移动的文件。
- `offset`为偏移量，也就是要移动的字节数。之所以为`long`类型，是希望移动的范围更大，能处理的文件更大。`offset`为正时，向后移动；`offset`为负时，向前移动。
- `origin`为起始位置，也就是从何处开始计算偏移量。
- C语言规定的起始位置有三种，分别为文件开头、当前位置和文件末尾，每个位置都用对应的常量来表示：

  ![起始位置](https://bbs.kanxue.com/upload/attach/202106/878023_USAV3R4RUPA5KZG.png)

示例：从键盘输入三组学生信息，保存到文件中，然后读取第二个学生的信息。
