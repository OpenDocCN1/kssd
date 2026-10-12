
# BCE32 压缩模块分析

## BCE32 压缩函数

* `esi` 中存放着要加压的部分指针
* `edi` 中存放着要写入的缓冲指针
* `ecx` 为要加压的长度
* `ebx`, `edx` 是两个 16 字节的临时缓冲的基地址

**代码:**

```assembly
;Compression routine from BCE32 starts here. This is used only in first gen.
;; BCE32 压缩函数
BCE32\_Compress  Proc
  ;; 保存所有寄存器
  pushad          ;save all regs
;stage 1
  ;; 再保存一次
  pushad          ;and again
  ;; 创建表,这里创建的表的算法是这样的
  ;; 利用 ebx 指向的表(4*4)的一个 DWORD 表
  ;; 读取一个 BYTE,测试它中间每两位为一组
  ;; 是否为 00 01 10 11 如果是则在相应表的
  ;; 位置增 1,形成所有要压缩数据的一个 BIT 记录
create\_table:
  ;; 保存 ecx
  push ecx        ;save for l8r usage
  ;; ecx = 4,这里是为了做运算使用
  push 4
  pop ecx         ;ECX = 4
  ;; 读取一个要压缩的字节到 al
  lodsb           ;load byte to AL
  ;; 保存它
l\_table:push eax        ;save it
  ;; edx = 0
  xor edx, edx        ;EDX = 0
  ;; 测试位 11
  and al, 3         ;this stuff will separate and test
  je st\_end        ;bit groups
  ;; 测试位 10
  cmp al, 2
  je st2
  ;; 测试为 al 时候等于 3
  cmp al, 3
  je st3
st1:  inc edx           ;01
  jmp st\_end
st2:  inc edx           ;10
  inc edx
  jmp st\_end
st3:  mov dl, 3         ;11
st\_end:  inc dword ptr [ebx+4*edx]    ;increment count in table
  pop eax
  ;; 测试下两个 bit
  ror al, 2         ;next bit group
  loop l\_table
  pop ecx           ;restore number of bytes
  loop create\_table      ;next byte
  
  ;; 以下循环测试以上形成的位记录表
  ;; 这个阶段为了使每个计数的位组计数
  ;; 都不一样,相差为 1
  ;; ecx = 4
  push 4          ;this will check for same numbers
  pop ecx         ;ECX = 4
  ;; edx = 0
re\_t:  cdq           ;EDX = 0
  ;; 读取当前位组的计数
t\_loop:  mov eax, [ebx+4*edx]      ;load DWORD
  ;; 自增当前位组的计数
  inc dword ptr [ebx+4*edx]    ;increment it
  ;; 测试当前位组的计数是否与其他组
  ;; 的计数一样.如果一样则当前位组增 1
  cmp eax, [ebx]        ;test for same numbers
  je \_inc\_        ;...
  cmp eax, [ebx+4]      ;...
  je \_inc\_        ;...
  cmp eax, [ebx+8]      ;...
  je \_inc\_        ;...
  cmp eax, [ebx+12]     ;...
  jne ninc\_        ;...
  ;; 当前位组计数增 1
\_inc\_:  inc dword ptr [ebx+4*edx]    ;same, increment it
  ;; 增加循环计数.直到当前位组的计数与
  ;; 其他组的计数不一样为止
  inc ecx           ;increment counter (check it in next turn)
  ;; 这里查看时候超出数组的边界
ninc\_:  cmp dl, 3         ;table overflow ?
  je re\_t          ;yeah, once again
  ;; 移动到下一个位组
  inc edx           ;increment offset to table
  loop t\_loop        ;loop
  ;; 恢复所有寄存器
  popad           ;restore regs

;stage 2
;; 这里进行对位组的泡沫排序,并将排序结构放入第二
;; 个表中
  ;; 保存所有寄存器
  pushad          ;save all regs
  ;; 计数表的基址设置给 esi
  mov esi, ebx        ;get pointer to table
  ;; ebx = 3
  push 3
  pop ebx         ;EBX = 3
  ;; ecx = 3
  mov ecx, ebx        ;ECX = 3
  ;; 这里对计数数组进行泡沫排序,以降序排列
  ;; 由于只有 4 个字节所以循环 3 次
rep\_sort:          ;bubble sort = the biggest value will
            ;always "bubble up", so we know number
            ;steps
  ;; 保存 ecx
  push ecx        ;save it
  
  mov ecx, ebx        ;set pointerz
  ;; edi 指向第二个(4*4)的表
  mov edi, edx        ;...
  ;; 保存 第二个(4*4)的表地址
  push edx        ;save it
  ;; 从位组计数表中读取一个 DWORD
  lodsd           ;load DWORD (count)
  ;; 保存这个 DWORD 到 edx
  mov edx, eax        ;save it
  ;; 读取下一个
sort:  lodsd           ;load next
  ;; 做比较
  cmp eax, edx        ;is it bigger
  ;; 如果后者小于前者,则不交换.
  ;; 这里是按照降序排列
  jb noswap         ;no, store it
  ;; 如果后者大于前者,则交换他们的值
  xchg eax, edx        ;yeah, swap DWORDs
  ;; 重新写入
noswap:  stosd           ;store it
  ;; 下个 DWORD,循环这个过程
  loop sort         ;next DWORD
  ;; 内层循环结束,这里到了外层循环,进行下个 DWORD 的对比
  mov eax, edx        ;biggest in EDX, swap it
  stosd           ;and store
  lea esi, [edi-16]      ;get back pointer
  pop edx           ;restore regs
  pop ecx
  loop rep\_sort        ;and try next DWORD
  popad
  ;; --- 泡沫排序结束 ---

;stage 3
;; 这个步骤用来生成加密密钥
;; 算法为轮询整个数组,找到与之
;; 对应的索引号于密钥的初值相加
;; 然后在循环左移 2 位
;; 最后将密钥存入 ebx 中
  ;; 保存所有的寄存器
  pushad          ;save all regs
  ;; eax = 0
  xor eax, eax        ;EAX = 0
  push eax        ;save it
  ;; ecx = 4,设置循环计数
  push 4
  pop ecx         ;ECX = 4
n\_search:
  ;; 保存第二个表的基址和循环计数
  push edx        ;save regs
  push ecx
  ;; esi 指向未排序的位组计数
  lea esi, [ebx+4*eax]      ;get pointer to table
  push eax        ;store reg
  ;; 读取一个 DWORD
  lodsd           ;load DWORD to EAX
  ;; ecx = 3
  push 3
  pop ecx         ;ECX = 3
  ;; edi = 3
  mov edi, ecx        ;set pointerz
  ;; esi 指向排序过后的计数组
search:  mov esi, edx
  ;; 保存 eax
  push eax        ;save it
  ;; 读取一个 DWORD
  lodsd           ;load next
  ;; 交换排序后的计数组的值到 ebp
  mov ebp, eax
  ;; 恢复 eax 到原先的技术组的值
  pop eax
  ;; ebp 保存为排序后数组的值
  ;; eax 保存为未排序数组的值
  cmp eax, ebp        ;end ?
  je end\_search
  ;; edi 记录索引
  dec edi           ;next search
  ;; 移动到下一个指针
  add edx, 4
  ;; 跳入到下一个循环,继续查找
  loop search
end\_search:
  ;; 记录循环数
  pop eax           ;and next step
  ;; 增加循环计数
  inc eax
  ;; 恢复 ecx, edx
  pop ecx
  pop edx
  ;; esp 指向 eax 的压入的栈顶
  add [esp], edi
  rol byte ptr [esp], 2
  loop n\_search
  ;; 保存到 ebx
  pop [esp.Pushad\_ebx]      ;restore all
  popad           ;...

;stage 4
;; 这个阶段用于加密并写入缓冲
  xor ebp, ebp        ;EBP = 0
  xor edx, edx        ;EDX = 0
  ;; 将密钥保存入第一个缓冲的字节
  mov [edi], bl         ;store decryption key
  ;; 移动指针
  inc edi           ;increment pointer
next\_byte:
  ;; eax = 0
  xor eax, eax        ;EAX = 0
  ;; 保存 ecx 的原值
  push ecx
  ;; 读入一个字节
  lodsb           ;load next byte
  ;; ecx = 4
  push 4
  pop ecx         ;ECX = 4
next\_bits:
  ;; 保存 ecx,eax 寄存器
  push ecx        ;store regs
  push eax
  ;; 测试位,将 al 其余位清 0
  and al, 3         ;separate bit group
  push ebx        ;compare with next group
  ;; 将 bl 其余位清 0
  and bl, 3
  ;; 测试当前字节时候与密钥有同样的位
  cmp al, bl
  ;; 恢复密钥
  pop ebx
  ;; 如果相同则跳入 cb0
  je cb0
  ;; 如果两个字节的测试位不同
  ;; 保存密钥
  push ebx        ;compare with next group
  ;; 密钥循环右移 2 位,测试下一位
  ror bl, 2
  ;; 密钥在与 3 做并操作,清 0 操作保留最后两位
  and bl, 3
  ;; 做对比
  cmp al, bl
  ;; 恢复密钥
  pop ebx
  ;; 相同则跳入 cb1
  je cb1
  ;; 如果不相同,则
  ;; 保存密钥
  push ebx        ;compare with next group
  ;; 移动密钥(将高 4 位与低 4 位对调)的
  ;; 并且将其他位清 0
  ror bl, 4
  and bl, 3
  ;; 与 al 的位做对比
  cmp al, bl
  ;; 恢复密钥
  pop ebx
  ;; 相同则跳入 cb2
  je cb2
  ;; 如果还不相同
  ;; 则保存 0,1 到对应的数组
  push 0          ;store bit 0
  call copy\_bit
  push 1          ;store bit 1
  call copy\_bit
cb0:  push 1          ;store bit 1
end\_cb1:call copy\_bit
  ;; 恢复 eax,ecx 寄存器
  pop eax
  pop ecx
  ;; 移动要加压的数据到下一位组
  ror al, 2
  loop next\_bits        ;next bit
  ;; 恢复 ecx
  pop ecx
  ;; 内循环用于设置一个字节的位
  ;; 外循环用于读取字节
  ;; 进行下一个字节
  loop next\_byte        ;next byte
  ;; 将现有的位置保存,并于起始值做减法运算
  ;; 求出当前的位置索引号
  mov eax, edi        ;save new size
  sub eax, [esp.Pushad\_edi]    ;...
  mov [esp.Pushad\_eax], eax    ;...
  popad           ;restore all regs
  cmp eax, ecx        ;test for negative compression
  jb c\_ok          ;positive compression
  stc           ;clear flag
  ret           ;and quit
c\_ok:  clc           ;negative compression, set flag
  ret           ;and quit
cb1:  push 0          ;store bit 0
end\_cb2:call copy\_bit
  push 0          ;store bit 0
  jmp end\_cb1
cb2:  push 0          ;store bit 0
  call copy\_bit
  push 1          ;store bit 1
  jmp end\_cb2
;; 复制位到内存
copy\_bit:
  mov eax, ebp        ;get byte from EBP
  shl al, 1         ;make space for next bit
  or al, [esp+4]        ;set bit
  jmp cbit
BCE32\_Compress  EndP        ;end of compression procedure
```

## BCE32 解压函数

以下是 `BCE32` 的解压函数，这里不做分析了，算法与压缩函数的反函数。

也比 `BCE32` 压缩函数要简单的多，解压段的第一个字节保存着密钥。

**代码:**

```
// 原文此处未提供具体解压代码，仅说明了算法特性
```
