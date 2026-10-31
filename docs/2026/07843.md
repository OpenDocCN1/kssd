
# [原创]在学PE格式时自己写的显示文件HEX数据的小程序

程序在2楼附件

``` 
#include <cstdlib>

#include <iostream>

#include <windows.h>

//#include <winnt.h>



using namespace std;

bool bQuit;

FILE * fp;

void ShowHelpInfo();

void GetCommand(char * commands);

void CheckCommand(char * commands);

bool GetOpenFileHandle(char * path);

bool ShowFileHexData(int lowrange,int highrange);

void show(unsigned char * showdata,int cha,long lowrange);

int main(int argc, char *argv[])

{



    char compt[10];

    char command[100];

    fp=0;

    //sprintf("-",compt);

    bQuit=1;

    ShowHelpInfo();

    // show();system("PAUSE");

    while (bQuit)

    {

        printf("-");

        scanf("%c",command);

        CheckCommand(command);

    }

    //  system("PAUSE");

    return EXIT_SUCCESS;

}

void ShowHelpInfo()

{

    char showstr[]="File Hex for Dos Version write by Chjshen.\n                              2009-05-21\n\n";

    printf(showstr);

    printf("Open  -f [filename]  |open Filename file.\n");

    printf("Show  -s [range]     |show hex data. \n");

    printf("Quit  -q             |quit.\n");

    printf("Hex   -h  Hex1 hex2  |Hex1+Hex2 and Hex1-Hex2 or Hex2-Hex1.\n");

    printf("Help  -?             |help information.\n");

    printf("\nFor Example:\n -f notepad.exe \n -s 100 15f \n -h 10 12 \n");



}

void CheckCommand(char * commands)

{

    char command;

    char hexresultstr[100];

    char path[MAX_PATH];

    int hex1,hex2;

    int lowrange,highrange;

    command=commands[0];



    switch (toupper(command))

    {

    case 'F':

        scanf("%s",path);

        if (GetOpenFileHandle(path))

            printf("Now \"%s\" is opened.",path);

        fflush(stdin);

        break;

    case 'E':

    case 'Q':

        if (fp!=0)

            fclose(fp);

        printf("quit.");

        bQuit=0;

        fflush(stdin);

        break;

    case 'S':

        scanf("%x %x",&lowrange,&highrange);

        ShowFileHexData(lowrange,highrange);

        //printf("Show hex data.");

        fflush(stdin);

        break;

    case 'H':

        scanf("%X %X",&hex1,&hex2);

        sprintf(hexresultstr,"%04X %04X",hex1+hex2,hex1>=hex2?hex1-hex2:hex2-hex1);

        printf(hexresultstr);

        fflush(stdin);

        break;

    case '?':

        ShowHelpInfo();

        fflush(stdin);

        break;

    case 10:

        fflush(stdin);

        break;

    default:

        fflush(stdin);

        printf("Error command line.");

    }

    printf("\n");

}

bool GetOpenFileHandle(char * path)

{

    if (strlen(path)==0)

    {

        scanf("%s",path);

    }

    if (fp!=0)

    {

        fclose(fp);

    }

    fp=fopen(path,"rb");



    if (fp==0)

    {

        printf("Open File Error,please check your enter's filename is vaild.");

    }

    else

    {

        return true;

    }



    return 0;

}

bool ShowFileHexData(int lowrange,int highrange)

{



    long temlength=0;

    if (fp==0)

    {

        printf("No file opened,please open file that you wanted to show hex data.");

        return 0;

    }

    long length;

    fseek(fp,0,SEEK_END);

    length=ftell(fp);

    if (highrange>length||lowrange>length)

    {

        printf("over file length.");

        return 0;

    }

    if (lowrange>=highrange)

    {

        printf("Range error,please enter correct hex number.");

        return 0;

    }

    printf("%X~%X  Hex Data:\n",lowrange,highrange);

    // printf("%02X",readdata);

    int totall=lowrange;

    int i;

    int cha=highrange-lowrange;

    i=0;



    unsigned char * temdata=(unsigned char*)malloc(cha+1);

    fseek(fp,lowrange,SEEK_SET);

    fread(temdata,sizeof(unsigned char),cha+1,fp);

    cha++;

    show(temdata,cha,lowrange);



    free(temdata);

    return 1;

}

void show(unsigned char * showdata,int cha,long lowrange)

{

    int j,i;

    i=0;

    char readdata[100]="";

    j=0;

    char asii[20]="";

    while (j<=(int)cha/16)

    {

        i=0;

        sprintf(readdata,"");

        sprintf(asii,"");

        printf("\n%08X:",lowrange+j*16);

        do

        {

            if (j*16+i<cha)

            {

                if (i%16==8)

                    printf("-");

                else

                    printf(" ");

                printf("%02X",showdata[j*16+i]);

                if (showdata[j*16+i]<33||showdata[j*16+i]>127)

                {

                    if (showdata[j*16+i]=='\0')

                    {

                        sprintf(asii+strlen(asii)," ");

                    }

                    else

                    {

                        sprintf(asii+strlen(asii)," ");

                    }

                }

                else

                {

                    sprintf(asii+strlen(asii),"%c",showdata[j*16+i]);

                }

            }

            else

            {

                printf("   ");

                sprintf(asii+strlen(asii)," ");

            }

            i++;

        }

        while (i<16);

        printf(" %s",asii);

        j++;

    }



}

```

写的比较乱。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
