
```
WcharToAchar(ch_ModuleName,ch_modulenames);  

for(unsigned int i=0;i<ModuleTion->Count;i++)  
{  
  if(strstr(ModuleTion->Module[i].ImageName,ch_modulenames))  
  {  
    if(b_open==FALSE)  
    {  
      data=(ULONG)ModuleTion->Module[0].Base;  
    }  
    else  
    {  
      data=(ULONG)ModuleTion->Module[0].Size;  
    }  
    break;  
  }  
}  
delete [] lpBuffer;  
return data;  
}
```
