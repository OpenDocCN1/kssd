
# 巧妙使用汇编,将当前 `this` 转化为主窗口 `this` 指针

> 原文：[`www.pediy.com/kssd/pediy11/122284.html`](https://www.pediy.com/kssd/pediy11/122284.html)

获取主窗口的指针 大家都知道,这样只是为了方便,好看,代码看着规范

代码:

```
CNewUserDlg *UserClass = (CNewUserDlg *)AfxGetMainWnd();    
__asm    
{    
    lea eax,this    
    mov ebx,UserClass     
    mov [eax],ebx     
}     
ASSERT(this->m_hWnd);     
int dwCount = m_UserList.GetItemCount();             //没用用本身的 this     
UserClass->m_UserList.InsertItem(dwCount,UserID,0); //用了 UserClass     
return 0; 

```
