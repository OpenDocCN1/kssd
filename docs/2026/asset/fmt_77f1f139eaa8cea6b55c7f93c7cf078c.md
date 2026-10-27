
资源这一块，由于显示是树形控件，则数据以树的结构组织，显示更为方便。问题关键是怎么组织成树的结构，我这里定义这个兄弟树结构体

```cpp
/*********************************************************************/

//树节点
typedef struct _TreeNode
{
        char  data[256];//节点数据
        _TreeNode* pFirstChild;  //第一个子节点
        _TreeNode* pNextBrother;//兄弟节点

        _TreeNode()
        {
                pFirstChild=pNextBrother=NULL;
        }

/*********************************************************************/
/*
        把资源数据以树的结构组织
*/
void PeTool::CreateResourceTree()
{
        // 根节点
        m_pResourceHeadNode=new TREENODE;
        //资源地址
        PIMAGE_RESOURCE_DIRECTORY  pReDirectory=(PIMAGE_RESOURCE_DIRECTORY)GetResourceOffset();
        //资源地址，根节点为参数 添加个个节点
        AddNodeToResourceTree(m_pResourceHeadNode,pReDirectory,0);
}

/*********************************************************************/
/*
                参数：
                TREENODE*pNode,  父节点
                PIMAGE_RESOURCE_DIRECTORY pDirectory,  资源头
                int depath   深度
*/
void PeTool::AddNodeToResourceTree(TREENODE*pNode,PIMAGE_RESOURCE_DIRECTORY pDirectory,int depath )
{
        //得到  PIMAGE_RESOURCE_DIRECTORY_ENTRY  地址
        PIMAGE_RESOURCE_DIRECTORY_ENTRY pEntry=(PIMAGE_RESOURCE_DIRECTORY_ENTRY)((DWORD)pDirectory+sizeof(IMAGE_RESOURCE_DIRECTORY));

        WORD    NumberOfNamedEntries;
        WORD    NumberOfIdEntries;
        NumberOfNamedEntries=pDirectory->NumberOfNamedEntries;
        NumberOfIdEntries=pDirectory->NumberOfIdEntries;
        //根据每一个资源目录有多少个来申请多少个节点
        _TreeNode* NodeArry=new _TreeNode[NumberOfNamedEntries+NumberOfIdEntries];
        pNode->pFirstChild=NodeArry;

        TREENODE* temp=NodeArry;
        for(int i=1;i<NumberOfNamedEntries+NumberOfIdEntries;i++)
        {
                temp->pNextBrother=NodeArry+i;
                temp=NodeArry+i;
        }

        temp=NodeArry;
        //采用先序遍历 来添加节点
        for (int i=0;i<NumberOfNamedEntries+NumberOfIdEntries;i++)
        {
                PIMAGE_RESOURCE_DIRECTORY_ENTRY  pEntryitem=pEntry+i;

                DWORD name=pEntryitem->Name;

                if(pEntryitem->DataIsDirectory)
                {
                        if(depath==0)//第一级目录 有id 标示和字符串标示
                        {
                        if(pEntryitem->NameIsString)//字符串标示
                        {
                                PIMAGE_RESOURCE_DIR_STRING_U pString=(PIMAGE_RESOURCE_DIR_STRING_U)RSAtoOffset(pEntryitem->NameOffset);
                                WideCharToMultiByte(CP_ACP,WC_COMPOSITECHECK |WC_DEFAULTCHAR ,
                                                                                        pString->NameString,pString->Length,
                                                                                        temp->data,
                                                                                        256,
                                                                                        NULL,
                                                                                        NULL);
                                temp->data[pString->Length]=0;
                        }
                        else //id标示
                        {
                                if(pEntryitem->Name>=17)
                                        sprintf(temp->data,"%s","自定义");
                                else
                                        strcpy(temp->data,szResourceType[pEntryitem->Name]);
                        }
                        }
                        else
                                sprintf(temp->data,"%d",pEntryitem->Id);
                        PIMAGE_RESOURCE_DIRECTORY dd=(PIMAGE_RESOURCE_DIRECTORY)RSAtoOffset(pEntryitem->OffsetToDirectory);
                        AddNodeToResourceTree(temp,dd,depath+1);
                }

                else
                {
                        PIMAGE_RESOURCE_DATA_ENTRY pDataEntry=(PIMAGE_RESOURCE_DATA_ENTRY)RSAtoOffset(pEntryitem->OffsetToData);
                        sprintf(temp->data,"OffsetToData (RVA): 0x%x,Size: %d,CodePage: %d,Reserved: %d",
                                pDataEntry->OffsetToData,pDataEntry->Size,pDataEntry->CodePage,pDataEntry->Reserved);
                }
                temp=temp->pNextBrother;
        }

}
/*********************************************************************/
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`PE_VIEW.rar`](javascript:void(0);)  
  （233.66kb，36次下载）
