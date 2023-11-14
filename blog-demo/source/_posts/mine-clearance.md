---
title: =C语言实践= 手把手教你做高端cmd简单扫雷
date: 2023-10-30 07:28:01
tags:
cover: https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-02_10-27-25.jpg
---
# 直接开始吧！

## 多文件项目 #
扫雷项目内容较多，需要调用的**函数**也较多，采用多文件的方式，可以使代码**条理清晰**，并且**易于管理和维护**。文件如下

`game.h`用于宏定义，函数声明，引入头文件等

`game.c`用于**函数的具体实现**

`front.c`用于实现程序的**主干部分**

`other.c`用于实现其他杂项函数，这里我用于实现`menu()`函数，~~主要内容太花了~~

**注**  `.c`结尾的源文件均需加一句`#include "game.h"`

## 头文件 #
本次用到的头文件有`stdio.h` `stdlib.h` `time.h` `windows.h`
和自己建的`game.h`

**均在**文件`game.h`中`#include`


## define宏定义 #
为了便于**阅读和维护**代码,在`game.h`中的宏定义如下

```C
//显示行列
#define ROW 9
#define COL 9

//实际数组大小
#define ROWS ROW+2
#define COLS COL + 2

//地雷信息
#define Bomb '*'
#define Blank ' '

//难度
#define EZ_RANK 10
#define HD_RANK 15

//显示区
#define UN ''
#define Flag '!'

```

### 为什么实际数组要大一圈？ #

如图，采用九宫格式访问时，大出来的一圈能有效**防止越界访问**

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/Snipaste_2023-11-02_11-23-51.jpg)


## 构建main函数 #
内容不多，主要是与菜单配合食用
```C
int main()
{
    //用time()获取时间戳，传给srand设置(随机值生成器的)种子
	srand((unsigned int)time(NULL));
	while (1)//循环游玩
	{
		int input = 0;
		Menu();//打印菜单
		printf("请输入:>");
		scanf("%d", &input);//获取指令
		switch (input)
		{
		case 1:
			Sleep(250);
			game();//开始游戏,游戏具体在game()函数中实现
			break;
		case 2:
			printf("游戏结束\n");
			return 0;//结束程序
		default:
			printf("输入错误,请重试\n");
			break;
		}
	}

	return 0;
}

```

## 打印菜单 #
还在做静态菜单?~~弱爆了！~~来试试**动态出现**的菜单！

![](https://picbed0521.oss-cn-shanghai.aliyuncs.com/blogpic/menu.gif)

原理很**简单**，就是`打印空白数组`->`向内逐个替换两侧元素`->`清屏`->`再打印`->`再替换`->`...`

*接下来的代码写在`other.c`中*

```C
//动态打印菜单
void Menu()
{
	char cover[]  =  "=======================";
	char option1[] = "======  play (1) ======";
	char option2[] = "======  exit (2) ======"; 
	char empty_c[] = "                       ";
	char empty_1[] = "                       ";
	char empty_2[] = "                       ";

	int left = 0;
	int right = 22;
	while (left < right)
	{
        //内容替换
		empty_c[left] = cover[left];
		empty_c[right] = cover[right];
		empty_1[left] = option1[left];
		empty_1[right] = option1[right];
		empty_2[left] = option2[left];
		empty_2[right] = option2[right];

		Sleep(50);
		system("cls");//清屏
		printf("%s\n%s\n%s\n%s\n",empty_c,empty_1,empty_2,empty_c);//打印菜单

		left++;
		right--;

	}
	if (left == right)//打印最终菜单
	{
		Sleep(50);
		system("cls");
		empty_c[left] = cover[left];
		empty_1[left] = option1[left];
		empty_2[left] = option2[left];
		printf("%s\n%s\n%s\n%s\n", empty_c, empty_1, empty_2, empty_c);
	}
}
```

## 实现game()函数 #
*游戏的主要逻辑在game()中实现*

```C
void game()
{
	char mine[ROWS][COLS] = { 0 };
	char show[ROWS][COLS] = { 0 };
	char check[ROWS][COLS] = { 0 };

    //初始化棋盘，其实就是用第二个形参填充二维数组
	InitBoard(mine, Blank);
	InitBoard(show, UN);

	InitCheck(check);//初始化check数组，逻辑与上面的初始化不同

	SetMine(mine,EZ_RANK);//设置地雷
	SetNum(mine);//设置雷周围的数字

	//DisplayBoard(mine); //用于开发时检查棋盘布局
	//DisplayBoard(show); //同上，不使用时注释掉

    //以上是前期准备
	OPMine(mine,show, check);//开始排雷

	printf("敲击enter以继续\n");
	getchar();
	getchar();

}
```

### 为什么用三个二维数组？ #
扫雷需要实现的功能较多，显然一个二维数组是不足以满足需求的，所以这里采用**三个**数组相叠加的方式，各自实现功能，并整合到一起。

数组`mine`用于存放`雷`和雷周围的`计数数字`

数组`show`用于储存给`用户`看到的内容，可以是`Unkown`,`空白`，`数字`，`旗帜`

数组`check`用于记录棋盘的哪些地块被检查过了，防止后面用**递归**打开成片的空白区时，出现无限递归。

**规定**：检查过的坐标储存`字符1`,没检查过的坐标储存`字符0`,大出来的**一圈**默认储存`字符1`

## 实现游戏用的函数 #
### 先看看有哪些要声明在`game.h`里的 #
```C
void Menu();//这个在上文实现过了

//一下函数将在下文实现

//初始化棋盘
void InitBoard(char board[ROWS][COLS], char sign);
//展示棋盘
void DisplayBoard(char board[ROWS][COLS]);
//初始化check棋盘
void InitCheck(char check[ROWS][COLS]);
//设置地雷/数字
void SetMine(char board[ROWS][COLS],int rank);
void SetNum(char board[ROWS][COLS]);
//玩家排雷用的函数
void OPMine(char mine[ROWS][COLS],char show[ROWS][COLS],char check[ROWS][COLS]);

```
*好，有了目标，接下来就去一个一个实现*

**注**：以下代码均写在`game.c`文件里

### InitBoard() #
```C
void InitBoard(char board[ROWS][COLS],char sign)
{
	for (int i = 0; i < ROWS; i++)
	{
		for (int j = 0; j < COLS; j++)
		{
			board[i][j] = sign;
		}
	}
}
```
这里初始化的方式比较简单粗暴，就是用形参`sign`填充整个二维数组

### DisplayBoard()函数 #
这里采用的展示方式是带横纵坐标的
```C
void DisplayBoard(char board[ROWS][COLS])
{
	//打印一排列坐标
	for (int k = 0; k <= COL; k++)
	{
		printf("%d ", k);
	}
	printf("\n");
	//打印一排横分割线
	for (int k = 0; k <= COL; k++)
	{
		printf("--");
	}
	printf("\n");
	for (int i = 1; i <= ROW; i++)
	{
		printf("%d|", i);//这句话打印横排坐标和竖分割线
		//打印一排棋盘内容
		for (int j = 1; j <= COL; j++)
		{
			printf("%c ", board[i][j]);
		}
		printf("\n");
	}
}

```
### InitCheck()函数 #
这里复用了`InitBoard()`函数，是在其基础上增加了内容

```C
void InitCheck(char check[ROWS][COLS])
{
	InitBoard(check, '0');

	//将边缘大出来的一圈改为'1'
	for (int k = 0; k < COLS; k++)
	{
		check[0][k] = '1';
		check[ROWS - 1][k] = '1';
	}
	for (int i = 1; i < ROWS -1; i++)
	{
		check[i][0] = '1';
		check[i][COLS - 1] = '1';
	}
}

```

### SetMine()函数 #
这里要使用`rand()`函数搭配`%`运算，来随机生成雷的坐标

```C
void SetMine(char board[ROWS][COLS],int rank)
{
	int x = 0;
	int y = 0;
	for (int count = 0;count < rank;)
	{
		x = rand() % ROW + 1;//x的范围是1~ROW
		y = rand() % COL + 1;

		if (board[x][y] == Blank)
		{
			count++;
			board[x][y] = Bomb;
		}
	}
}

```

### SetNum()函数 #
这里遍历一遍数组并采用`九宫格式`计数
```C
//该函数用于九宫格式计数，并在下个函数中被调用
int CountMine(char board[ROWS][COLS], int x, int y)
{
	int sum = 0;
	for (int i = x - 1; i <= x + 1; i++)//上中下三行
	{
		for (int j = y - 1; j <= y + 1; j++)//左中右三列
		{
			if (i != x || j != y)
			{
				if (board[i][j] == Bomb)
				{
					sum++;
				}

			}
		}
	}
	return sum;
}

void SetNum(char board[ROWS][COLS])
{
	//遍历二维数组
	for (int i = 1; i <= ROW; i++)
	{
		for (int j = 0; j <= COLS; j++)
		{
			if (board[i][j] == Blank)//仅操作非雷的格子
			{
				if (CountMine(board, i, j))
				{
					board[i][j] = '0' + CountMine(board, i, j);//将返回的数字转化成字符
				}
			}
		}
	}
}

```

### OPMine()函数--核心函数 #
该函数为游戏的`核心函数`，有内置菜单，且多次调用其它函数，其中`函数`的具体实现见`四级标题`处

```C
void OPMine(char mine[ROWS][COLS],char show[ROWS][COLS],char check[ROWS][COLS])
{
	int x = 0;//横纵坐标
	int y = 0;
	int flag = 1;//用于菜单选项
	int cont = 1;//cont为0时游戏结束
	while (cont)
	{
		system("cls");
		DisplayBoard(show);
		printf("排雷(1)\n插旗/拔旗(2)\n请输入:>");
		scanf("%d", &flag);
		
		switch (flag)
		{
			case 1:
			{
				//排雷
				printf("坐标格式,例>2(空格)2\n");
				printf("请输入坐标:>");
				scanf("%d %d", &x, &y);
				if (show[x][y] == Flag)
				{

					printf("此处为旗帜，不可排雷\n");
					break;
				}
				else if (show[x][y] != UN)
				{
					printf("不可重复排查\n");
					break;
				}
				//具体排雷操作
				cont = FindMine(mine,show,check, x, y);
				if (cont)
				{
					//检查是否赢得游戏
					cont = CheckWin(mine,show);
				}
				break;
			}
			case 2:
			{
				//插旗
				printf("坐标格式,例>2(空格)2\n");
				printf("请输入坐标:>");
				scanf("%d %d", &x, &y);
				SetFlag(show, x, y);//插旗函数
				break;
			}
			default:
			{
				system("cls");
				printf("\n输入错误(恼\n");
			}
		}
	}
}

```

#### SetFlag()函数 #
先捏软柿子，插旗函数比较简单

```C
void SetFlag(char show[ROWS][COLS], int x, int y)
{
	if (show[x][y] == UN)//插旗
	{
		show[x][y] = Flag;
	}
	else if (show[x][y] == Flag)//拔旗
	{
		show[x][y] = UN;
	}
	else
	{
		printf("报错\n");
	}
}

```
#### ExpandBlank()函数 #
这个函数用于打开成片的`空白区`,因为要从连着的空白连续开下去，所以要用到`函数递归`,此时`二维数组check`用于防止死递归

**注**：这个函数一定要写在下一个函数(FindMine)前

```C
void ExpandBlank(char mine[ROWS][COLS], char show[ROWS][COLS], char check[ROWS][COLS],int x,int y)
{
	show[x][y] = mine[x][y];//将用户看到的格子改成mine中的格子,包括空白和数字格子
	check[x][y] = '1';//探测过的格子放`1`
	if (mine[x][y] == Blank)//仅空白格子会触发递归，数字格子不会
	{
		//九宫格式探测
		for (int i = x - 1; i <= x + 1; i++)
		{
			for (int j = y - 1; j <= y + 1; j++)
			{
				if (check[i][j] == '0' && mine[i][j] != Bomb && show[i][j] != Flag)
				{
					//含雷的格子不会执行ExpandBlank函数，就不会把雷放出来给用户看，但数字格子会
					ExpandBlank(mine, show, check, i, j);
				}
			}
		}
	}
}
```

#### FindMine()函数 #
排雷用的函数

```C
int FindMine(char mine[ROWS][COLS], char show[ROW][COLS],char check[ROWS][COLS], int x, int y)
{
	if (mine[x][y] == Bomb)
	{
		DisplayBoard(mine);
		printf("炸死，游戏结束:)\n");
		return 0;//返回0来结束游戏
	}
	else if (mine[x][y] != Blank)
	{
		show[x][y] = mine[x][y];
		return 1;//返回1来继续游戏
	}
	else
	{
		//这里有对上一个函数的调用
		ExpandBlank(mine, show, check,x,y);
		return 1;
	}
}
```
#### CheckWin()函数 #
用于检查玩家是否完全排雷，赢得游戏

```C
int CheckWin(char mine[ROWS][COLS], char show[ROWS][COLS])
{
	int count = 0;//统计没排雷的格子数
	for (int i = 1; i <= ROW; i++)
	{
		for (int j = 1; j <= COL; j++)
		{
			if (show[i][j] == UN || show[i][j] == Flag)
			{
				count++;
			}
		}
	}
	if (count == EZ_RANK)//统计数==雷数
	{
		printf("恭喜排雷成功!\n");
		DisplayBoard(mine);
		return 0;//返回0，停止游戏
	}
	else
	{
		return 1;
	}
}
```

# 总结 #
至此游戏所需的代码全部完成，已经可以编译出来玩耍啦。

该实践项目主要练习了`二维数组`,`函数`,`函数递归`,`宏定义`等内容，代码量在入门学习中算较大的，本人在初次编写的时候也写出了不少bug，debug的过程是相当~~快乐~~

建议多多画示意图，**耐下性子**写代码和debug,哪怕是实现这样的小游戏项目，也是颇有意义的