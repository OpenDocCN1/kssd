
# MFC 框架学习笔记本 (1) 类工厂

> 原文：[`www.pediy.com/kssd/pediy10/92194.html`](https://www.pediy.com/kssd/pediy10/92194.html)

今天开始学习 MFC 框架原理，首先了解一下 MFC 框架类的派生机制，这其中老师提到了一个“类工厂”的概念，工厂一个零件的产生流程大致如下：

原材料 > 符合某种特性的材料（热处理改变物理性质） > 粗毛坯（粗步加工物理形状） > 细坯 (精加工物理形状) > 成品（经过一些表面特殊处理）

在此原材料是所有零件的基础，如同 MFC 中的 `CObject`，其加工出来半成品都是属于由基类派生出来的（派生类），相对于上一道工序来说是成品，相对于下一道工序来说是原料，因此在一个加工环节需要指定它的原料与目标，下面来看一下 `CMyApp` 的产生流程:

`CObject -> CCmdTarget -> CWinThread -> CWinApp -> CMyApp`

`CObject` 是所有派生类的基类，指定了其重要的公有属性：

*   静态库：用一个 `CRuntimeClass* m_pBaseClass` 来保存其父类的指针
*   动态 DLL: 用一个 `CRuntimeClass* (PASCAL* m_pfnGetBaseClass)()` 获取父类指针

这个指针被保存 `static const AFX_DATA CRuntimeClass class##class_name` 的结构中，此结构如下：

代码:

```
struct CRuntimeClass
{
  LPCSTR m_lpszClassName;   
  int m_nObjectSize;               
  UINT m_wSchema; // schema number of the loaded class
  CObject* (PASCAL* m_pfnCreateObject)(); // NULL => abstract class
#ifdef _AFXDLL
  CRuntimeClass* (PASCAL* m_pfnGetBaseClass)();
#else
  CRuntimeClass* m_pBaseClass;
#endif
};
// Operations
  CObject* CreateObject();
  BOOL IsDerivedFrom(const CRuntimeClass* pBaseClass) const;

// Implementation
  void Store(CArchive& ar) const;
  static CRuntimeClass* PASCAL Load(CArchive& ar, UINT* pwSchemaNum);

  // CRuntimeClass objects linked together in simple list
  CRuntimeClass* m_pNextClass;       // linked list of registered classes

```

了解了派生关系后，我们再看看这些零件是如何被组装起来的，从工厂的角度来说，需要组装完成一个产品，首先需要组装图纸，但是一般图纸是根据成熟的实验室产品来制定的，所以实验室出来的产品是没有图纸，而且由实验过程制造出来的。。我们在写程序代码的过程就是制造这个产品的实验过程，不同的添加资源、建立新类型的实例等等都是处于实验步骤，一旦按下 F5 就开始试车了。

当然很多时候存在 BUG，这时就需要微调了。。。经过 N 次反复操作以后程序终于运行了，产品出来了，这时是没有图纸的，好在实验室有一个好的环境啊，给你忠实的记录下来了每一步的步骤并且给你进行优化（开发环境的编译及优化过程），事实上我们写的 MFC 框架程序都是由框架帮我们记录并管理的，所以我们实在太幸福了，由于运行期的初始化过程太多内容，暂时先不谈这个课程。

先记住有一个叫 `m_classList` 的链表记录下了我们类的指针，在 `CRuntimeClass` 结构中有一个指针反映了这个过程（`CRuntimeClass* m_pNextClass`)......

综上所述，基于 MFC `CObject` 派生的类都有一个叫 `CRuntimeClass` 的静态结构记录下我们建立的对象信息，因此在其派生类都有类似的过程，这一过程 MFC 框架会帮我们自动生成:

*   声明宏：`DECLARE_DYNAMIC(class_name)`  
*   实现宏：`IMPLEMENT_DYNAMIC(class_name)`  

为什么框架会做得如此复杂，其目的为自动化组装过程中的自动识别，定位，这些内容将在后续的课程中得到解释。

文笔粗糙，仅以此加深记忆，望各位同学别笑话我
