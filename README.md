#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<windows.h>
#include<conio.h>
#include<time.h>
#define Y "E:/编程/ATM机/用户信息.txt"
#define L "E:/编程/ATM机/流水信息.txt"

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

int loading();
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

FR *frhead=NULL;
FR *frtail=NULL;
FR *frnow=NULL;

const char bank[13]="啥也不会银行";

void tuxiang()
{
		system("CLS"); 
		printf("********************************\n");
		printf("**********");
		printf("欢迎来到银行");
		printf("**********\n");
		printf("********************************\n");
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
	printf("\t1、登录\t\t\t\n");
	printf("********************************\n");
	printf("\t2、开户\n");
	printf("********************************\n");
	printf("\t3、退出系统\t\t\n");
	printf("********************************\n");
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

int signin()//登录 
{
	Account x;
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
			else 
			{
				printf("请输入正确的银行卡号\n");
			}
		}
	}
	while(1)//判断密码是否正确 ,a代表密码已经错误的次数 
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
		printf("1、工商银行\n");
		printf("2、交通银行\n"); 
		printf("3、农业银行\n"); 
		printf("4、建设银行\n"); 
		printf("5、重庆农村商业银行\n");
		printf("6、孟浩东银行\n");
		printf("请选择您要进行开户的银行\n");
		w=getche();
		putchar('\n');
		switch(w)
		{
			case '1':strcpy(pNewAccount->bank,"工商银行");break;
			case '2':strcpy(pNewAccount->bank,"交通银行");break;
			case '3':strcpy(pNewAccount->bank,"农业银行");break;
			case '4':strcpy(pNewAccount->bank,"建设银行");break;
			case '5':strcpy(pNewAccount->bank,"重庆农村商业银行");break;
			case '6':strcpy(pNewAccount->bank,"孟浩东银行");break;
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


int continueTrading()
{
	char i;
	printf("交易成功,是否继续交易?\n");
	printf("1、是\t\t2、否\t\t3、退出登录\n");
	while(1)
	{
		i=getche();
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

int findOtherAccount(char userName[])
{
	char a;
	otherAccount=head;
	while(otherAccount!=NULL)
	{
		if(strcmp(otherAccount->userName,userName)==0)
		{
			return 3;
		}
		else
		{
			otherAccount=otherAccount->next;
		} 
	}
	return 0;
}

void tr()
{
	char account[100];
	int d;
	float money;
	FILE*fp;
	Account *now;
	
	while(1)
	{
		printf("请输入您要转账的金额\n");
		scanf("%f",&money)	||	scanf("%s",account);
		d=(int)(money*1000000)%10000;
		if(d>0&&d<10000 || money==0.0)
		{
			printf("您的输入有误，请重新输入\n");
			continue;
		}
		else if(strcmp(nowAccount->bank,bank)==0)
		{
			if(nowAccount->money<money)
			{
				printf("您的余额不足(余额:%.2f),请重新输入\n",nowAccount->money);
				continue;
			}
			else 
				break;
		}
		else if(strcmp(nowAccount->bank,bank)!=0)
		{
			if(nowAccount->money<(money+money*0.02))
			{
				printf("您的余额不足(余额:%.2f),请重新输入\n",nowAccount->money);
				continue;				
			}
			else 
				break;
		}
	}
	FR *frpa=(FR*)calloc(1,sizeof(FR));
	FR *frpb=(FR*)calloc(1,sizeof(FR));
	frpa->next=frpb;
	frpb->next=NULL;
	if(strcmp(nowAccount->bank,bank)==0)
	{
		nowAccount->money-=money;
		frpa->serbiceCharge=0.00;
		otherAccount->money+=money;
		frpb->serbiceCharge=0.00;
	}
	else 
	{
		nowAccount->money-=(money+money*0.02);
		frpa->serbiceCharge=-money*0.02;
		otherAccount->money+=money;
		frpb->serbiceCharge=0.00;		
	}
	frpa->balance=nowAccount->money;
	frpb->balance=otherAccount->money;
	frpa->timestamp=time(0);
	frpb->timestamp=time(NULL);
	strcpy(frpa->username,nowAccount->userName);
	strcpy(frpa->type,"转出");
	frpa->money=0-money;
	
	strcpy(frpb->username,otherAccount->userName);
	strcpy(frpb->type,"转入");
	frpb->money=money;
	
	if(frhead==NULL)
	{
		frhead=frpa;
		frtail=frpb;
	}
	else
	{
		frtail->next=frpa;
		frtail=frpb;
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
	now=head;
	while(now!=NULL)
	{
		fprintf(fp,"%s\t%s\t%s\t%s\t%s\t%.2f\n",now->Name,now->tel,now->userName,now->passWord,now->bank,now->money);
		now=now->next;
	}
	fclose(fp);
}

int Transfer()
{
	Account a;
	char account[100];
	int i,d;
	float money;
	FILE*fp;
	Account *now;
	while(1)
	{
		system("cls");
		printf("请输入对方卡号\n");
		scanf("%s",account);
		if(strlen(account)!=16)
		{
			printf("您的输入有误，请重新输入\n");
			continue; 
		}
		else 
		{
			strcpy(a.userName,account);
			if(strcmp(a.userName,nowAccount->userName)==0)
			{
				printf("您的输入有误，请重新输入\n");
				continue;
			}
			i=findOtherAccount(a.userName);
			if(i==0)
			{
				printf("您输入的卡号不存在\n");
				return 0;
			}
			else if(i==1)
			{
				break;
			}
			else if(i==2)
			{
				return 0;
			}
			else if(i==3)
			{
				break;
			} 
		}
	}
	tr();
	return 1;
}

int transf()
{
	char a;
	while(1)
	{
		printf("是否继续转账?\n");
		printf("1、是\t2、否\n");
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
		else
		{
			printf("您的输入有误，请重新输入\n");
		}
	}
}

int continueTransfer()
{
	char a;
	while(1)
	{
		printf("是否继续转账?\n");
		printf("1、继续向该用户转账\n");
		printf("2、向其他用户转账\n");
		printf("3、继续其他操作\n");
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
			return 3;
		}
		else 
		{
			printf("您的输入有误，请重新输入\n");
		}
	}
}

void PrintPipeline(char userName[])//打印流水 
{
	struct tm *t;
	printf("账号:%s\t用户名:%s\n",nowAccount->userName,nowAccount->Name);
	printf("\t    时间\t\t方式\t  金额变动\t手续费\t     余额\n");
	frnow=frhead;
	while(frnow!=NULL)
	{
		if(strcmp(frnow->username,userName)==0)
		{
			t=localtime( &(frnow->timestamp) );
			printf("%4d年%02d月%02d日\t%02d:%02d:%02d\t%s\t%+10.2f\t%+5.2f\t%10.2f\n",t->tm_year+1900,t->tm_mon+1,t->tm_mday,t->tm_hour,t->tm_min,t->tm_sec,frnow->type,frnow->money,frnow->serbiceCharge,frnow->balance);
			frnow=frnow->next;
		}
		else 
		{
			frnow=frnow->next;
		}
	}
}

void PrintInformation()//打印个人信息 
{
	system("cls"); 
	printf("姓名:%s\t卡号:%s\t电话号码:%s\t所属银行:%s\t密码:%s\t余额:%.2f\n",nowAccount->Name,nowAccount->userName,nowAccount->tel,nowAccount->bank,nowAccount->passWord,nowAccount->money);
}

int search()
{
	char a;
	char b;
	while(1)
	{
		system("CLS"); 
		printf("1、查询账户信息\n");
		printf("2、查询流水记录\n");
		printf("3、返回上一级\n");
		while(1)
		{
			a=getche();
			putchar('\n');
			if(a=='1')
			{
				PrintInformation();
				putchar('\n');
				printf("返回上一级请按1\n");
				printf("返回主菜单请按0\n");
				b=getche();
				putchar('\n');
				if(b=='1')
				{
					break;
				}
				else if(b=='0')
				{
					system("cls");
					return 1;	
				}
				else 
				{
					printf("您的输入有误，请重新输入\n");
				}
			}
			else if(a=='2')
			{
				system("CLS"); 
				PrintPipeline(nowAccount->userName);
				putchar('\n');
				printf("返回上一级请按1\n");
				printf("返回主菜单请按0\n");
				b=getche();
				putchar('\n');
				if(b=='1')
				{
					break;
				}
				else if(b=='0')
				{
					system("cls");
					return 1;	
				}
				else 
				{
					printf("您的输入有误，请重新输入\n");
				}
			}
			else if(a=='3')
			{
				system("cls");
				return 0;
			}
			else 
			{
				printf("您的输入有误，请重新输入\n");
			}
		}
	}
}

int changepassword()
{
	int i;
	long t;
	Account *paccount;
	char password[7];
	FILE*fp=fopen(Y,"r+");
	
	while(1)
	{	
		system("cls");
		printf("请输入原密码\n");
		for(i=0;i<6;i++)
		{
			password[i]=getch();
			putchar('*');
		}
		putchar('\n');
		if( strcmp(nowAccount->passWord,password)==0)
		{
			printf("请输入新密码\n");
			for(i=0;i<6;i++)
			{
				password[i]=getch();
				putchar('*');
			}
			putchar('\n');
			strcpy(nowAccount->passWord,password);
			paccount=head;
			while(paccount!=NULL)
			{
				fprintf(fp,"%s\t%s\t%s\t%s\t%s\t%.2f\n",paccount->Name,paccount->tel,paccount->userName,paccount->passWord,paccount->bank,paccount->money);
				paccount=paccount->next;
			}
			fclose(fp);
			printf("密码修改成功，请重新登录...");
			Sleep(1000);
			return 5;
		}
		else
		{
			printf("您的密码输入错误，请重新输入\n");
		}
	}

	
	
}
int menu()//菜单 
{
	printf("********************************\n");
	printf("************");
	printf("工商银行");
	printf("************\n");
	printf("********************************\n");
	printf("1、存款\t\t\t2、取款\n");
	printf("3、转账\t\t\t4、查询流水\n");
	printf("5、修改密码\t\t6、退出登录\n");
	//基本功能：1.开户、2.登录、3.中英文、4.初始化、5.退出保存、6.修改密码、7.取款，8.存款，9.转账，10.正确运行
	char i;
	char k[100];
	while(1)
	{
		printf("请选择您要办理的业务:\n");
		i=getche();
		putchar('\n');
		switch(i)
		{
			case '1':
			case '2':
			case '3':
			case '4':
			case '5':
			case '6':	return i-48;
			default :printf("您的输入有误，请重新输入\n");
						break;
		}
	}
}


int main()
{
	int i=loading();
	int a,c;
	if(i==0)
	{
		return 0;
	}
	
	while(1)
	{
		tuxiang();
		a=initialization();
		if(a==0)
			return 0;
		else if(a==1)
		{
			i=signin();
			if(i==0)
				return 0;
			else if(i==1)
				continue; 
		}
		else if(a==2)
		{
			OpenAccount();
			i=immediate();
			if(i==0)
				continue;
		}
		system("CLS"); 
		
		
		int b=menu();
		while(1)
		{
			if(b==1)//存款 
			{
				saveMoney();
				i=continueSave();
				if(i==1)
				{
					continue;
				}
				else if(i==2)
				{
					system("CLS");
					b=menu();
					continue;
				}
				else 
					break;
				
			}
			else if(b==2)//取款 
			{
				if(nowAccount->money==0.00)
				{
					printf("您的账户余额为0.00,不能进行取款操作\n");
					Sleep(2000);
					system("CLS");
					b=menu();
					continue;					
				}
				i=Withdraw_Money();
				if(i==0)
				{
					system("CLS");
					b=menu();
					continue;
				}
				else if(i==1)
				{
					c=continueTrading();
					if(c==1)
						continue;
					else if(c==2)
					{
						system("CLS"); 
						b=menu();
						continue;
					}
					else 
						break;
					
				}
			}
			else if(b==3)//转账 
			{
				if(nowAccount->money==0.00)
				{
					printf("您的账户余额为0.00,不能进行转账操作\n");
					Sleep(2000);
					system("CLS");
					b=menu();
					continue;
				}
				i=Transfer();
				if(i==0)
				{
					a=transf();
					if(a==1)
						continue;
					else
					{
						system("cls");
						b=menu();
						continue;
					}
				}
				else if(i==1)
				{

					i=continueTransfer();
					while(i==1)
					{
						system("cls");
						tr();
						i=continueTransfer();
					}
					if(i==2)
					{
						continue;
					}
					else if(i==3)
					{
						system("cls");
						b=menu();
						continue;
					}
				}
			}
			else if(b==4)//查询 
			{
				search(); 
				b=menu();
				continue;
			}
			else if(b==5)//修改密码 
			{
				i=changepassword();
				if(i==5)
				{
					break;
				}
			}
			else if(b==6)//退出登录 
			{
				break;
			}
		}
	}
}
 
