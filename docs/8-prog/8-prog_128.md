
# 我写的队列程序

> 原文：[`www.pediy.com/kssd/pediy12/136441.html`](https://www.pediy.com/kssd/pediy12/136441.html)

主函数：

`main.cpp`

```cpp
#include <iostream.h>
#include "queue.h"

void main()
{
  Queue q;
  int i;
  cout<<"初始化状态:"<<endl;
  q.init();
  cout<<"队列长度="<<q.GetLength()<<endl;
  q.print();
  cout<<"加入 10 个元素"<<endl;
  for (i=0;i<10;i++)
  {
    q.EnQueue(i);
  }
  cout<<"当前长度是:"<<q.GetLength()<<endl;
  q.print();
  cout<<"删除 5 个元素"<<endl;
  for (i=0;i<5;i++)
  {
    q.DelQueue();
  }
  cout<<"当前长度是:"<<q.GetLength()<<endl;
  q.print();
  cout<<"在加 3 个元素"<<endl;
  for (i=0;i<3;i++)
  {
    q.EnQueue(i);
  }
  cout<<"当前长度是:"<<q.GetLength()<<endl;
  q.print();
}
```

`queue.cpp`

```cpp
#include <iostream.h>
#include "queue.h"

Queue::Node *Queue::CreateNode(int val)
{
  Node * pNew=new Node;
  if (pNew==NULL)
  {
    cout<<"Memory Insufficient!";
    return NULL;
  }
    pNew->elem=val;
    pNew->next=NULL;
    return pNew;
}

void Queue::EnQueue(int newElem)
{
  Node *pNew=CreateNode(newElem);
  if (pNew==NULL)
    return;
  if(elemTail == NULL)
  {
    //队列为空,即是头又是尾
    elemTail=elemHead=pNew;
  }else{
      elemTail->next=pNew;
      elemTail=pNew;//使当前节点成为尾节点
  }
  length++;//队列长度增加 1
}

int Queue::DelQueue()
{
  int ret=elemHead->elem;
  Node *pDel=elemHead;
  elemHead=elemHead->next;
  delete pDel;
  length--;
  return ret;
}

void Queue::init()
{
  elemHead=elemTail=NULL;
  length=0;
}

void Queue::print()
{
  Node *pNode=elemHead;
  cout<<"Queue:";
  while (pNode!=NULL)
  {
    cout<<pNode->elem<<"";
    pNode=pNode->next;
  }
  cout<<endl;
}
```

`queue.h`

```cpp
class Queue
{
  struct Node{
  public:
    int elem;
    Node *next;
  };

public:
  void init();
  void EnQueue(int newElem);  //入队列
  int DelQueue();        //元素出队列
  int GetLength(){return length;};
  void print();

private:
  Node *elemHead;        //首指针
  Node *elemTail;        //尾指针
  int length;          //队列长度
  Node *CreateNode(int val);
};
```

我自己写的，又照书改了一下，能给我一个邀请码吗？

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 队列例子.rar
|
