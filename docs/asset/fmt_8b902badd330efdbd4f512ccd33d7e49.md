
`Lable11:`
`inc ecx`
`cmp ecx, esi`
`jl again5`
`mov dword ptr ds:[dValue2], eax`
`fild qword ptr ds:[dValue2]`
`fsin`
`fld st`
`fmul qword ptr ss:[nValue3]`
`wait`
`fstcw word ptr ds:[dSaveFcw]`
`wait`
`mov ax, word ptr ds:[dSaveFcw]`
`or ah, 0xc`
`mov word ptr ds:[dSaveFcw + 8], ax`
`fldcw word ptr ds:[dSaveFcw + 8]`
`fistp qword ptr ss:[dValue4]`
`fldcw word ptr ss:[dSaveFcw]`
`mov eax, dword ptr ss:[dValue4]`
`mov ecx, eax`
`test ecx, ecx`
`jl Lable12`
`fmul qword ptr ss:[dTemp7]`
`wait`
`fstcw word ptr ds:[dSaveFcw]`
`wait`
`mov ax, word ptr ds:[dSaveFcw]`
`or ah, 0xc`
`mov word ptr ds:[dSaveFcw + 8], ax`
`fldcw word ptr ds:[dSaveFcw + 8]`
`fistp qword ptr ss:[dValue4]`
`fldcw word ptr ss:[dSaveFcw]`
`mov eax, dword ptr ss:[dValue4]`
`mov ecx, 0xcc7`
`sub ecx, eax`
`mov eax, ecx`
`jmp Lable13`
`Lable12:`
`mov eax, 0xcc7`
`fstp st`
`sub eax, ecx`
`Lable13:`
`cdq`
`mov ecx, 5`
`idiv ecx`
`mov dword ptr ds:[number], edx`
`popad`

`}`
`return chChart[number];`

`}`

```c
int main(int argc, char* argv[])
{
  int i = 0;
  char chPart2Key[8] = {0};
  char chRandNumber[8] = {0};
  srand( (unsigned)time( NULL ) );
  for (i = 0; i < 8; i++)
  {
    int j = rand() % 3 + 1;
    switch(j)
    {
      case 1:
        {
          chRandNumber[i] = (rand() % 9) + 0x30;
          break;
        }
      case 2:
        {
          chRandNumber[i] = (rand() % 26) + 0x41;
          break;
        }
      case 3:
        {
          chRandNumber[i] = (rand() % 26) + 0x61;
          break;
        }
      default:
        {
          printf("damn it! please try again!  :(\n");
          return -1;
        }
    }
  }
  for (i = 0; i < 8; i++)
  {
    chPart2Key[i] = GeneratePart2Key(chRandNumber[i], i);
  }
  printf("enjoy key: %s\n", chRandNumber/*chTest, chPart2Key*/);
  return 0;
}
```

就在我写完这篇分析的时候，最新版 `3.95` released，经过测试，算法没有变，仍然可以使用。

任何问题可至：`Cracker_prince@163.com`

菜鸟写菜文，请大侠们不吝赐教~

prince 2005.07.16

`4dd466d0e1e0cc123afbe504da2462ca`
