
# 关于在 DOS 下如何调用汉字库显示汉字

> 原文：[`www.pediy.com/kssd/pediy10/90127.html`](https://www.pediy.com/kssd/pediy10/90127.html)

在 DOS 方式下显示中文，有 2 种办法, 要看分辨率

如果是标准的 DOS 文本方式下，那只有修改 ASCII 的点阵信息, 但是这种方法太麻烦，而且汉字数量有限制,一般情况下不用,用的最多的还是图形方式下调用中文字库显示汉字

我这个程序已经好久了, 是在 `640x480` 的分辨率下直接写屏输出汉字

程序很简单，一般都 DOS 编程经验的人一看就懂

要用到的有 2 个文件,`HZK16` 是 UCDOS 里带的 16 点阵的字库文件,这个随便网上找个 UCDOS 下载就有了

`'C:\\WINDOWS\\STHVCD\\README.TXT'` 是以前金山影霸的 README 文件，这个随便自己换

;〈１〉 汉字的字模与西文下的字模略有不同，但原理相同。
;〈２〉 西文下的字模是将一个字符左右拆分，左半边为一组数据，右半边为一组数据。
;〈３〉 汉字则不同，它将一个汉字上下拆分，上半边为一组数据，下半边为一组数据。
;〈４〉 所以在传输汉字的字模是要一“字”为单位，而西文字符要以“字节”为单位。
;〈５〉 在６４０Ｘ４８０的分辨率下，每行能存放８０个字节的数据，即每行可以显示
;       ４０个１６点阵的汉字。
;〈６〉 要注意：在写屏时，每写完２个字节的数据，要换行显示。（跳过７６个字节）
;〈７〉 １６Ｘ１６点阵是按行存储的，而２４Ｘ２４点阵是按列存储的。
;〈８〉 汉字的机内码就是它的ＡＳＣＩＩ码，将低位和高位各减去１６１，就是区位;码了，低位是位码，高位是区码。
;〈９〉 有了区位码，就可根据公式“（区码Ｘ９４＋位码）Ｘ３２ ”来得出该汉字
;       字模在字库中的位置。
;〈Ａ〉 同样，利用上面的公式也可以求出汉字在其它点阵字库（如２４Ｘ２４点阵）
;       位置，只要将上式的３２换成７２即可。
;〈Ｂ〉 汉字换行，要将ＶＩＤＥＯ  ＲＡＭ中的指针增加８０Ｘ１６Ｘ行数。
;       公式：８０Ｘ１６Ｘ行数＋列数（一个汉字有２列）

```
stack segment stack 'stack'
      db 1024 dup (0)
stack ends
```

```
data segment
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  DATA SEGMENT  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
ver_msg db 50*1024 dup (0)
err_msg db 0dh,0ah,' Open Or Read File " HZK16 " Error !'
        db 0dh,0ah,' Path: C:\\UCDOS\\HZK16 ',0dh,0ah,07,'$'
hzk_file db 'C:\\UCDOS\\HZK16',0
read_file db 'C:\\WINDOWS\\STHVCD\\README.TXT',0
file_byte dw ?
msg_ip dw 0
hz_q dw ?
hz_w dw ?
hzk_ip dw ?
       dw ?
file_handle dw ?
hzk_buf db 32 dup(0)
crt_emm db ?
ram_add dw 0
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  DATA SEGMENT  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
data ends
```

```
cseg segment
     assume cs:cseg,ds:data,es:data,ss:stack
begin:       mov ax,data
             mov ds,ax
             mov ax,stack
             mov ss,ax
             mov ax,3d00h
             mov dx,offset read_file
             int 21h
             jc read_err
             mov word ptr [file_handle],ax
             mov ax,4202h
             mov bx,[file_handle]
             xor cx,cx
             xor dx,dx
             int 21h
             jc open_err
             mov word ptr [file_byte],ax
             mov ax,4200h
             mov bx,[file_handle]
             xor cx,cx
             xor dx,dx
             int 21h
             jc open_err
             mov ah,3fh
             mov bx,[file_handle]
             mov cx,[file_byte]
             mov dx,offset ver_msg
             int 21h
             jc open_err
             mov ax,3e00h
             mov bx,[file_handle]
             int 21h
             jc open_err
             mov cx,[file_byte]
             mov dx,offset ver_msg
             add cx,dx
             inc cx
             mov si,cx
             mov byte ptr ds:[si],1ah
             mov ax,0012h
             int 10h
             mov ax,0f00h
             int 10h
             mov byte ptr [crt_emm],bh
             mov ah,02h
             xor dx,dx
             int 10h
           mov ax,3d00h
           mov dx,offset hzk_file
           int 21h
           jc open_err
           mov word ptr [file_handle],ax
load_msg:    call wait_user
             mov si,offset ver_msg
             add si,[msg_ip]
             mov al,0a1h
             cmp byte ptr [si],al
             jc dis_char
             add word ptr [msg_ip],2
             call set_ram
             call asc2qwm
             call hz2buf
             mov ax,0a000h
             mov es,ax
             mov si,offset hzk_buf
             mov di,[ram_add]
             call display_hz
             call set_gb
             jmp load_msg
quit:        mov ah,0
             int 16h
             mov ah,3eh
             mov bx,[file_handle]
             int 21h
             mov ax,0003h
             int 10h
open_err:    mov ax,4c00h
             int 21h
dis_char:    cmp byte ptr [si],1ah
             jz quit
             inc word ptr [msg_ip]
             mov ah,02h
             mov byte ptr dl,[si]
             int 21h
             jmp load_msg
```

```
display_hz  proc near
             sti
             pushf
             push cx
             push si
             push di
             mov cx,16
display:     movsw
             add di,4eh
             loop display
             pop di
             pop si
             pop cx
             popf
             ret
display_hz  endp
```

```
asc2qwm     proc near
            push ax
            push bx
            push cx
            push dx
            push si
          ;  mov si,offset message
            mov word ptr ax,[si]
            xchg ah,al                 ; 将区码和位码交换,AH 是区码,AL 是位码
            sub ax,0a1a1h              ; 将机内码转换成区位码
          ;  and ax,7f7fh
          ;  sub ax,2121h
            mov byte ptr [hz_q],ah
            mov byte ptr [hz_w],al
            xor ax,ax
            mov ax,[hz_q]
            mov dx,94
            mul dx
            push dx
            push ax
            xor ax,ax
            xor dx,dx
            mov ax,[hz_w]
            pop bx
            adc ax,bx
;            cmp dx,0
;            jz no_add
            mov dx,32
            mul dx
            pop cx                       ;pop high, cx
            mov word ptr [hzk_ip],ax
            add dx,cx
            mov word ptr [hzk_ip+2],dx
            pop si
            pop dx
            pop cx
            pop bx
            pop ax
            ret
asc2qwm     endp
```

```
hz2buf     proc near
           push ax
           push bx
           push cx
           push dx
;           mov ax,3d00h
;           mov dx,offset hzk_file
;           int 21h
;           jc error
;           mov word ptr [file_handle],ax
           mov ax,4200h
           mov bx,[file_handle]
           mov cx,[hzk_ip+2]
           mov dx,[hzk_ip]
           int 21h
           jc read_err
           mov ah,3fh
           mov bx,[file_handle]
           mov cx,32
           mov dx,offset hzk_buf
           int 21h
           jc read_err
;           mov ah,3eh
;           mov bx,[file_handle]
;           int 21h
           pop dx
           pop cx
           pop bx
           pop ax
           ret
read_err:  mov ah,3eh
           mov bx,[file_handle]
           int 21h
error:     mov ah,09
           mov dx,offset err_msg
           int 21h
           mov ax,4c01h
           int 21h
hz2buf     endp
```

```
set_gb     proc near
           push ax
           push bx
           push cx
           push dx
           mov ah,03h
           mov bh,[crt_emm]
           int 10h
           add dl,2
           cmp dl,80
           jc no_set
           inc dh
           xor dl,dl
no_set:    mov ah,02h
           int 10h
           pop dx
           pop cx
           pop bx
           pop ax
           ret
set_gb     endp
```

```
set_ram    proc near
           push ax
           push bx
           push cx
           push dx
           mov ah,03h
           mov bh,[crt_emm]
           int 10h
           mov cx,dx
           mov bx,80*16
           xor ax,ax
           mov al,ch
           mul bx
           push ax          ; X
           xor ax,ax
```



```
mov al,cl

pop bx

add ax,bx

mov word ptr [ram_add],ax

pop dx

pop cx

pop bx

pop ax

ret

set_ram    endp

wait_user  proc near

push ax

push bx

push cx

push dx

mov ah,03h

mov bh,[crt_emm]

int 10h

cmp dh,29

jnz no_change

xor ah,ah

int 16h

mov ax,0600h

mov bh,70h

mov cx,0

mov dx,2c4fh

int 10h

mov ah,02h

mov bh,[crt_emm]

xor dx,dx

int 10h

no_change: pop dx

pop cx

pop bx

pop ax

ret

wait_user  endp

cseg ends

     end begin
```
