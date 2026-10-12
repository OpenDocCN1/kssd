
```
  mov eax,[ecx+34h]
  call eax
  mov eax,ptrClassHandle
  mov ecx,[eax]
  mov edx,ptrDestPort
  push edx
  push szwRecentPort
  push eax
  mov eax,[ecx+30h]
  call eax
  popf
  popad
}
(*ptrDestPort) &= 0xFFFF;
if ((*ptrDestIp) != NULL && (*ptrDestPort) != NULL) return true;

_asm
{
  pushad
  pushf
  mov eax,ptrClassHandle
  mov ecx,[eax]
  mov edx,ptrDestIp
  push edx
  push szdwC2CIP
  push eax
  mov eax,[ecx+34h]
  call eax
  mov eax,ptrClassHandle
  mov ecx,[eax]
  mov edx,ptrDestPort
  push edx
  push szwC2CPort
  push eax
  mov eax,[ecx+30h]
  call eax
  popf
  popad
}
(*ptrDestPort) &= 0xFFFF;
if ((*ptrDestIp) != NULL && (*ptrDestPort) != NULL) return true;

_asm
{
  pushad
  pushf
  mov eax,ptrClassHandle
  mov ecx,[eax]
  mov edx,ptrDestIp
  push edx
  push szdwIP
  push eax
  mov eax,[ecx+34h]
  call eax
  mov eax,ptrClassHandle
  mov ecx,[eax]
  mov edx,ptrDestPort
  push edx
  push szwPort
  push eax
  mov eax,[ecx+30h]
  call eax
  popf
  popad
}
(*ptrDestPort) &= 0xFFFF;
return ((*ptrDestIp) != NULL && (*ptrDestPort) != NULL);
}
```
`ptrClassHandle` 是前面得到的 `returnVal`，IP 地址的 `DWORD` 数据和 `WORD` 的 `port` 信息将保存在后 2 个参数指针的地址中。（代码借鉴了 CoralQQ）

到目前为止已大功告成！其他的细节代码就请各位发挥吧

#### 4.扩展

上面并没有介绍如何得到对方 QQ 版本的信息，其实和获取 IP 一样，你在 `CQQApplication.dll` 中找到 `wProcotol` 的字符串参考，就会发现它和获取 IP 的代码很像，直接抄下来用就是了

CoralQQ 还能在鼠标移动到好友头像上弹出的信息卡片下方显示 IP，其实机制是一样的，这里就不说了，可以看看明日帝国的教程~

好了，我的教程就到此为止，希望能对大家有用~有什么错误希望能立刻指正
