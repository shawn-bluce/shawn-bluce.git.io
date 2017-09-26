---
title: C语言 推箱子 gcc编译通过
date: 2015-10-19 12:12
---


# 0X00 编译环境
Ubuntu 14.04 + GNU/gcc
如果要在Windows下编译的话，*可能*需要注释掉16-18的编译预处理，还*可能*要注释掉getch()的函数声明及定义，最后在加上conio.h的头文件。

# 0X01 遇到的问题
Windows中能使用getch()函数，这个函数是以输入流的方式输入。（简单地说就是按下去一个按键就能有反应，而不用点击回车）。但是在Linux环境下没有这个函数也没有connio.h的头文件。但是每次按一下还要按回车还是挺逆天的。不过我在网上找到了替代品（感谢*幽鬼*）
>http://my.oschina.net/yougui/blog/111345

# 0X02 代码实现
```c
/*******************************************************************
 * Project name     : push the boxs
 * Create date      : 2015.10.17
 * Last modify date : 2015.10.19
 * Auther name      : mouse_ts
 * E-mail address   : michaelhaozi@hotmail.com
 * Description      : this is game, you control a boy push the boxs 
 * to the destination. but you can't push the stone and two boxs.
 * if you'r box touch the wall , you can't pull it.
 * ****************************************************************/

#include <stdio.h>
#include <string.h>
#include <math.h>
#include <stdlib.h>
#include <termios.h>//using getch()
#include <unistd.h>
#include <assert.h>

//this is constant
#define TRUE   1
#define FALSE  0
#define MAX    10
#define WALL   6
#define PLAYER 7
#define BOX    8
#define BLANK  5
#define DES    9
#define W      'w'
#define A      'a'
#define S      's'
#define D      'd'

//this is game map
int map[MAX][MAX];
int ok = 0;

//player
struct player
{
    int x;
    int y;
}player;

//boxs
struct box
{
    int x;
    int y;
}box_1, box_2, box_3;

//des
struct des
{
    int x;
    int y;
}des_1, des_2, des_3;

//statement function
void initMap();         //init the map
void initPlayer();      //init the player
void initBox();         //init the boxs
void initDes();         //init the des
void printMap();        //print the map
void setMap();          //set the player, boxs, des
char getch();           //getch()
void goUP();            //go up
void goDown();          //go down
void goLeft();          //go left
void goRight();         //go right
int  computingSuccess();//computing how many box seccessd

int main()
{
    char ch;

    system("clear");

    //init the game
    initMap();
    initPlayer();
    initBox();
    setMap();
    printMap();

    //control the boy
    while (ch = getch())
    {
        switch(ch)//where is the boy move
        {
            case W:
                goUP();
                break;
            case A:
                goLeft();
                break;
            case S:
                goDown();
                break;
            case D:
                goRight();
                break;
            defualt:
                printf ("You should press w, a, s, d to control the boy to move\n");
        }
        setMap();
        system("clear");
        printMap();

        if (computingSuccess() == 3)
            break;
        else
            continue;
    }
    system("clear");
    printf ("\n\n\n\n\n\n\n             You win the game!\n");
    getch();
    system("clear");
    return 0;
}

//getch()   by.YouGui   http://my.oschina.net/yougui/blog/111345
char getch()
{
    int c = 0;
    struct termios org_opts, new_opts;
    int res = 0;
    res = tcgetattr(STDIN_FILENO, &org_opts);
    assert(res == 0);
    memcpy(&new_opts, &org_opts, sizeof(new_opts));
    new_opts.c_lflag &= ~(ICANON | ECHO | ECHOE | ECHOK | ECHONL | ECHOPRT | ECHOKE | ICRNL);
    tcsetattr(STDIN_FILENO, TCSANOW, &new_opts);
    c = getchar();
    res = tcsetattr(STDIN_FILENO, TCSANOW, &org_opts);
    assert(res == 0);
    return c;
}

//init this map
void initMap()
{
    int i, j;

    for (i = 0; i < MAX; i++)
    {
        for (j = 0; j < MAX; j++)
        {
            map[i][j] = WALL;
        }
    }
    for (i = 2; i < 8; i++)
    {
        map[i][2] = BLANK;
        map[i][3] = BLANK;
        map[i][5] = BLANK;
        map[i][6] = BLANK;
        map[i][7] = BLANK;
    }
    map[5][4] = BLANK;

    initDes();
}

//print map
void printMap()
{
    printf ("This is a game !\n");

    int i, j;
    for (i = 0; i < MAX; i++)
    {
        for (j = 0; j < MAX; j++)
        {
            if (map[i][j] == WALL)
                printf (" # ");
            else if (map[i][j] == BOX)
                printf (" @ ");
            else if (map[i][j] == PLAYER)
                printf (" X ");
            else if (map[i][j] == BLANK)
                printf ("   ");
            else if (map[i][j] == DES)
                printf (" O ");
        }
        printf ("\n");
    }
}

//init the player
void initPlayer()
{
    player.x = 2;
    player.y = 2;
}

//init the boxs
void initBox()
{
    box_1.x = 3;
    box_1.y = 6;

    box_2.x = 4;
    box_2.y = 3;

    box_3.x = 6;
    box_3.y = 3;
}

//init the des
void initDes()
{
    des_1.x = 5;
    des_1.y = 7;

    des_2.x = 6;
    des_2.y = 7;

    des_3.x = 7;
    des_3.y = 7;
}

//set map
void setMap()
{
    int i, j;

    //set blank
    for (i = 2; i < 8; i++)
    {
        map[i][2] = BLANK;
        map[i][3] = BLANK;
        map[i][5] = BLANK;
        map[i][6] = BLANK;
        map[i][7] = BLANK;
    }
    map[5][4] = BLANK;


    //set des
    map[des_1.x][des_1.y] = DES;
    map[des_2.x][des_2.y] = DES;
    map[des_3.x][des_3.y] = DES;

    //set player
    map[player.x][player.y] = PLAYER;

    //set box
    map[box_1.x][box_1.y] = BOX;
    map[box_2.x][box_2.y] = BOX;
    map[box_3.x][box_3.y] = BOX;
}

//computing the success move the box to the des
int computingSuccess()
{
    int num = 0;
    if (map[des_1.x][des_1.y] == BOX)
        num++;
    if (map[des_2.x][des_2.y] == BOX)
        num++;
    if (map[des_3.x][des_3.y] == BOX)
        num++;
    return num;
}

/*
 * after this is control your boy to move up down left and right
 * all of the  functions to control the boy to move
 */

//control the boy go up
void goUP()
{
    if (map[player.x - 1][player.y] == BLANK ||
        map[player.x - 1][player.y] == DES)
    {
        player.x--;
        return ;
    }
    if (player.x - 1 == box_1.x && player.y == box_1.y &&
        map[box_1.x - 1][box_1.y] == BLANK ||
        player.x - 1 == box_1.x && player.y == box_1.y &&
        map[box_1.x - 1][box_1.y] == DES)
    {
        box_1.x--;
        player.x--;
        return ;
    }
    else if (player.x - 1 == box_2.x && player.y == box_2.y &&
             map[box_2.x - 1][box_2.y] == BLANK ||
             player.x - 1 == box_2.x && player.y == box_2.y &&
             map[box_2.x - 1][box_2.y] == DES)
    {
        box_2.x--;
        player.x--;
        return ;
    }
    else if (player.x - 1 == box_3.x && player.y == box_3.y &&
             map[box_3.x - 1][box_3.y] == BLANK ||
             player.x - 1 == box_3.x && player.y == box_3.y &&
             map[box_3.x - 1][box_3.y] == DES)
    {
        box_3.x--;
        player.x--;
        return ;
    }
}

//control the boy go down
void goDown()
{
    if (map[player.x + 1][player.y] == BLANK ||
        map[player.x + 1][player.y] == DES)
        player.x++;
    if (player.x + 1 == box_1.x && player.y == box_1.y &&
        map[box_1.x + 1][box_1.y] == BLANK ||
        player.x + 1 == box_1.x && player.y == box_1.y &&
        map[box_1.x + 1][box_1.y] == DES)
    {
        box_1.x++;
        player.x++;
        return ;
    }
    else if (player.x + 1 == box_2.x && player.y == box_2.y &&
             map[box_2.x + 1][box_2.y] == BLANK ||
             player.x + 1 == box_2.x && player.y == box_2.y &&
             map[box_2.x + 1][box_2.y] == DES)
    {
        box_2.x++;
        player.x++;
        return ;
    }
    else if (player.x + 1 == box_3.x && player.y == box_3.y &&
             map[box_3.x + 1][box_3.y] == BLANK ||
             player.x + 1 == box_3.x && player.y == box_3.y &&
             map[box_3.x + 1][box_3.y] == DES)
    {
        box_3.x++;
        player.x++;
        return ;
    }
}

//control the boy go left
void goLeft()
{
    if (map[player.x][player.y - 1] == BLANK ||
        map[player.x][player.y - 1] == DES)
        player.y--;
    if (player.x == box_1.x && player.y - 1 == box_1.y &&
        map[box_1.x][box_1.y - 1] == BLANK ||
        player.x == box_1.x && player.y - 1 == box_1.y &&
        map[box_1.x][box_1.y - 1] == DES)
    {
        box_1.y--;
        player.y--;
        return ;
    }
    else if (player.x == box_2.x && player.y - 1 == box_2.y &&
             map[box_2.x][box_2.y - 1] == BLANK ||
             player.x == box_2.x && player.y - 1 == box_2.y &&
             map[box_2.x][box_2.y - 1] == DES)
    {
        box_2.y--;
        player.y--;
        return ;
    }
    else if (player.x == box_3.x && player.y - 1 == box_3.y &&
             map[box_3.x][box_3.y - 1] == BLANK ||
             player.x == box_3.x && player.y - 1 == box_3.y &&
             map[box_3.x][box_3.y - 1] == DES)
    {
        box_3.y--;
        player.y--;
        return ;
    }
}

//control the boy go right
void goRight()
{
    if (map[player.x][player.y + 1] == BLANK ||
        map[player.x][player.y + 1] == DES)
        player.y++;
    if (player.x == box_1.x && player.y + 1 == box_1.y &&
        map[box_1.x][box_1.y + 1] == BLANK ||
        player.x == box_1.x && player.y + 1 == box_1.y &&
        map[box_1.x][box_1.y + 1] == DES)
    {
        box_1.y++;
        player.y++;
        return ;
    }
    else if (player.x == box_2.x && player.y + 1 == box_2.y &&
             map[box_2.x][box_2.y + 1] == BLANK ||
             player.x == box_2.x && player.y + 1 == box_2.y &&
             map[box_2.x][box_2.y + 1] == DES)
    {
        box_2.y++;
        player.y++;
        return ;
    }
    else if (player.x == box_3.x && player.y + 1 == box_3.y &&
             map[box_3.x][box_3.y + 1] == BLANK ||
             player.x == box_3.x && player.y + 1 == box_3.y &&
             map[box_3.x][box_3.y + 1] == DES)
    {
        box_3.y++;
        player.y++;
        return ;
    }
}
```
