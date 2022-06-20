# 123m
atm机
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<windows.h>
#include<conio.h>
#include<time.h>
#define Y "E:/编程/ATM机/用户数据.txt"
#define L "E:/编程/ATM机/流水记录.txt"

struct Account
{

	char Name[20];//姓名 
	char tel[12];//电话 	
	char userName[17];//银行账号卡号
	char passWord[7];//密码 
	char bank[20];//归属银行 
	float money;//余额

	struct Account* next; //下一个账户结点的地址 
};
typedef struct Account Account;

struct FlowRecord
{
	char username[17];
	time_t timestamp;//时间戳 ,time_t是一种数据类型 
	char type[5]; 
	float money;
	float serbiceCharge;//手续费 
	float balance;//交易后的余额 
	
	struct FlowRecord * next; 
}; 
typedef struct FlowRecord FR;

int loading();;
int initialization();
int signin();
int finduserName(char userName[]);
int findOtherAccount(char userName[]);
int immediate();
int continueTrading();
int Withdraw_Money();
int continueTransfer();
int changepassword();
int menu();

Account *head=NULL;
Account *tail=NULL;
Account *nowAccount=NULL;//当前登录账户 
Account *otherAccount=NULL;//另一个账户 
Account *otherhead=NULL;//被吞卡的链表的头节点 
Account *othertail=NULL;//被吞卡的链表的尾节点 
Account *othernow=NULL;//遍历吞卡链表的指针

FR *frhead=NULL;
FR *frtail=NULL;
FR *frnow=NULL;

const char bank[13]="啥也不会银行";

void tuxiang()
{
		system("CLS"); 
		printf("--------------------------------\n");
		printf("-------");
		printf("欢迎来到啥也不会银行");
		printf("-------\n");
		printf("--------------------------------\n");
}

int loading()
{
	printf("加载中......\n");
	FILE*fp=fopen(Y,"r");
	if(fp==NULL)
	{
		Sleep(1000);
		printf("加载失败\n");
		return 0;
	}
	else
	{ 
		while(!feof(fp))//创建链表 
		{
			Account*newAccount=(Account*)calloc(1,sizeof(Account));
			newAccount->next=NULL;
			fscanf(fp,"%s %s %s %s %s %f\n",newAccount->Name,newAccount->tel,newAccount->userName,newAccount->passWord,newAccount->bank,&newAccount->money);
			if(newAccount==NULL)
			{
				printf("加载失败\n");
				return 0;
			}
			if(newAccount->userName[1]=='\0')
			{
				free(newAccount);
				continue;
			}
			if(head==NULL)
			{
				head=newAccount;
				tail=newAccount;
			}
			else 
			{
				tail->next=newAccount;
				tail=newAccount;
			}
		}//创建链表结束
	fclose(fp);
	}
	
	fp=fopen(L,"r");
	if(fp==NULL)
	{
		Sleep(1000);
		printf("加载失败\n");
		return 0;
	}
	else 
	{
		while(!feof(fp))
		{
			FR*flowrecord=(FR*)calloc(1,sizeof(FR));
			flowrecord->next=NULL;
			fscanf(fp,"%s %ld %s %f %f %f",flowrecord->username,&flowrecord->timestamp,flowrecord->type,&flowrecord->money,&flowrecord->serbiceCharge,&flowrecord->balance);
			if(flowrecord==NULL)
			{
				printf("加载失败\n");
				return 0;
			}
			
			if(flowrecord->username[1]=='\0')
			{
				free(flowrecord);
				continue;
			}
			
			if(frhead==NULL)
			{
				frhead=flowrecord;
				frtail=flowrecord;
			}
			else
			{
				frtail->next=flowrecord;
				frtail=flowrecord;
			}
		}
		fclose(fp);

		printf("加载成功\n");
		Sleep(1000);
	}
	return 1;
}

int initialization()//初始化界面 
{
	char a;
	printf("1、登录\t\t\t\n");
	printf("--------------------------------\n");
	printf("2、开户\n");
	printf("--------------------------------\n");
	printf("3、退出系统\t\t\n");
	printf("--------------------------------\n");
	while(1)
	{
		printf("请选择您要办理的业务:\n");
		a=getche();
		putchar('\n');
		if(a=='1')
		{
			return 1;
		}
		else if(a=='2')
		{
			return 2;
		}
		else if(a=='3')
		{
			return 0;
		}
		else 
		{
			printf("您的输入有误，请重新输入\n");
		}
	}
}

int  finduserName(char userName[])
{
	char c;
	othernow=otherhead;
	while(othernow !=NULL)
	{
		if(strcmp(userName,othernow->userName)==0)
		{
			printf("检测到您输入的卡号已被吞，是否进行取回?\n");
			while(1)
			{
				printf("1、取回\n2、不取回\n");
				c=getche();
				putchar('\n');
				if( c=='1' )
				{
					return 2;
				}
				else if(c=='2')
				{
					return 3;
				}
				else 
				{
					printf("您的输入有误，请重新输入\n");
					continue;
				}
			}
		}
		else
		{
			othernow=othernow->next;
		}
	}
	
	nowAccount=head;
	while(nowAccount!=NULL)
	{
		if(strcmp(userName,nowAccount->userName)==0)
		{
			return 1;
		}
		else 
		{
			nowAccount=nowAccount->next;
		}
	}
	return 0;
}

void dispose(char userName[])//处理吞卡 
{
	Account *pothernow=NULL;//pothernow用于遍历吞卡链表 
	
	othernow=otherhead;//othernow用于在吞卡链表中寻找被吞卡 
	while(othernow!=NULL)
	{
		if(strcmp(othernow->userName,userName)==0)
		{
			if(head==NULL)//将当前节点移动至常规链表 
			{
				head=othernow;
				tail=othernow;
			}
			else 
			{
				tail->next=othernow;
				tail=othernow;
			}
			break;
		}
		else 
		{
			othernow=othernow->next;
		}
	}
	
	pothernow=otherhead;//删除吞卡链表中的该节点 
	if(othernow==otherhead)
	{
		if(otherhead->next==NULL)
		{
			otherhead=NULL;
			othertail=NULL;
		}
		else 
		{
			otherhead=otherhead->next;
		}
	}
	else if(otherhead!=othertail&&othernow==othertail)
	{
		pothernow=otherhead;
		while(pothernow!=NULL)
		{
			if(strcmp((pothernow->next)->userName,othertail->userName)==0)
			{
				pothernow->next=NULL;
				othertail=pothernow;
				break;
			}
			else 
			{
				pothernow=pothernow->next;
			}
		}
	}
	else 
	{
		pothernow=otherhead; 
		while(pothernow!=NULL)
		{
			if(strcmp((pothernow->next)->userName,pothernow->userName)==0)
			{
				pothernow->next=(pothernow->next)->next;
				(pothernow->next)->next=NULL;
				break;
			}
			else 
			{
				pothernow=pothernow->next;
			}
		}
	}
	othernow->next=NULL;
}


int signin()//登录 
{
	Account x;
	int a;
	char nowuserName_[100];
	while(1)//判断卡号是否存在 
	{
		printf("请输入银行卡号:\n");
		scanf("%s",nowuserName_);
		if(strlen(nowuserName_)!=16)
		{
			printf("请输入正确的银行卡号\n");
			continue;
		}
		else
		{
			strcpy(x.userName,nowuserName_);
			int i=finduserName(x.userName);
			if(i==1)
			{
				break;
			}
			else if(i=2)//取回被吞卡 
			{
				dispose(x.userName);
				printf("处理成功\n");
				return 1;
			}
			else if(i=3)//不取回
			{
				continue;
				
			} 
			else 
			{
				printf("请输入正确的银行卡号\n");
			}
		}
	}
	for(a=0;a<3;a++)//判断密码是否正确 ,a代表密码已经错误的次数 
	{
		printf("请输入密码\n");
		for(int i=0;i<6;i++)
		{
			x.passWord[i]=getch();
			putchar('*');
		}
		if(strcmp(nowAccount->passWord,x.passWord)==0) 
		{
			putchar('\n');
			printf("登录成功\n");
			printf("正在进入菜单...\n");
			Sleep(1000);
			return 2;
		}
		else
		{
			printf("\n您的密码输入错误，请重新输入\n");
		}
	}
	if(a==3)
	{
		printf("密码连续输错三次，请到柜台人工处理\n");
		
		othernow=nowAccount;//将当前节点移动至吞卡链表 
		if(otherhead==NULL)
		{
			otherhead=othernow;
			othertail=othernow;
		}
		else 
		{
			othertail->next=othernow;
			othertail=othernow;
		}
		
		nowAccount=head;//删除原链表中的该节点 
		if(othernow==head)
		{
			if(head->next==NULL)
			{
				head=NULL;
				tail=NULL;
			}
			else 
			{
				head=head->next;
			}
		}
		else if(head!=tail&&othernow==tail)
		{
			while(nowAccount!=NULL)
			{
				if(strcmp((nowAccount->next)->userName,tail->userName)==0)
				{
					nowAccount->next=NULL;
					tail=nowAccount;
					break;
				}
				else 
				{
					nowAccount=nowAccount->next;
				}
			}
		}
		else 
		{
			while(nowAccount!=NULL)
			{
				if(strcmp((nowAccount->next)->userName,othernow->userName)==0)
				{
					nowAccount->next=(nowAccount->next)->next;
					(nowAccount->next)->next=NULL;
					break;
				}
				else 
				{
					nowAccount=nowAccount->next;
				}
			}
		}
		othernow->next=NULL;
		char x;
		while(1)
		{
			printf("退出系统请按0,返回上一级请按1\n");
			x=getche();
			putchar('\n');
			if(x=='0')
			{
				return 0;
			}
			else if(x=='1')
			{
				return 1;
			}
			else
			{
				printf("您的输入有误，请重新输入\n");
			}
		}
	}
}

void OpenAccount()//开户
{
	//开户信息:姓名，电话，账号，密码，余额(初始化为0) 

	Account*pNewAccount=(Account*)malloc(sizeof(Account));
	pNewAccount->next=NULL;
	if(head==NULL)
	{
		head=pNewAccount;
		tail=pNewAccount;
	}
	else 
	{
		tail->next=pNewAccount;
		tail=pNewAccount;
	}
	char w;
	while(1)
	{
		system("cls");
		printf("1、开心岛银行\n");
		printf("2、取款机会咬人银行\n"); 
		printf("3、没有利润银行\n"); 
		printf("4、卷钱跑路银行\n"); 
		printf("5、不会编程银行\n");
		printf("6、啥也不会银行\n");
		printf("请选择您要进行开户的银行\n");
		w=getche();
		putchar('\n');
		switch(w)
		{
			case '1':strcpy(pNewAccount->bank,"开心岛银行");break;
			case '2':strcpy(pNewAccount->bank,"取款机会咬人银行");break;
			case '3':strcpy(pNewAccount->bank,"没有利润银行");break;
			case '4':strcpy(pNewAccount->bank,"卷钱跑路银行");break;
			case '5':strcpy(pNewAccount->bank,"不会编程银行");break;
			case '6':strcpy(pNewAccount->bank,"啥也不会银行");break;
			default :printf("您的输入有误，请重新输入\n");
					Sleep(1000);
						break;
		}
		if(w>='1'&&w<='6')
			break;  
	}
	
	printf("请输入您的姓名:\n");
	scanf("%s", pNewAccount->Name);
	
	char tel_[20];
	while(1)
	{
		printf("请输入您的电话号码\n");
		scanf("%s",tel_);
		if(strlen(tel_)==11)
		{
			strcpy(pNewAccount->tel,tel_);
			break;
		}
		else 
		{
			printf("您的输入有误，请重新输入\n");
		}
	}
	
	char Password_1[7];
	char Password_2[7];
	while(1)//设置密码 
	{
		printf("请设置您的银行卡密码\n");
		for(int i=0;i<6;i++)
		{
			Password_1[i]=getch();
			putchar('*');
		}
		putchar('\n');
		printf("请再次输入您的密码\n");
		for(int i=0;i<6;i++)
		{
			Password_2[i]=getch();
			putchar('*'); 
		}
		putchar('\n');
		if(strcmp(Password_1,Password_2)==0)
		{
			printf("密码设置成功\n");
			strcpy(pNewAccount->passWord,Password_1);
				break;
		}
		else 
		{
			printf("您两次输入的密码不一致，请重新输入\n");
		}
	}
	
	char userName_[17]={0};
	int a[17];
	srand( (unsigned)time(0) );
	for(int i=0;i<16;i++)//生成随机账号 
	{
		if(i==0)
		{
			a[i]=rand()%9+1;
			userName_[i]=a[i]+48;
			continue;
		}
		a[i]=rand()%10;
		userName_[i]=a[i]+48;
	}
	
	nowAccount=head;
	while(nowAccount!=NULL)//检查随机生成的账号是否存在
	{
		if(strcmp(nowAccount->userName,userName_)==0)
		{
			srand( (unsigned)time(0) );
			for(int i=0;i<16;i++)//重新生成随机账号 
			{
				if(i==0)
				{
					a[i]=rand()%9+1;
					userName_[i]=a[i]+48;
					continue;
				}
				a[i]=rand()%10;
				userName_[i]=a[i]+48;
			}
		}
		else 
		{
			nowAccount=nowAccount->next;
		}
	}
	strcpy(pNewAccount->userName,userName_);
	printf("您的账号为:%s\n",pNewAccount->userName);
	
	pNewAccount->money=0.00;
	
	FILE*fp=fopen(Y,"a");
	fprintf(fp,"%s\t%s\t%s\t%s\t%s\t%.2f\n",pNewAccount->Name,pNewAccount->tel,pNewAccount->userName,pNewAccount->passWord,pNewAccount->bank,pNewAccount->money);
	fclose(fp);
}

int immediate()
{
	char a;
	while(1)
	{
		printf("是否直接登录?\n");
		printf("1、是\t\t2、否\n");
		a=getche();
		putchar('\n');
		if(a=='1')
		{
			nowAccount=tail;
			return 1;
		}
		else if(a=='2')
		{
			return 0;
		}
		else
		{
			printf("您的输入有误，请重新输入\n");
		}
	}
}

void saveMoney()//存钱 
{
	float money;
	char a[100];
	char b;
	int d,c;
	FILE*fp;
	Account *paccount;
	while(1)
	{
		system("cls");
		printf("请输入您要存放的数额\n");
		scanf("%f",&money)	||	scanf("%s",a);
		d=((int)(money*1000000))%10000;
		if(d>0&&d<10000 || money==0.0)
		{
			printf("您的输入有误，请重新输入\n");
			continue;
		}
		else 
		{	
			FR*frpa=(FR*)calloc(1,sizeof(FR));
			frpa->next=NULL;
			nowAccount->money+=money;
			frpa->serbiceCharge=0.00;
			frpa->timestamp=time(0);
			frpa->balance=nowAccount->money;
			strcpy(frpa->username,nowAccount->userName);
			strcpy(frpa->type,"存款");
			frpa->money=money;
			
			if(frhead==NULL)
			{
				frhead=frpa;
				frtail=frpa;
			}
			else 
			{
				frtail->next=frpa;
				frtail=frpa;
			}
			
			fp=fopen(L,"w");
			frnow=frhead;
			while(frnow!=NULL)
			{
				fprintf(fp,"%s\t%ld\t%s\t%+.2f\t%+.2f\t%.2f\n",frnow->username,frnow->timestamp,frnow->type,frnow->money,frpa->serbiceCharge,frnow->balance);
				frnow=frnow->next;
			}
			fclose(fp);
			
			fp=fopen(Y,"w");
			paccount=head;
			while(paccount!=NULL)
			{
				fprintf(fp,"%s\t%s\t%s\t%s\t%s\t%.2f\n",paccount->Name,paccount->tel,paccount->userName,paccount->passWord,paccount->bank,paccount->money);
				paccount=paccount->next;
			}
			fclose(fp);
			break;	
		}				
	}	
}

int continueSave()
{
	char i;
	printf("交易成功，是否继续交易?\n");
	printf("1、是\t\t2、否\t\t3、退出登录\n");
	while(1)
	{
		i=getche();
		putchar('\n');
		if(i=='1')
		{
			return 1;
		} 
		else if(i=='2')
		{
			return 2;
		}
		else if(i=='3')
		{
			return 0;
		}
		else
		{
			printf("您的输入有误，请重新输入\n");
			continue;
		}
	}	
}

int Withdraw_Money()//取钱 
{
	float money;
	char b;
	int d;
	char a[100],c[100];
	FILE*fp;
	Account *paccount;
	while(1)
	{
		system("cls");
		printf("请输入您要取出的人民币金额\n");
		scanf("%f",&money)	||	scanf("%s",a);
		d=(int)(money*1000000)%10000;
		if(d>0&&d<10000 || money==0.0)
		{
			printf("您的输入有误，请重新输入\n");
			continue;
		}
		else if(strcmp(nowAccount->bank,bank)!=0)
		{
			if(nowAccount->money<(money+money*0.02)){}
			else
				break;
		}
		else 
		{
			if(nowAccount->money<money){}
			else
				break;
		}
		printf("您的余额不足");
		printf("(余额:%.2f)\n",nowAccount->money);
		printf("是否继续取钱?\n");
		printf("1、是\t\t2、否\n");			
		while(1)
		{
			b=getche();
			if(b=='1')
			{
				break;
			}
			else if(b=='2')
				return 0;
			else 
			{
				printf("你的输入有误，请重新输入\n");
				continue;
			}
		}		
	}	
			
	FR*frpa=(FR*)calloc(1,sizeof(FR));
	frpa->next=NULL;			
	if(strcmp(nowAccount->bank,bank)!=0)
	{
		nowAccount->money-=(money+money*0.02);
		frpa->serbiceCharge=0-money*0.02;				
	}		
	else 
	{
		nowAccount->money-=money;
		frpa->serbiceCharge=0.00;
	}
	frpa->balance=nowAccount->money;
	frpa->timestamp=time(NULL);
	strcpy(frpa->type,"取款");
	strcpy(frpa->username,nowAccount->userName);
	frpa->money=0-money;
	if(frhead==NULL)
	{
		frhead=frpa;
		frtail=frpa;
	}
	else
	{
		frtail->next=frpa;
		frtail=frpa;
	}
	fp=fopen(L,"w");
	frnow=frhead;
	while(frnow!=NULL)
	{
		fprintf(fp,"%s\t%ld\t%s\t%+.2f\t%+.2f\t%.2f\n",frnow->username,frnow->timestamp,frnow->type,frnow->money,frnow->serbiceCharge,frnow->balance);
		frnow=frnow->next;
	}
	fclose(fp);
	
	fp=fopen(Y,"w");
	paccount=head;
	while(paccount!=NULL)
	{
		fprintf(fp,"%s\t%s\t%s\t%s\t%s\t%.2f\n",paccount->Name,paccount->tel,paccount->userName,paccount->passWord,paccount->bank,paccount->money);
		paccount=paccount->next;
	}
	fclose(fp);
	return 1;			
}
