
```
// gcc -ggdb note.c -o note
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

char *notelist[1000];
int *freelist[1000];

int count = 0;

void backdoor() {
    puts("You hacked me!!");
    system("/bin/sh");
}

void malloc_add_note(){
    int i = count;
    char buf[8];
    int size;
    char *chunk;
    printf("Note size :");
    read(0, buf, 8);
    size = atoi(buf);
    chunk = (char *)malloc(size);
    if (!chunk)
    {
        puts("Alloca Error");
        exit(-1);
    }
    printf("Content :");
    read(0, chunk, size);
    puts("Success!");
    notelist[i] = chunk;
    count++;
}

void calloc_add_note(){
    int i = count;
    char buf[8];
    int size;
    char *chunk;
    printf("Note size :");
    read(0, buf, 8);
    size = atoi(buf);
    chunk = (char *)calloc(0x1,size);
    if (!chunk)
    {
        puts("Alloca Error");
        exit(-1);
    }
    printf("Content :");
    read(0, chunk, size);
    puts("Success!");
    notelist[i] = chunk;
    count++;
}

void del_note()
{
    char buf[4];
    int idx;
    printf("Index :");
    read(0, buf, 4);
    idx = atoi(buf);
    if (idx < 0 || idx >= count)
    {
        puts("Out of bound!");
        return;
    }
    if (notelist[idx] && (freelist[idx] != idx))
    {
        free(notelist[idx]);
        freelist[idx] = idx;
        puts("Success!");
        return;
    }
    else
    {
        puts("Can not double free!");
        return;
    }
}

void print_note()
{
    char buf[4];
    int idx;
    printf("Index :");
    read(0, buf, 4);
    idx = atoi(buf);
    if (idx < 0 || idx >= count)
    {
        puts("Out of bound!");
        return;
    }
    if (notelist[idx])
    {
        puts(notelist[idx]);
        return;
    }
}

void edit_note()
{
    char buf[8];
    int idx;
    int size;
    printf("Index :");
    read(0, buf, 4);
    idx = atoi(buf);
    if (idx < 0 || idx >= count)
    {
        puts("Out of bound!");
        return;
    }
    printf("Size :");
    read(0, buf, 8);
    size = atoi(buf);
    if (notelist[idx])
    {
        printf("Content :");
        read(0, notelist[idx], size);
        puts("Success!");
        return;
    }
}

void menu() {
    puts("----------------------");
    puts("       MY  NOTE       ");
    puts("----------------------");
    puts(" 1. Malloc Add note   ");
    puts(" 2. Delete note       ");
    puts(" 3. Print note        ");
    puts(" 4. Edit note         ");
    puts(" 5. Calloc Add note   ");
    puts(" 6. Exit              ");
    puts("--------Author:PIG-007");
    printf("Your choice :");
};

int main() {
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
    freelist[0] = 1001;
    char *heap_leak = (char *)(malloc(0x438));
    printf("Gift_Heap:%p\n",heap_leak);

    char *libc_leak = (char *)&printf;
    printf("Gift_Libc:%p\n",libc_leak);

    char *elf_leak = (char *)&main;
    printf("Gift_elf:%p\n",elf_leak);

    free(heap_leak);
    heap_leak = NULL;
    libc_leak = NULL;
    elf_leak = NULL;
    char buf[4];
    while (1) {
        menu();
        read(0, buf, 4);
        switch(atoi(buf))
        {
            case 1:
                malloc_add_note();
                break;
            case 2:
                del_note();
                break;
            case 3:
                print_note();
                break;
            case 4:
                edit_note();
                break;
            case 5:
                calloc_add_note();
                break;
            case 6:
                exit(0);
                break;
            default:
                puts("Invalid choice!");
                break;
        }
    }
    return 0;
}
```

对应 exp 设置：

|  |  |
| --- | --- |
