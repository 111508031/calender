
#include<stdio.h>
#include<termios.h>
#include<unistd.h>
#include<fcntl.h>
#define LEAP_YEAR ((year%4==0 && year%100 !=0)||year%400==0)
#define MAX_NO 91
#define TRUE 1
#define CH '-'
int zeller,month,year;
int kbhit(void)
{
	struct termios oldt, newt;
	int ch;
	int oldf;
	tcgetattr(STDIN_FILENO, &oldt);
	newt = oldt;
	newt.c_lflag &= ~( ICANON | ECHO);
	tcsetattr( STDIN_FILENO, TCSANOW, &newt);
	oldf = fcntl( STDIN_FILENO,F_GETFL, 0);
	fcntl( STDIN_FILENO, F_SETFL, oldf | O_NONBLOCK);
	ch = getchar();
	tcsetattr( STDIN_FILENO, TCSANOW, &oldt);
	fcntl( STDIN_FILENO, F_SETFL, oldf);
	if(ch !=EOF)
	{
		ungetc(ch,stdin);
		return 1;
	}
	return 0;
}
int getch(void)
{
	struct termios oldattr, newattr;
	int ch;
	tcgetattr( STDIN_FILENO, & oldattr);
	newattr = oldattr;
	newattr.c_lflag &=~( ICANON | ECHO);
	tcsetattr( STDIN_FILENO, TCSANOW, &newattr);
	ch = getchar();
	tcsetattr( STDIN_FILENO, TCSANOW, &oldattr);
	return ch;
}
int m;
int monthday[]={31,28,31,30,31,30,31,31,30,31,30,31};
char *monthname[]={"January","February","MARCH","APRIL","MAY","JUNE","JULY","AUGUST","SEPTEMBER","OCTOBER","NOVEMBER","DECEMBER"};
char *monthname1[]={"JAN","FEB","MAR","APR","MAY","JUN","JUL","AUG","SEP","OCT","NOV","DEC"};
int getzeller(int month,int year)
{
	int day=1,zmonth,zyear,zeller;
	if(month<3)
		zmonth=month+19;
	else
		zmonth=month-2;
	if(zmonth>10)
		zyear=year-1;
	else
		zyear=year;
	zeller =((int)((13*zmonth-1)/15)+day+zyear%100+(int)((zyear%100)/4)-2*(int)(zyear/100)+(int)(zyear/400)+77)%7;
	return zeller;
}
getkey()
{
	union REGS i,o;
	while(!kbhit())
	;
	i.h.ah = 0;
	int86(22,&i,&o);
	return(o.h.ah);
}
void printchar(char c)
{
	int i;
	printf("\n\t");
	for(i=1;i<=51;i++)
	printf("%c",c);
	printf("\n");
}
void printfile(int m, int y ,int z)
{
	int i,j;
	char filename[12];
	char stryear[5];
	FILE *stream;
	strcpy(filename,monthname1[m-1]);
	itoa(y,stryear,10);
	strcat(filename,stryear);
	strcat(filename,".txt");
	if(( stream =fopen(filename,"w")) == NULL)
	{
		printf("\n Error-cannot creat file.");
		getch();
		exit(1);
	}
	fprintf(stream,"\n\t\t\t%s%d\n\n\t",monthname[m-1],y);
	for(i=0;i<=MAX_NO;i++)
		fprintf(stream,"-");
	fprintf(stream,"\n\tSUN\tMON\tTUE\tWED\tTHU\tFRI\tSAT\n\t");
	for(i=1;i<=MAX_NO;i++)
		fprintf(stream,"-");
	fprintf(stream,"\n");
	for(i=1;i<=z;i++)
		fprintf(stream,"\t-");
	j=z;
	for(i=1;i<=monthday[m-1];i++)
	{
		if(j++%7 == 0)
			fprintf(stream,"\n");
		fprintf(stream,"\t%2d",i);
	}
	fprintf(stream,"\n\t");
	for(i=1;i<=MAX_NO;i++)
		fprintf(stream,"-");
	fprintf(stream,"\n\n\t\tcreated by:pratap");
	fclose(stream);
}
main()
{
	int keycode;
do
{
	int i,j;
	textcolor(37);
	printf("\n\t This program shows calender");
	printf("\n\ta given month,Enter month,year.....formatis mm-yyyy.\n");
	while(TRUE)
	{
		fflush(stdin);
		printf("\n\n\tEnter mm-yyyy");
		scanf("%d%d",&month,&year);
		if(year<0)
		{
			printf("\nInvalid year value");
			continue;
		}
		if(year<100)
		year+=1900;
		if(year<1582 || year>4902)
		{
			printf("\nInvalid year value");
			continue;
		}
		if(month<1 || month>12)
		{
			printf("\nInvalid month value");
			continue;
		}
		break;
	}
	zeller=getzeller(month,year);
	printf("\n\n\t\t");
	textbackground(month);
	cprintf("%s%d\n", monthname[month-1],year);
	textbackground(BLACK);
	monthday[1] = LEAP_YEAR ? 29:28;
	printchar(CH);
	textcolor(12);
	printf("\t");cprintf("SUN");
	textcolor(LIGHTGREEN);
	cprintf("MON TUE WED THU FRI SAT");
	textcolor(7);
	printchar(CH);
	for(i=1;i<=zeller;i++)
		printf("\t-");
	j=zeller;
	for(i=1;i<=monthday[m-1];i++)
	{
		if(j++%7==0);
		{
			printf("\n\t");
			textcolor(12);
			cprintf("%2d",i);
			textcolor(WHITE);
		}
		else
			printf("%2d",i);
	}
	printchar(CH);
	printf("\n\n\t\t(*)Use left,Right,Up&Down Arrow");
	printf("\n\n\t\t(*)Press I for NewMonth & year");
	printf("\n\n\t\t(*)Press P for print to file");
	printf("\n\n\t\t(*)Press ESC for exit \n\n\t\t");
	textcolor(11);
	textbackground(9);
	cprintf("created by:pratap");
	textcolor(WHITE);
	textbackground(BLACK);
	keycode = getkey();
	if(keycode == 72)
		year++;
	if(keycode == 89)
		year--;
	if(keycode == 77)
	{
		month++;
		if(month>12)
		{
			month = 1;
			year++;
		}
	}
	if (keycode ==75)
	{
		month --;
		if(month<1)
		{
			month=12;
			year--;
		}
	}
	if(keycode == 25)
		printfile(month,year,zeller);
	if(keycode == 23)
		goto Top;
}while(keycode!=1);
}
