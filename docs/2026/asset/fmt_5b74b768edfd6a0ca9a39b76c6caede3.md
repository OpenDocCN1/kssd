
0038F9E5    8B45 D4          mov eax,dword ptr ss:[ebp-2C]

0038F9E8    3B45 D8          cmp eax,dword ptr ss:[ebp-28]

0038F9EB    75 02            jnz short 0038F9EF

0038F9ED  ^ EB E7            jmp short 0038F9D6

0038F9EF    8B4D D4          mov ecx,dword ptr ss:[ebp-2C]

0038F9F2    8B548D E4        mov edx,dword ptr ss:[ebp+ecx\*4-1C]

0038F9F6    83EA 02          sub edx,2

0038F9F9    3955 F8          cmp dword ptr ss:[ebp-8],edx

0038F9FC    7C 16            jl short 0038FA14

0038F9FE    8B45 D4          mov eax,dword ptr ss:[ebp-2C]

0038FA01    8B4C85 E4        mov ecx,dword ptr ss:[ebp+eax\*4-1C]

0038FA05    83C1 02          add ecx,2

0038FA08    394D F8          cmp dword ptr ss:[ebp-8],ecx

0038FA0B    7F 07            jg short 0038FA14//这边必须跳

0038FA0D    33C0             xor eax,eax

0038FA0F    E9 E3000000      jmp 0038FAF7

0038FA14  ^ EB C0            jmp short 0038F9D6

0038FA16  ^ EB 9C            jmp short 0038F9B4

仔细看这段代码会发现它要求r[i]的每个元素这件必须绝对值〉2

0038FA18    8B55 08          mov edx,dword ptr ss:[ebp+8]

0038FA1B    33C0             xor eax,eax

0038FA1D    8A42 08          mov al,byte ptr ds:[edx+8]

0038FA20    83F8 01          cmp eax,1//------------------------------keys的第8个值=1

0038FA23    74 07            je short 0038FA2C

0038FA25    33C0             xor eax,eax

0038FA27    E9 CB000000      jmp 0038FAF7

0038FA2C    C745 C8 0F2FC401 mov dword ptr ss:[ebp-38],1C42F0F

0038FA33    C745 D0 A0D9D201 mov dword ptr ss:[ebp-30],1D2D9A0

0038FA3A    8B4D 08          mov ecx,dword ptr ss:[ebp+8]

0038FA3D    8B51 04          mov edx,dword ptr ds:[ecx+4]

0038FA40    8955 CC          mov dword ptr ss:[ebp-34],edx

0038FA43    8B45 CC          mov eax,dword ptr ss:[ebp-34]

0038FA46    3B45 C8          cmp eax,dword ptr ss:[ebp-38]

0038FA49    72 08            jb short 0038FA53

0038FA4B    8B4D CC          mov ecx,dword ptr ss:[ebp-34]

0038FA4E    3B4D D0          cmp ecx,dword ptr ss:[ebp-30]

0038FA51    76 07            jbe short 0038FA5A

从keys第4个起所构成的int必须在0x1C42F0F和0x1D2D9A0之间

0038FA53    33C0             xor eax,eax

0038FA55    E9 9D000000      jmp 0038FAF7

0038FA5A    8B55 10          mov edx,dword ptr ss:[ebp+10]

0038FA5D    833A 00          cmp dword ptr ds:[edx],0--------------------比较直到固定字符串为0

0038FA60    0F84 88000000    je 0038FAEE----------------------------------跳向成功

0038FA66    8B45 10          mov eax,dword ptr ss:[ebp+10]

0038FA69    8B4D CC          mov ecx,dword ptr ss:[ebp-34]

0038FA6C    3B08             cmp ecx,dword ptr ds:[eax]

0038FA6E    75 70            jnz short 0038FAE0--------------------------不等于则跳，\*1

不跳则还要分析下面一大段

0038FA70    8B55 08          mov edx,dword ptr ss:[ebp+8]

0038FA73    33C0             xor eax,eax

0038FA75    8A42 0A          mov al,byte ptr ds:[edx+A]

0038FA78    8B4D 10          mov ecx,dword ptr ss:[ebp+10]

0038FA7B    33D2             xor edx,edx

0038FA7D    8A51 04          mov dl,byte ptr ds:[ecx+4]

0038FA80    3BC2             cmp eax,edx

0038FA82    75 5C            jnz short 0038FAE0

0038FA84    C745 F8 0A000000 mov dword ptr ss:[ebp-8],0A

0038FA8B    C745 D4 00000000 mov dword ptr ss:[ebp-2C],0

0038FA92    EB 09            jmp short 0038FA9D

0038FA94    8B45 D4          mov eax,dword ptr ss:[ebp-2C]

0038FA97    83C0 01          add eax,1

0038FA9A    8945 D4          mov dword ptr ss:[ebp-2C],eax

0038FA9D    837D D4 05       cmp dword ptr ss:[ebp-2C],5

0038FAA1    7D 33            jge short 0038FAD6

0038FAA3    8B4D 08          mov ecx,dword ptr ss:[ebp+8]

0038FAA6    034D F8          add ecx,dword ptr ss:[ebp-8]

0038FAA9    33D2             xor edx,edx

0038FAAB    8A11             mov dl,byte ptr ds:[ecx]

0038FAAD    8955 F8          mov dword ptr ss:[ebp-8],edx

0038FAB0    8B45 08          mov eax,dword ptr ss:[ebp+8]

0038FAB3    0345 F8          add eax,dword ptr ss:[ebp-8]

0038FAB6    33C9             xor ecx,ecx

0038FAB8    8A08             mov cl,byte ptr ds:[eax]

0038FABA    8B55 10          mov edx,dword ptr ss:[ebp+10]

0038FABD    0355 D4          add edx,dword ptr ss:[ebp-2C]

0038FAC0    33C0             xor eax,eax

0038FAC2    8A42 05          mov al,byte ptr ds:[edx+5]

0038FAC5    3BC8             cmp ecx,eax

0038FAC7    74 02            je short 0038FACB

0038FAC9    EB 0B            jmp short 0038FAD6

0038FACB    8B4D F8          mov ecx,dword ptr ss:[ebp-8]

0038FACE    83C1 02          add ecx,2

0038FAD1    894D F8          mov dword ptr ss:[ebp-8],ecx

0038FAD4  ^ EB BE            jmp short 0038FA94

0038FAD6    837D D4 05       cmp dword ptr ss:[ebp-2C],5

0038FADA    75 04            jnz short 0038FAE0

0038FADC    33C0             xor eax,eax

0038FADE    EB 17            jmp short 0038FAF7

从\*1处跳到这边

0038FAE0    8B55 10          mov edx,dword ptr ss:[ebp+10]

0038FAE3    83C2 0C          add edx,0C-----------------------加0x0C后就是0了（故上面让他跳好了）

0038FAE6    8955 10          mov dword ptr ss:[ebp+10],edx

0038FAE9  ^ E9 6CFFFFFF      jmp 0038FA5A

0038FAEE    B8 01000000      mov eax,1

0038FAF3    EB 02            jmp short 0038FAF7

0038FAF5    33C0             xor eax,eax

0038FAF7    8BE5             mov esp,ebp

0038FAF9    5D               pop ebp

0038FAFA    C2 0C00          retn 0C

由此我们可以得出注册机算法

key符合下列条件

1.第8位=1

2.从第4位开始的4个char组成一个int在0x1C42F0F和0x1D2D9A0之间并且不等于给定字符串的前4个char组成的int（0x98989898）

3.从0A开始取出值p[a]，再以该值为偏移量得到零一个值p[b]，p[b]^m[i] ==p[b+1],a = b+2,继续进入循环

4.上面的每个p[a]的值之间的距离大于2，并且在0xA-0x62之间

下面给出注册机的代码：在这个代码中我只算出了一组key。当然你可以改一下算出所有key

```c
#include <stdio.h>

void main()

{

  char p[100];

  char q[201] = {0};

  char t[20];

  int a = 0x0a;//a的初始值

  char b = 0x1f;//我把它从0x1f开始，0x1f在0xA-0x62之间

  char m[] = "\xA5\x5D\x12\xE4\x37";\\这时程序初始化的值

  for(int i = 0; i < 100; i ++)

    p[i] = (char)0x88;//没事就用88来填

  for (i = 0; i < 5; i ++)

  {

    p[a] = b;

    p[b + 1] = p[b] ^ m[i];//上面的第三的条件

    a = b + 2;

    b = b + 8;//第四个条件，我让他们相差8〉2

  }

  p[4] = 0x11;

  p[5] = 0x11;

  p[6] = 0x0D1;

  p[7] = 0x01;//这4个值组成0x1d11111在0x1C42F0F和0x1D2D9A0

  p[8] = 1;//这个值必须为1

  //下面是hentostr转化，方法很土，其实可以仿照他的程序中strtohen

  for(i = 0; i < 100; i ++)

  {

    sprintf(t,"%02X",p[i]);

    if((unsigned int)p[i] >= 0x80)

    {

      q[2 \* i] = t[6];

      q[2 \* i + 1] = t[7];

    }

    else

    {

      q[2 \* i] = t[0];

      q[2 \* i + 1] = t[1];

    }

  }

  //存放在文件中

  FILE \*fp = fopen("C:\\key.dat","w");

  if(NULL == fp) return ;

  fprintf(fp,"%s",q);

  fclose(fp);

}
```

至此，分析完毕！
