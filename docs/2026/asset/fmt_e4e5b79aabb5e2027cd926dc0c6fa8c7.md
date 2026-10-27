

```text
/* 000078C4: 0027 */    // -- Start Return Code -- //
/* 000078C8: 0023 */    return 0;
end;
```

---

## 第二部分：序列号生成器

为了便于初学者学习，全部使用数组，只需有一点C的基础...:-)

另：`MACINFO.TXT`可执行`TEST_001.EXE`获得

---

**某某研究所**

**版权所有(C) 19xx-2001**

**实时操作系统 - 辅助程序**

**REAL TIME OPERATING SYSTEM (RTOS)**

---

### 文件信息

- 文件名：`KeyGen.C`
- 描述：Lambda Pro v2.0安装序列号生成器
- 创建日期：2001-08-13
- 最后修改日期：2001-08-13
- 编制人：FiNALSErAPH

### 修改记录

| 修改号 | 日期       | 修改人 | 修改描述                           |
|--------|------------|--------|-------------------------------------|
| 初始版本 | 2001-08-13 | HZH    | 这是整理后的改写版添加了注释，比较易懂 |

### 头文件段

```c
#include <stdio.h>
#include <conio.h>
#include <stdlib.h>
#include <process.h>
#include <dos.h>
#include <time.h>

/*-------------------------------------------------------------*/
/*  数据定义段
        */
/*-------------------------------------------------------------*/
FILE *InMacInfo;
FILE *OutSerial;
int iCount = 0;
int iTemp = 0;
char cTable0[36] = {'A','B','C','D','E','F','G','H','I','J','K','L',
            'M','N','O','P','Q','R','S','T','U','V','W','X',
            'Y','Z','0','1','2','3','4','5','6','7','8','9'};
char cTable1[36] = {'Y','W','9','K','V','8','I','U','7','M','T','6',
            'D','S','5','N','Q','4','A','P','3','L','J','2',
            'E','H','1','R','G','0','O','F','Z','C','B','X'};
char cTable2[36] = {'I','R','6','U','Q','5','G','P','4','M','O','3',
            'Y','N','2','S','K','1','A','J','0','T','H','Z',
            'L','F','X','9','E','C','W','8','D','B','V','7'};
char cTarget[36];

/*-------------------------------------------------------------*/
/*  内部函数调用声明段
*/
/*-------------------------------------------------------------*/
void vCopyRightMsg(void);
void vKeyGenerate(void);
void vWaitFor(void);
char cCharToNum(char cInput);
char cNumToChar(char cInput);
int iCheckLocation(char cInput);

/*-------------------------------------------------------------*/
/*  代码段
          */
/*-------------------------------------------------------------*/
void main(void)
{
  vCopyRightMsg();
  vWaitFor();
  vKeyGenerate();
}

void vCopyRightMsg(void)
{
  printf("+----------------------------------------+\n");
  printf("|  LambdaPro v2.0-MAC-Limit-DEMO-0724  |\n");
  printf("|      Install Key Generator v1.0.1
  |\n");
  printf("|        =- Proudly  Present -= 
      |\n");
  printf("|              2001-08-14
            |\n");
  printf("|              FiNAL SErAPH
            |\n");
  printf("+----------------------------------------+\n");
  printf("\n");
}

void vKeyGenerate()
{
//安装序列号生成算法 - 开始
  //从MACINFO.TXT中提取信息
  if((InMacInfo = fopen("MACINFO.TXT","r")) == NULL)
  {
      printf("Can't Open \"MACINFO.TXT\"!\n");
      exit(0);
  }
  char cMacInfo[12];
  fgets(cMacInfo,13,InMacInfo);
  fclose(InMacInfo);
  //字符转为数字
  for(iCount = 0;iCount < 12;iCount++)
  {
      cMacInfo[iCount] = cCharToNum(cMacInfo[iCount]);
  }
  //随机生成长度为52的数字串
  char cOutSerial[52];
  srand(time(NULL));
  iCount = 0;
  while(iCount < 52)
  {
      cOutSerial[iCount] = rand()%16;
      iCount++;
  }
  //将网卡MAC地址插入长度为52的数字串中
  for(iCount = 0;iCount < 12;iCount++)
  {
      cOutSerial[iCount + 2] = cMacInfo[iCount];
  }
  //使长度为52的数字串满足两种校验条件
  //1 - 控制[29]在 2 - 9之间
  iTemp = rand()%10;
  if(iTemp < 2)
  {
      iTemp = 9 - iTemp;
  }
  cOutSerial[29] = iTemp;
  //2
  cOutSerial[31] = 0;
  cOutSerial[32] = 1;
  //调整长度为52的数字串中48位的顺序
  char cTemp[9];
  for(iCount = 0;iCount < 9;iCount++)
  {
      cTemp[iCount]= cOutSerial[iCount+1];
  }
  for(iCount = 1;iCount < 40;iCount++)
  {
      cOutSerial[iCount] = cOutSerial[iCount+9];
  }
  for(iCount = 0;iCount < 9;iCount++)
  {
      cOutSerial[iCount+40] = cTemp[iCount];
  }
  //调整3个校验位
  //1
  iTemp = cOutSerial[1];
  for(iCount = 2;iCount < 17;iCount++)
  {
      iTemp = iTemp ^ cOutSerial[iCount];
  }
  cOutSerial[49] = iTemp;
  //2
  iTemp = cOutSerial[17];
  for(iCount = 18;iCount < 33;iCount++)
  {
      iTemp = iTemp ^ cOutSerial[iCount];
  }
  cOutSerial[50] = iTemp;
  //3
  iTemp = cOutSerial[33];
  for(iCount = 34;iCount < 49;iCount++)
  {
      iTemp = iTemp ^ cOutSerial[iCount];
  }
  cOutSerial[51] = iTemp;
  //将长度为52的数字串转为字符串
  for(iCount = 0;iCount < 52;iCount++)
  {
      cOutSerial[iCount] = cNumToChar(cOutSerial[iCount]);
  }
  //根据字符替换表生成最终序列号，输出结果到屏幕
  if((cOutSerial[0] >= '0') && (cOutSerial[0] <= '4'))
  {
      for(iCount = 0;iCount < 36;iCount++)
      {
        cTarget[iCount] = cTable1[iCount];
      }
  }
  else
  {
      for(iCount = 0;iCount < 36;iCount++)
      {
        cTarget[iCount] = cTable2[iCount];
      }
  }
  printf("Your Install Key:\n\n");
  printf("%c",cOutSerial[0]);
  for(iCount = 1;iCount < 52;iCount++)
  {
      cOutSerial[iCount] = cTarget[iCheckLocation(cOutSerial[iCount])];
      printf("%c",cOutSerial[iCount]);
  }
  //输出结果到文件
  if((OutSerial = fopen("SERIAL.TXT","w+")) == NULL)
  {
      printf("I Can't Create File \"SERIAL.TXT\" !!!\n");
      exit(0);
  }
  for(iCount = 0;iCount < 52;iCount++)
  {
      fputc(cOutSerial[iCount],OutSerial);
  }
  fclose(OutSerial);
//安装序列号生成算法 - 结束
}

void vWaitFor(void)
{
  delay(300);
  printf("Please wait");
  for(int i = 0;i < 6;i++)
  {
      printf(".");
      delay(400);
  }
  printf("\n\n");
}

char cCharToNum(char cInput)
{
  char cOutput;
  if((cInput >= '0') && (cInput <= '9'))
  {
      cOutput = cInput - 48;
  }
  if((cInput >= 'A') && (cInput <= 'F'))
  {
      cOutput = cInput - 55;
  }
  if((cInput >= 'a') && (cInput <= 'f'))
  {
      cOutput = cInput - 87;
  }
  return(cOutput);
}

char cNumToChar(char cInput)
{
  char cOutput;
  if((cInput >= 0) && (cInput <= 9))
  {
      cOutput = cInput + 48;
  }
  if((cInput >= 10) && (cInput <= 15))
  {
      cOutput = cInput + 55;
  }
  return(cOutput);
}

int iCheckLocation(char cInput)
{
  for(int i = 0;i < 36;i++)
  {
      if(cInput == cTable0[i])
      {
        return(i);
      }
  }
  return(-1);
}
```

