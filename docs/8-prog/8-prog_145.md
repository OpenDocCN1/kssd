
# 自动 delete 或 Release

> 原文：[`www.pediy.com/kssd/pediy12/131439.html`](https://www.pediy.com/kssd/pediy12/131439.html)

用了一天时间，终于搞出来这个能自动 delete 或 Release 的函数！

有些该 Release 的指针，如果 delete 就会运行错。这个 bug 不容易排查。

如果能有一个函数，自动根据指针的类型，智能选择使用 delete 还是 Release，

就可以省不少事。

查了相关资料，看了半天 `boost::is_base_of` 代码，却原来是 C++ 支持的 `__is_base_of`。

再用模板写出来，OK！

WinMount 继续招程序员！简历请发 `support@winmount.com`

代码:

```cpp
#pragma once

// 如果一个类指针是从 IUnknown 派生的，则自动用 Release，否则用 delete

namespace boost_bookaa
{

template<typename T, T val> struct bool_
{
  enum { value = val };
};

typedef bool_<bool, true> true_;
typedef bool_<bool, false> false_;


template <typename T> 
void DeleteOrRelease_IMPL(T* p, true_)
{
  p->Release();
}
template <typename T> 
void DeleteOrRelease_IMPL(T* p, false_)
{
  delete p;
}

} // namespace

template <typename T> 
void DeleteOrRelease(T* &p)
{
  if (p != NULL)
  {
    boost_bookaa::bool_<bool, __is_base_of(IUnknown, T)> m;
    boost_bookaa::DeleteOrRelease_IMPL(p, m);
    p = NULL;
  }
}
```

这个代码我前几天看到过一份类似的，也是用模板实现，

作者是个韩国人，在 `codeproject` 上找到的。

代码:



```
// VMemPool.h: interface for the CVMemMan class.
//
// Virtual Memory Pool Management Class
// by bro (bro@shinbiro.com)
// (Cho,Kyung-min)
// 2002-02-19
//
//--------------------------------------------------------------------
// History.
// 2002-02-19 : Fisrt Code.
// 2002-02-20 : vmXXX public Function supported 
//              DEBUG\_NEW supported ( but not impl yet.. )
// 2002-03-08 : New mechanism : alloc bitset, not use std::list is so heavy.
//              just i use 'Free-MemoryBlock-Queue'
//              Support Helper Function: IsBadPtr.
//--------------------------------------------------------------------
// Manual
// Usage
/\*

class CObj : public CVMemPool<>
{
public:
   :
};

CObj* pObj = new CObj; // Virtual Memory Pool 1000 blocks created, allocated 1st block
CObj* pObj2 = new CObj;// allocated second block
delete pObj;           // first block deleted
delete pObj2;          // second block deleted

\*//////////////////////////////////////////////////////////////////////

#if !defined(AFX\_VMEMPOOL\_H\_\_BD52675B\_6C82\_4CDE\_8618\_0141D7A4653F\_\_INCLUDED\_)
#define AFX\_VMEMPOOL\_H\_\_BD52675B\_6C82\_4CDE\_8618\_0141D7A4653F\_\_INCLUDED\_

#if \_MSC\_VER > 1000
#pragma once
#endif // \_MSC\_VER > 1000

#include "BitSet.h"
//====================================================================
// CVMemPool<int T> : Virtual Memory Pool Management Class
//--------------------------------------------------------------------
// Virtual Memory Pool for managing same object block
//====================================================================
template <class objT,DWORD \_dwPoolSizeT = 1000>
class CVMemPool   
{
public:
  //===========================================================
  // Name   : vmInitPool
  // Desc   : Initilize Pool ( you must call this before 'new',
  //          if you want to reconfigure Pool. if you don't call
  //          it, then Pool works with default configuration )
  // Param  : nPoolSize - Size of Pool (Object Block Total count)
  //          bAllocNow - Virtual Allocation right now
  //          nObjSize - bAllocNow is true, then you must give Object size 
  //                     cos, i don't know object's size in this time )
  // Return : if bAllocNow is true, so pool try to alloc, but it fail
  //          then it return false.
  //-----------------------------------------------------------
  // Coder  Dated                    Desc
  // bro    2002-02-20
  //===========================================================
  static inline BOOL vmInitPool( DWORD dwPoolSize , BOOL bAllocNow = FALSE, DWORD dwObjSize = 0 )
  {
    ms\_dwPoolSize = dwPoolSize;
    if( bAllocNow && dwObjSize > 0 )
    {
      ms\_dwObjSize = dwObjSize;
      if( !AllocPool(ms\_dwPoolSize,ms\_dwObjSize) )
      {
        return FALSE;
      }
    }

    return TRUE;
  }
  
  //===========================================================
  // Name   : vmCleanUp
  // Desc   : you should put it on term code
  //          but, it's optional. cos all freed safely when 
  //          process terminate.
  // Param  : 
  // Return : 
  //-----------------------------------------------------------
  // Coder  Dated                    Desc
  // bro    2002-02-20
  //===========================================================
  static inline void vmCleanUp()
  {
    if( ms\_pMemPool )
      VirtualFree( ms\_pMemPool, DWORD(ms\_dwPoolSize\*ms\_dwObjSize), MEM\_RELEASE|MEM\_DECOMMIT );
    DeleteCriticalSection( &ms\_csLock );
  }

  //===========================================================
  // Name   : vmGetPoolInfo
  // Desc   : Get Pool information
  //          If you interest of pool memory usage statistics
  //          then you can use it for that.
  // Param  : pdwPoolSize - \[out\] Pool Object total count( Not bytes )
  //          pdwAllocObjCount - \[out\] Current allocated object count
  //          pdwObjSize - \[out\] one object bytes size.
  // Return : 
  //-----------------------------------------------------------
  // Coder  Dated                    Desc
  // bro    2002-02-20
  //===========================================================
  static inline BOOL vmGetPoolInfo( DWORD\* pdwPoolSize, DWORD\* pdwAllocObjCount, DWORD\* pdwObjSize )
  {
    BOOL bRet = TRUE;
    if( !ms\_pMemPool )
      bRet = FALSE;
    if( pdwPoolSize ) \*pdwPoolSize = ms\_dwPoolSize;
    if( pdwAllocObjCount ) \*pdwAllocObjCount = ms\_dwAllocObjCount;
    if( pdwObjSize ) \*pdwObjSize = ms\_dwObjSize;

    return bRet;
  }
  // memory block validation checker
  static inline BOOL vmIsBadPtr( void\* p )
  {
    // is follow basic rule of pool block?
    if( p && p>=ms\_pData && p<=ms\_pData+(ms\_dwPoolSize\*ms\_dwObjSize)&&
        ((char\*)p-ms\_pData)%ms\_dwObjSize == 0 )
    {
      // is really allocated in pool ?
      DWORD id = Addr2Id(p);
      return (ms\_bsAllocSet.Get(id)==0);
    }
    return TRUE;
  }
  struct CSLock
  {
    CRITICAL\_SECTION  cs;
    CSLock(){ InitializeCriticalSection(&cs); }
    ~CSLock(){ DeleteCriticalSection(&cs); }
    void Lock(){ EnterCriticalSection(&cs); }
    void Unlock() { LeaveCriticalSection(&cs); }
  };
private:
  static CBitSet      ms\_bsAllocSet;      // bitset for allocated blocks
  static DWORD      ms\_dwSrtFree;      // freeblock start marker in circular queue
  static DWORD      ms\_dwEndFree;      // freeblock end marker
  static BOOL        ms\_bEmptyFree;      // state value for empry queue
  static BOOL        ms\_bFullFree;      // state value for full queue
  
  static DWORD      ms\_dwFreeQueueSize;
  static DWORD      ms\_dwTotalSize;


  static CSLock  ms\_csLock;
  static char\*  ms\_pMemPool;           // Virtual memory pool pointer
  static char\*  ms\_pData;            // real data pointer
  static DWORD  ms\_dwPoolSize;           // Total Pool object counter
  static DWORD  ms\_dwObjSize;           // one object size (bytes)
  static DWORD  ms\_dwAllocObjCount;        // current allocated counts
  

  // translate address to block numer
  static inline DWORD  Addr2Id( void\* p )
  {
    return DWORD((char\*)p-ms\_pData)/ms\_dwObjSize;
  }
  // translate block number to address
  static inline void\* Id2Addr( DWORD id )
  {
    return ms\_pData+(ms\_dwObjSize\*id);
  }

  // add free block number to free-circular queue
  static inline BOOL InsertFree( DWORD id )
  {
    if( (ms\_dwEndFree+1) % ms\_dwFreeQueueSize == ms\_dwSrtFree )
    {
      // OverFlow
      return FALSE;
    }
    // write newbie free block
    \*((DWORD\*)ms\_pMemPool+ms\_dwEndFree) = id;
    ms\_dwEndFree = ++ms\_dwEndFree % ms\_dwFreeQueueSize;
    // free block bit set 
    ms\_bsAllocSet.Set( id, 0 );

    return TRUE;
  }
  // pop free block from free-circular queue
  static inline DWORD PopFree()
  {
    if( ms\_dwSrtFree == ms\_dwEndFree )
    {
      // Underflow ( no block for use )
      return 0xFFFFFFFF;
    }

    // get free id
    DWORD id = \*((DWORD\*)ms\_pMemPool+ms\_dwSrtFree);
    // bit set it is used.
    ms\_bsAllocSet.Set( id, 1 );
     
    // increment start free marker
    ms\_dwSrtFree=(++ms\_dwSrtFree)%ms\_dwFreeQueueSize;
    return id;
  }

  static inline BOOL AllocPool(DWORD dwPoolSize, DWORD dwObjSize)
  {
    DWORD dwUsedBytes = ms\_bsAllocSet.sCalcUsedBytes(dwPoolSize);
    DWORD dwFreeQueue = (dwPoolSize+1)\*sizeof(DWORD);
    DWORD dwTotalSize = dwUsedBytes + dwFreeQueue + dwPoolSize\*dwObjSize;

    ms\_pMemPool = (char\*)VirtualAlloc( NULL, dwTotalSize, 
            MEM\_RESERVE|MEM\_COMMIT|MEM\_TOP\_DOWN, 
            PAGE\_READWRITE );
    if( !ms\_pMemPool)
      return FALSE;
    ms\_dwTotalSize = dwTotalSize;
    ms\_dwFreeQueueSize = dwPoolSize+1;
    ms\_pData = ms\_pMemPool+dwUsedBytes+dwFreeQueue;

    // allocation bitset create
    if(!ms\_bsAllocSet.Create( dwPoolSize, TRUE, TRUE, ms\_pMemPool+dwFreeQueue,dwUsedBytes ) )
      return FALSE;
      
    // free circular queue init
    ms\_dwSrtFree = 0;
    ms\_dwEndFree = 0;
    ms\_bEmptyFree = TRUE;
    ms\_bFullFree = FALSE;
    for( DWORD i = 0 ; i < dwPoolSize; i++)
    {
      // all block is available when pool is created.
      InsertFree(i);
    }

    return TRUE;
  }
public:
  // DEBUG\_NEW support function ( No support memory tracking for MFC )
  inline void\* operator new(size\_t nSize, LPCSTR lpszFileName, int nLine)
  {
    return operator new( nSize );
  }
  inline void operator delete(void\* p, LPCSTR lpszFileName, int nLine)
  {
    operator delete( p );
  }

  // new operator override
  void\* operator new( size\_t size )
  {
    ASSERT( size > 0 );
    // pointer for newed free address in pool
    void\* pNew = NULL;
    
    ms\_csLock.Lock();


    //====================================================
    // if pool isn't init, then init
    if( !ms\_pMemPool )
    {
      if( !ms\_dwPoolSize ) ms\_dwPoolSize = \_dwPoolSizeT;
      ms\_dwObjSize = size;
      if( !AllocPool(ms\_dwPoolSize,ms\_dwObjSize) )
      {
        ms\_csLock.Unlock();
        return NULL;
      }
    }
    
    // get available block in pool
    DWORD id = PopFree();
    if( id == 0xFFFFFFFF )
    {
      return NULL;
    }
    pNew = Id2Addr(id);
    
    //====================================================
    if( pNew )
      ms\_dwAllocObjCount++;
    else
      ;//allocation fail!
    ms\_csLock.Unlock();
    return pNew;
  }
  // free memory
  void operator delete( void\* p )
  {
    ASSERT( p && 
        p>=ms\_pData&&
        p<=ms\_pData+(ms\_dwPoolSize\*ms\_dwObjSize)&&
        ((char\*)p-ms\_pData)%ms\_dwObjSize == 0 );
        

    if( !p ) return;
    
    // lock pool
    ms\_csLock.Lock();

    //====================================================
    // add to 'free-queue'for reusing this block 
    InsertFree( Addr2Id(p) );

    ms\_dwAllocObjCount--;
    //====================================================
    
    // unlock
    ms\_csLock.Unlock();
  }
};

// declaration static valuables
template <class T,DWORD F> CBitSet CVMemPool<T,F>::ms\_bsAllocSet;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwSrtFree;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwEndFree;
template <class T,DWORD F> BOOL CVMemPool<T,F>::ms\_bEmptyFree;
template <class T,DWORD F> BOOL CVMemPool<T,F>::ms\_bFullFree;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwTotalSize;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwFreeQueueSize;
template <class T,DWORD F> char\* CVMemPool<T,F>::ms\_pData;

template <class T,DWORD F> CVMemPool<T,F>::CSLock CVMemPool<T,F>::ms\_csLock;
template <class T,DWORD F> char\*  CVMemPool<T,F>::ms\_pMemPool = NULL;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwObjSize;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwAllocObjCount;
template <class T,DWORD F> DWORD CVMemPool<T,F>::ms\_dwPoolSize;


#endif // !defined(AFX\_VMEMPool\_H\_\_BD52675B\_6C82\_4CDE\_8618\_0141D7A4653F\_\_INCLUDED\_)

```
