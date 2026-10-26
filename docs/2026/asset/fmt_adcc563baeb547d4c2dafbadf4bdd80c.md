
```c
if ( GetAsyncKeyState ( VK_NUMPAD1 ) &1 )
{
    Chams = !Chams;
}
if(Chams && (Stride == 32 && StartIndex == 0))
{
    DWORD dwOldZEnable = D3DZB_TRUE;
    Device_Interface->SetRenderState(D3DRS_ZENABLE,false); //1
    Device_Interface->SetRenderState(D3DRS_FILLMODE,D3DFILL_SOLID);//2
    Device_Interface->SetRenderState(D3DRS_FILLMODE, D3DFILL_SOLID);
    Device_Interface->GetRenderState(D3DRS_ZENABLE, &dwOldZEnable);
    Device_Interface->SetRenderState(D3DRS_ZENABLE, D3DZB_FALSE);
    Device_Interface->SetPixelShader( Front );
    DrawIndexedPrimitive_Pointer(Device_Interface, Type, BaseIndex, MinIndex, NumVertices, StartIndex, PrimitiveCount);
    Device_Interface->SetRenderState(D3DRS_ZENABLE, dwOldZEnable);
    Device_Interface->SetPixelShader( Back );
}
//-------------结束
return DrawIndexedPrimitive_Pointer(Device_Interface, Type, BaseIndex, MinIndex, NumVertices, StartIndex, PrimitiveCount);
}

//=====================================================================================
DWORD WINAPI VirtualMethodTableRepatchingLoopToCounterExtensionRepatching(LPVOID Param)
{
  UNREFERENCED_PARAMETER(Param);
  while(1)
  {
    Sleep(100);
    *(PDWORD)&Direct3D_VMTable[42] = (DWORD)EndScene_Detour;
    *(PDWORD)&Direct3D_VMTable[82] = (DWORD)DrawIndexedPrimitive_Detour;
    *(PDWORD)&Direct3D_VMTable[16] = (DWORD)Reset_Detour;
  }
  return 1;
}
//=====================================================================================
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
