
# 我写的俄罗斯方块，基本完成，源码已经附上

> 原文：[`www.pediy.com/kssd/pediy11/115344.html`](https://www.pediy.com/kssd/pediy11/115344.html)

`sdk`+`api`，源码里有少量注释，代码风格和变量名都很不堪![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")  

感谢各位帮助测试，谢谢 `pcasa` 指教，目前尚未发现问题，欢迎继续测试  

![](img/738c2a646b223f67548135a5198123de.jpg)

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `俄罗斯方块.rar`  
|  
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `rsc.rar`  
|

定义两个变量 `hbrushold` `hpenold`  

代码:

```
    case WM_PAINT:
      hdc = BeginPaint(hWnd, &ps);
      hbrush=CreateSolidBrush(RGB(240,240,240));//矩形背景色
      hpen=CreatePen(PS_SOLID,2,RGB(50,50,50));//矩形边框色
      hbrushold = SelectObject(hdc,hbrush);//画刷
      hpenold = SelectObject(hdc,hpen);//画笔
      Rectangle(hdc,1,1,300,540);//绘制游戏区域矩形
      Rectangle(hdc,338,40,458,160);//绘制"下一个"矩形
      DeleteObject(SelectObject(hdc,hbrushold));
      hbrush=CreateSolidBrush(RGB(226,121,7));//矩形背景色
      SelectObject(hdc,hbrush);//画刷
      if(gamestatus!=0)
      {
        for(i=0;i<10;i++)
          for(j=0;j<18;j++)
            if(mainmap[i][j]==1) Rectangle(hdc,i*30,j*30,i*30+30,j*30+30);//绘制主棋盘数据
        for(i=0;i<4;i++)
          for(j=0;j<4;j++)
            if(cube[next][0][i][j]==1) Rectangle(hdc,338+i*30,40+j*30,338+i*30+30,40+j*30+30);//绘制"下一个"
      }
      DeleteObject(SelectObject(hdc,hbrushold));
      DeleteObject(SelectObject(hdc,hpenold));
      EndPaint(hWnd, &ps);
      break;
```
