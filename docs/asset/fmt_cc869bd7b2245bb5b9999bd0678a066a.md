
# C 数据结构-单向链表

> 原文：[`www.pediy.com/kssd/pediy12/129415.html`](https://www.pediy.com/kssd/pediy12/129415.html)

适合于广大菜鸟 高手就无视吧

简单详解链表的难点

什么是链表?

链表是一种物理存储单元上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。链表由一系列结点（链表中每一个元素称为结点）组成，结点可以在运行时动态生成。每个结点包括两个部分：一个是存储数据元素的数据域，另一个是存储下一个结点地址的指针域。 相比于线性表顺序结构，链表比较方便插入和删除操作。

```c
class book

{

public:

  book()

  {

    id=NULL;

    price=NULL;

  }

  int id;//编号

          book *next; //指向下一结点指针

};

book *createlinst(book *head,int n);

void listoutput(book *head);

void findvalue(book *head,int value1);

int main()

{

  book *head;

  head=new book;

  head->next=NULL; //空表

  createlinst(head,2);

  listoutput(head);

  findvalue(head,2);

  getchar();

  getchar();

}

book *createlinst(book *head,int n)

{

     book *p;

     for (int i=1;i<=n;i++)

     {

       p=new book;

       p->id=i;

      head->next=p;

       head=p;

     }

     head->next=NULL;

  return head;

}

void listoutput(book *head) //链表遍历输出

{

  book *p;

  int i=0;

  p=head->next;

  while (p!=NULL)

  {

    i++;

      printf("节点%d\n",i);

        printf("价格为%d\n",p->id);

      p=p->next;

  }

}

void findvalue(book *head,int value1)//查找节点数据

{

  while (head->next!=NULL)

  {

    head=head->next;

    if (head->id==value1)

    {

      printf("%d",head->id);

      break;

    }

  }

}
```

无论是链表的遍历 删除 查找 建立 都少不了头节点

图中我们看到第一个结点 就是头结点 那么他是怎么建立的 别急往下看

```c
book *head;
head=new book; // 这里就是我们看到的第一个结点
head->next=NULL; //空表 // 可以看做为初始化
```

接着往下看

```c
book *p;

for (int i=1;i<=n;i++)

{

  p=new book;

  p->id=i;

 head->next=p;

  head=p; //这里可以看做 p1 第二次循环 p1.next=第二个结点地址

}

head->next=NULL; //最后一个结点建立完后设为尾结点
```

讲得有些乱 高手无视吧 - -
