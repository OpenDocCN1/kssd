
# 编程中多叉树的使用和遍历

> 原文：[`www.pediy.com/kssd/pediy10/80173.html`](https://www.pediy.com/kssd/pediy10/80173.html)

一个简单的多叉树类, 结合双向链表实现, 不足之处请高手指点, 看代码..  

引用: /\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

`created: 2009/01/09`  

`created: 9:1:2009 12:42`  

`author: cooldiyer`  

`site: http://www.xcodez.com/`  

`purpose: 多叉树类文件`  

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  

```c
#include <windows.h>  

#include <stdio.h>  

VOID  
FORCEINLINE  
InitializeListHead(  
           IN PLIST_ENTRY ListHead  
           )  
{  
    ListHead->Flink = ListHead->Blink = ListHead;  
}  

#define IsListEmpty(ListHead) \  
((ListHead)->Flink == (ListHead))  

VOID  
FORCEINLINE  
RemoveEntryList(  
        IN PLIST_ENTRY Entry  
        )  
{  
    PLIST_ENTRY Blink;  
    PLIST_ENTRY Flink;  
    Flink = Entry->Flink;  
    Blink = Entry->Blink;  
    Blink->Flink = Flink;  
    Flink->Blink = Blink;  
}  

VOID  
FORCEINLINE  
InsertTailList(  
         IN PLIST_ENTRY ListHead,  
         IN PLIST_ENTRY Entry  
         )  
{  
    PLIST_ENTRY Blink;  
    Blink = ListHead->Blink;  
    Entry->Flink = ListHead;  
    Entry->Blink = Blink;  
    Blink->Flink = Entry;  
    ListHead->Blink = Entry;  
}  

VOID  
FORCEINLINE  
InsertHeadList(  
         IN PLIST_ENTRY ListHead,  
         IN PLIST_ENTRY Entry  
         )  
{  
    PLIST_ENTRY Flink;  
    Flink = ListHead->Flink;  
    Entry->Flink = Flink;  
    Entry->Blink = ListHead;  
    Flink->Blink = Entry;  
    ListHead->Flink = Entry;  
}  

typedef struct tagTREENODE  
{  
  LPARAM      lParam;      // 关联的值  
  int        cChildren;    // 子节点个数  
  LIST_ENTRY    ListEntry;    // 同一等级的节点  
  LIST_ENTRY    ChildListEntry;  // 子节点头  
  tagTREENODE *  pParentNode;  // 父节点  
} TREENODE, *PTREENODE;  

#define TN_ROOT            ((HTREENODE)(ULONG_PTR)-0x10000)  

#define TN_FIRST           ((HTREENODE)(ULONG_PTR)-0x0FFFF)  

#define TN_LAST            ((HTREENODE)(ULONG_PTR)-0x0FFFE)  

typedef PTREENODE  HTREENODE;  

class CTree  
{   
public:  
  CTree() {  
    m_nCount = 0;  
    m_nRootChildCount = 0;  
    InitializeListHead(&m_ListHead);  
  }  
  ~CTree() {  
    DeleteAllNodes();  
  }  
private:  
  int m_nCount;  
  int  m_nRootChildCount;  
  LIST_ENTRY m_ListHead;  
public:  
  int getCount() {  
    return m_nCount;  
  }  
  int  getChildNodeCount(HTREENODE hNode) {  
    if (isRootNode(hNode)) {  
      return m_nRootChildCount;  
    }  
    else {  
      return hNode->cChildren;  
    }  
  }  
  HTREENODE getChildNode(HTREENODE hNode) {  
    if (NodeHasChildren(hNode) > 0) {  
      if (isRootNode(hNode)) {  
        return CONTAINING_RECORD(m_ListHead.Flink, TREENODE, ListEntry);  
      }  
      else {  
        return CONTAINING_RECORD(hNode->ChildListEntry.Flink, TREENODE, ListEntry);  
      }  
    }  
    else {  
      return NULL;  
    }  
  }  
  BOOL isRootNode(HTREENODE hNode) {  
    return TN_ROOT == hNode;  
  }  
  HTREENODE getRootNode(){  
    return TN_ROOT;  
  }  
  HTREENODE GetParentNode(HTREENODE hNode) {  
    return hNode->pParentNode;  
  }  
  HTREENODE getNextNode( HTREENODE hNode ) {  
    PLIST_ENTRY  pListHead;  
    if (isRootNode(hNode)) {  
      return NULL;  
    }  
    if (isRootNode(hNode->pParentNode)) {  
      pListHead = &m_ListHead;  
    }  
    else {  
      pListHead = &hNode->pParentNode->ChildListEntry;  
    }  
    PLIST_ENTRY  pNext = hNode->ListEntry.Flink;  
    if (pListHead == pNext) {  
      return NULL;  
    }  
    else {  
      return CONTAINING_RECORD(pNext, TREENODE, ListEntry);  
    }  
  }  
  BOOL NodeHasChildren( HTREENODE hNode ) {  
    if (isRootNode(hNode)) {  
      return m_nRootChildCount > 0;  
    }  
    else {  
      return hNode->cChildren > 0;  
    }  
  }  
  HTREENODE InsertNode(LPARAM lparam, HTREENODE hParent = TN_ROOT, HTREENODE hInsertAfter = TN_LAST ) {  
    TREENODE *  pTreeNode = new TREENODE;  
    pTreeNode->cChildren = 0;  
    pTreeNode->lParam = lparam;  
    pTreeNode->pParentNode = hParent;  
    InitializeListHead(&pTreeNode->ListEntry);  
    InitializeListHead(&pTreeNode->ChildListEntry);  
    PLIST_ENTRY  pListEntry = NULL;  
    if (TN_ROOT == hParent) {  
      pListEntry = &m_ListHead;  
      m_nRootChildCount++;  
    }  
    else {  
      pListEntry = &hParent->ChildListEntry;  
      hParent->cChildren++;  
    }  
    if (TN_LAST == hInsertAfter) {  
      InsertTailList(pListEntry, &pTreeNode->ListEntry);  
    }  
    else if (TN_FIRST == hInsertAfter){  
      InsertHeadList(pListEntry, &pTreeNode->ListEntry);  
    }  
    else {  
      InsertHeadList(&hInsertAfter->ListEntry, &pTreeNode->ListEntry);  
    }  
    m_nCount++;  
    return pTreeNode;  
  }  
  DWORD GetNodeData( HTREENODE hNode ) {  
    if (isRootNode(hNode)) {  
      return -1;  
    }  
    else {  
      return hNode->lParam;  
    }  
  }  
  BOOL SetNodeData( HTREENODE hNode, DWORD dwData ) {  
    if (isRootNode(hNode)) {  
      return FALSE;  
    }  
    else {  
      hNode->lParam = dwData;  
    }  
    return TRUE;  
  }  
  BOOL DeleteAllNodes() {  
    return DeleteNode(TN_ROOT);  
  }  
  BOOL DeleteNode(HTREENODE hNode) {  
    HTREENODE hNext = getChildNode(hNode);  
    while (hNext) {  
      HTREENODE hNextNode = getNextNode(hNext);  
      DeleteNode(hNext);  
      hNext = hNextNode;  
    }  
    if (!isRootNode(hNode)) {  
      if (isRootNode(hNode->pParentNode))  {  
        m_nRootChildCount--;  
      }  
      else {  
        hNode->pParentNode->cChildren--;  
      }  
      m_nCount--;  
      RemoveEntryList(&hNode->ListEntry);  
      delete hNode;  
    }  
    return TRUE;  
  }  
};  

void PrintNode(CTree * pTree, HTREENODE hNode, BOOL bIsRecursive)  
{  
  HTREENODE hNext = pTree->getChildNode(hNode);  
  while (hNext)  
  {  
    printf("0x%X 0x%X\t%d\n", hNext, pTree->GetParentNode(hNext), pTree->GetNodeData(hNext));  
    if (bIsRecursive){  
      PrintNode(pTree, hNext, bIsRecursive);  
    }  
    hNext = pTree->getNextNode(hNext);  
  }  
}  

int main(int argc, char* argv[])  
{  
  CTree  tree;  
  HTREENODE hTreeNode = tree.InsertNode(1);  
  tree.InsertNode(2);  
  tree.InsertNode(3);  
  HTREENODE hTreeChild = tree.InsertNode(4);  
  tree.InsertNode(6, hTreeChild);  
  tree.InsertNode(5, hTreeChild, TN_FIRST);  
  HTREENODE hNewChild = tree.InsertNode(7, hTreeChild);  
  tree.InsertNode(8, hNewChild);  
  tree.InsertNode(9, hNewChild);  
  PrintNode(&tree, tree.getRootNode(), TRUE);  
  return 0;  
}
```
