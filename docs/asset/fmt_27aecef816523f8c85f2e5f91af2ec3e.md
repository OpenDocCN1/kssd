
```
chKey[0] -= nNameRemainder;

} while(chKey[0] > 122);

}

nTemp1 = chKey[5] & chKey[6];

nTemp1 *= chKey[7];

chKey[4] = (nNameRemainder * 100 + 8 - nTemp1) % nNameRemainder;

if (chKey[4] < 33)

{

  do

  {

    chKey[4] += nNameRemainder;

  } while(chKey[4] < 33);

}

else if (chKey[4] > 122)

{

  do

  {

    chKey[4] -= nNameRemainder;

  } while(chKey[4] > 122);

}

printf("Woooo~ :) Enjoy your private key: \n%s\n", chKey);

return 0;

}

void FillKeyRandom(char *chKey)

{

  //
  // Sets a random starting point
  //

srand((unsigned)time(NULL));

for (int i = 0; i < 40; i++)

{

  chKey[i] = rand() % 89 + 33  ;

}

return;

}
```

注册信息保存在：`HKEY_CURRENT_USER\Software\Magic C Enterrise Edition\User` 下面的 `Reg Code` 键里面，删除可重新注册。

菜鸟写菜文，大侠见笑了~

`prince` 2006.01.04

`4dd466d0e1e0cc123afbe504da2462ca`
