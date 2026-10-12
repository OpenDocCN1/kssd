
# Design Jacobi programming code.

适合有数论基础者。

> 原文：[`www.pediy.com/kssd/pediy10/99100.html`](https://www.pediy.com/kssd/pediy10/99100.html)

![](img/256611794b5e80e6fd1472d0890c8053.jpg)

代码:

```
Int jacobi(int a, int n)
{
  if (n%2==0) throw "Undefined";
  if (a>=n) a=a%n;// property 2
  int ***=*** (a,b);
  if (a==***)
     return 0;
  else if (***!=0)
       return jacobi(***,n) * jacobi(a/***,n);
       // property 3
  if (a<0) // property 3,4
     if (n%4==1)
        return jacobi(-a,n);
     else
        return -jacobi(-a,n);
     while (a%2==0) //property 3,5
          if (n%8==1||n%8==-1)
             return jacobi(a/2,n);
          else
             return -jacobi(a/2,n);
          if (((a-1)*(m-1)/4)%2==0) // Property 6
             return jacobi(n,a);
          else
             return -jacobi(n,a);
}
```
