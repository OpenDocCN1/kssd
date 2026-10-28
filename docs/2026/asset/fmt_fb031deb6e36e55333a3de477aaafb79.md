
```
/*

c语言编写五子棋，暂时没有加入难度控制和重新开局功能。

*/

#include <stdio.h>

#include <Windows.h>//使用system("cls");

int main(void)

{

	int user[9][9]={0};//判断玩家1玩家2数组
	int line=9;//棋盘大小
	int count=0;//玩家轮流循环下棋次数

	int a=0,b=0;//构建棋局计数
	int d=0,e=0;//加载数组棋局计数
	int num1=0,num2=0;//用户输入棋盘如 5 5

	int hang_zong=2*line;//构建棋局行总数
	int lie_zong=hang_zong*2+1;//构建棋局列总数
	char c[19][39]={' '};//hang_zong+1,lie_zong+3   18  37棋局字符数组
	char gamestart=0;//接受用户是否开始游戏字符

	c[18][2]='1';//为了cls后有1,2,3,4,5,。。当然这样写比较尴尬。
	c[18][6]='2';
	c[18][10]='3';
	c[18][14]='4';
	c[18][18]='5';
	c[18][22]='6';
	c[18][26]='7';
	c[18][30]='8';
	c[18][34]='9';

	c[0][38]='1';
	c[2][38]='2';
	c[4][38]='3';
	c[6][38]='4';
	c[8][38]='5';
	c[10][38]='6';
	c[12][38]='7';
	c[14][38]='8';
	c[16][38]='9';

    system("color 70");
	for(a=0;a<hang_zong;a++)//构建棋局，赋值数组。
	{
		if ( a%2 ==0 ){//a等于偶数，字符为|   奇数__| 同时赋值c字符数组
			for(b=0;b<lie_zong;b++){
				if(b%4==0 ){//b等于0和4的倍数，字符为|  否则为空格‘ ’ 同时赋值c字符数组
					printf("|");
					c[a][b]='|';
				}
				else {
					printf(" ");
					c[a][b]=' ';
				}
			}
			printf("%2d\n",a/2+1);//行计数
		}
		else{
			for(b=0;b<lie_zong;b++){
				if(b%4==0){
					printf("|");
					c[a][b]='|';
				}
				else {
					printf("_");
					c[a][b]='_';
				}
			}
			printf("\n");
		}
	}

	for(a=0;a<lie_zong-1;a++)//列计数输出1,2,3,4,。。
	{
		if(a%4==0)
			printf(" %2d ",a/4+1);
	}
	printf("\n五子棋游戏玩法：\n输入2 9回车代表在第2行第9个框里下棋\t输入5 5回车代表在第5行第5个框里下棋。\n\n玩家1用\"X\"表示\t\t玩家2用\"#\"表示\t\t每次输入后更换选手进行下棋。\nLet's GO！！！！");

game:
	printf("\t\t\t按Y键开始游戏。");
	scanf("%c",&gamestart);
if(gamestart=='y' || gamestart == 'Y')
{
	// 开始游戏
	while(count<=line*line)//进行81次交换输入
	{
		system("cls");
		for(d=0;d<hang_zong+1;d++)//加载c字符数组
		{
			for(e=0;e<lie_zong+2;e++)
				printf("%c",c[d][e]);
			printf("\n");
		}

		printf("\n玩家1 X 请下棋：");
    agine1:
		scanf("%d %d",&num1,&num2);
		if (user [num1-1][num2-1] ==0)
		{
			c[2*num1-2][4*num2-3]='X';
			user[num1-1][num2-1]=1;
			system("cls");//当用户输入实时加载当前棋盘
			for(d=0;d<hang_zong+1;d++)
			{
				for(e=0;e<lie_zong+2;e++)
					printf("%c",c[d][e]);
				printf("\n");
			}
		}
		else{
			printf("\nERROR:输入的位置已经有棋子\n玩家1 X 请重新输入：");
			goto agine1;
		}

		////////////////////////////////////////////////
		//下面为核心计算是否形成5子。  左上，右上，左下，右下，左横，右横，上竖，下竖。 有点迷，慢慢看。
		//////////////GO GO！！！！！！！

		if(num1<=(line/2+1) && user[num1][num2]==1)//检查是否超出数组范围，然后检测右下是否形成5子
		{
			if( user[num1+1][num2+1]==1 )
			{
				if ( user[num1+2][num2+2]==1 )
				{
					if ( user[num1+3][num2+3] ==1)
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num1>=(line/2+1) && user[num1-2][num2-2]==1 ) //检查是否超出数组范围，然后检测左上是否形成5子
		{
			if( user[num1-3][num2-3]==1 )
			{
				if ( user[num1-4][num2-4]==1 )
				{
					if ( user[num1-5][num2-5]==1 )
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if(num2>=(line/2+1) && user[num1][num2-2] ==1)//检查是否超出数组范围，然后检测左下是否形成5子
		{
			if( user[num1+1][num2-3]==1 )
			{
				if ( user[num1+2][num2-4]==1 )
				{
					if ( user[num1+3][num2-5] ==1)
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num2<=(line/2+1) && user[num1-2][num2]==1 ) //检查是否超出数组范围，然后检测右上是否形成5子
		{
			if( user[num1-3][num2+1]==1 )
			{
				if ( user[num1-4][num2+2]==1 )
				{
					if ( user[num1-5][num2+3]==1 )
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if(num2<=(line/2+1) && user[num1-1][num2]==1 )//检查是否超出数组范围，然后检测右横是否形成5子
		{
			if( user[num1-1][++num2+1]==1 )
			{
				if ( user[num1-1][num2+2]==1 )
				{
					if ( user[num1-1][num2+3]==1 )
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num2>=(line/2+1) && user[num1-1][num2-2]==1 ) //检查是否超出数组范围，然后检测左横是否形成5子
		{
			if( user[num1-1][num2-3]==1 )
			{
				if ( user[num1-1][num2-4] ==1)
				{
					if ( user[num1-1][num2-5]==1 )
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if(num1>=(line/2+1) && user[num1-2][num2-1]==1 )//检查是否超出数组范围，然后检测上竖是否形成5子
		{
			if( user[num1-3][num2]-1==1 )
			{
				if ( user[num1-4][num2-1]==1 )
				{
					if ( user[num1-5][num2-1]==1 )
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num2<=(line/2+1) && user[num1][num2-1]==1 ) //检查是否超出数组范围，然后检测下竖是否形成5子
		{
			if( user[num1+1][num2-1] ==1)
			{
				if ( user[num1+2][num2-1] ==1)
				{
					if ( user[num1+3][num2-1]==1 )
					{
						printf("恭喜玩家1X取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		//玩家2
		printf("\n玩家2#请下棋：");
agine2:
		scanf("%d %d",&num1,&num2);
		if (user [num1-1][num2-1] ==0)
		{
			c[2*num1-2][4*num2-3]='#';
			user[num1-1][num2-1]=2;
			system("cls");
			for(d=0;d<hang_zong+1;d++)//加载c字符数组
			{
				for(e=0;e<lie_zong+2;e++)
					printf("%c",c[d][e]);
				printf("\n");
			}
		}
		else{
			printf("\nERROR:输入的位置已经有棋子\n玩家2#请重新输入：");
			getchar();
			goto agine2;
		}

		if(num1<=(line/2+1) && user[num1][num2]==2 )//检查是否超出数组范围，然后检测右下是否形成5子
		{
			if( user[num1+1][num2+1]==2 )
			{
				if ( user[num1+2][num2+2] ==2)
				{
					if ( user[num1+3][num2+3]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num1>=(line/2+1) && user[num1-2][num2-2]==2 ) //检查是否超出数组范围，然后检测左上是否形成5子
		{
			if( user[num1-3][num2-3] ==2)
			{
				if ( user[num1-4][num2-4]==2 )
				{
					if ( user[num1-5][num2-5] ==2)
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if(num2>=(line/2+1) && user[num1][num2-2]==2 )//检查是否超出数组范围，然后检测左下是否形成5子1111111111111111111111111111111111111111111111111111111111
		{
			if( user[num1+1][num2-3]==2 )
			{
				if ( user[num1+2][num2-4]==2 )
				{
					if ( user[num1+3][num2-5]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num2<=(line/2+1) && user[num1-2][num2] ==2 ) //检查是否超出数组范围，然后检测右上是否形成5子
		{
			if( user[num1-3][num2+1]==2 )
			{
				if ( user[num1-4][num2+2] ==2)
				{
					if ( user[num1-5][num2+3]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		///////////////////////////////////

		if(num2<=(line/2+1) && user[num1-1][num2]==2 )//检查是否超出数组范围，然后检测右横是否形成5子
		{
			if( user[num1-1][++num2+1]==2 )
			{
				if ( user[num1-1][num2+2]==2 )
				{
					if ( user[num1-1][num2+3]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num2>=(line/2+1) && user[num1-1][num2-2]==2 ) //检查是否超出数组范围，然后检测左横是否形成5子
		{
			if( user[num1-1][num2-3] ==2)
			{
				if ( user[num1-1][num2-4]==2 )
				{
					if ( user[num1-1][num2-5]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if(num1>=(line/2+1) && user[num1-2][num2-1]==2 )//检查是否超出数组范围，然后检测上竖是否形成5子
		{
			if( user[num1-3][num2-1]==2 )
			{
				if ( user[num1-4][num2-1]==2 )
				{
					if ( user[num1-5][num2-1]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		if( num2<=(line/2+1) && user[num1][num2-1]==2 ) //检查是否超出数组范围，然后检测下竖是否形成5子
		{
			if( user[num1+1][num2-1]==2 )
			{
				if ( user[num1+2][num2-1]==2 )
				{
					if ( user[num1+3][num2-1]==2 )
					{
						printf("恭喜玩家2#取得胜利！！！！\n按回车键结束");
						getchar();
						getchar();
						return 0;
					}
				}
			}
		}

		count++;
	}
}
else 
{
	printf("ERROR,请重新输入。\n");
	getchar();
	goto game;
}

	getchar();
	getchar();
	return 0;
}
```
