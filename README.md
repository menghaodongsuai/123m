# 123m
atm机
#include<iostream>
#include<fstream>
#include<string>
#include<sstream>
#include<iomanip>
#include<vector>
#include<algorithm>
#include<Windows.h>
using namespace std;
class User;
//静态变量
static bool login = false;
//客户信息容器的创建
vector<User> Usr;
vector<User>::iterator it = Usr.begin();
//创建类
class Assist
{
public:
	void Save(User u);//存储数据到本地
	bool JudgeID(string s);//判断身份证号输入是否合法
	bool JudgePass(string s);//判断密码输入是否合法
	bool IDIsReg(string s);//判断身份证号是否已经被注册
	bool JudgeAcc(string s);//判断帐号输入是否合法
	bool AccIsReg(string s);//判断帐号是否已经被注册
};
class User
{
	friend class Assist;
	friend User Login(Assist a);
	friend void Change();
	friend string itos(int i);
public:
	User();
	void Read(string a, string b, string c, string d, int e, string f);//从本地读取用户信息
	void Register(Assist a);//开户
	void ShowInform();//显示客户信息
	void ShowBalance();//查询余额
	void Transfer();//转账
	void TakeCash();//取款
	void SaveCash();//存款
	void ChangePass(Assist a);//修改密码
private:
	string acc;//账户
	string name;//姓名
	string ID;//身份证
	string pass;//密码
	string record;//存取款记录
	int money;//余额
};
//全局函数:
void Change()//余额、密码改变后存到本地
{
	fstream os;
	os.open("D:\\UserList.txt", ios::out);
	for (vector<User>::iterator out = Usr.begin(); out != Usr.end(); out++)
	{
		os << out->name << " " << out->ID << " " << out->acc << " " << out->pass << " " << out->money <<" "<< out->record << endl;
	}
	os.close();
}
User Login(Assist a)//登录
{
	User u;
	ifstream input, judgein;
	input.open("D:\\UserList.txt", ios::in);
	judgein.open("D:\\Frozen.txt", ios::in);
	cout << "----------------------------插卡:请输入以下信息--------------------------" << endl;
	string InAcc, InPass;
	string temp;
	string read;
	cout << setiosflags(ios::left);
	cout << setw(52) << "请输入您的帐号:"; cin >> InAcc;
	while (getline(input, read))//将本地的数据读取到Usr容器中
	{
		string name, id, acc, pass, num, record;
		stringstream strin(read);
		strin >> name >> id >> acc >> pass >> num>> record;
		int money = stoi(num);
		u.Read(name, id, acc, pass, money, record);
		Usr.push_back(u);
	}
	while (getline(judgein, temp))
	{
		if (temp == InAcc)
		{
			cout << "您的账户已被锁定，无法登录" << endl << "即将返回主界面" << endl;
			Sleep(1000);
			login = false;
			it = Usr.begin();
			return *it;
		}
	}
	for (it = Usr.begin(); it != Usr.end(); it++)//遍历容器找出此帐号
	{
		if ((*it).acc == InAcc)
		{
			cout << setw(52) << "请输入您的密码:"; cin >> InPass;
			if ((*it).pass == InPass)
			{
				cout << "登陆成功" << endl;
				login = true;
				return *it;
			}
			else
			{
				cout << setw(52) << "密码错误!您还有2次输入机会:"; cin >> InPass;
				if ((*it).pass == InPass)
				{
					cout << "登陆成功" << endl;
					login = true;
					return *it;
				}
				else
				{
					cout << setw(52) << "密码错误!您还有1次输入机会:"; cin >> InPass;
					if ((*it).pass == InPass)
					{
						cout << "登陆成功" << endl;
						login = true;
						return *it;
					}
					else
					{
						cout << "您已输错三次密码，您的账户已被锁定" << endl << "即将返回主页面" << endl;
						Sleep(1000);
						ofstream output;
						output.open("D:\\Frozen.txt", ios::app);
						output << InAcc << endl;
						login = false;
						return *it;
					}
				}
			}
		}
	}
	cout << "!错误!不存在此帐号" << endl << "即将返回主页面" << endl;
	Sleep(1000);
	input.close();
	it = Usr.begin();
	return *it;
}
string itos(int i)//将整形转为字符串
{
	stringstream s;
	s << i;
	return s.str();
}
//类Assist的成员函数:
bool Assist::JudgeID(string s)
{
	int t = 0;
	for (int i = 0; i < s.length()-1; i++)
	{
		if (s[i] >= '0' && s[i] <= '9')
		{
			t++;
		}
	}
	if ((s[s.length()-1] >= '0' && s[s.length()-1] <= '9') || s[s.length()-1] == 'X')//考虑到身份证可能X结尾
	{
		t++;
	}
	if (t == 18) { return true; }
	else { return false; }
}
bool Assist::JudgeAcc(string s)
{
	if (s.length() != 19) { return false; }
	else
	{
		for (int i = 0; i < 19; i++)
		{
			if (s[i] < '0' || s[i]>'9') { return false; }
		}
		return true;
	}
}
bool Assist::JudgePass(string s)
{
	int t=0;
	for (int i = 0; i < s.length(); i++)
	{
		if (s[i] >= '0' && s[i] <= '9')
		{
			t++;
		}
	}
	if (t == 6) { return true; }
	else { return false; }
}
bool Assist::IDIsReg(string s)
{
	ifstream input;
	input.open("D:\\UserList.txt", ios::in);
	string temp;
	while (getline(input, temp))
	{
		stringstream ss(temp);
		string temp1, temp2, temp3;
		ss >> temp1 >> temp2 >> temp3;
		if (temp2 == s)
		{
			return false;
		}
	}
	return true;
}
bool Assist::AccIsReg(string s)
{
	{
		ifstream input;
		input.open("D:\\UserList.txt", ios::in);
		string temp;
		while (getline(input, temp))
		{
			stringstream ss(temp);
			string temp1, temp2, temp3;
			ss >> temp1 >> temp2 >> temp3;
			if (temp3 == s)
			{
				return false;
			}
		}
		return true;
	}
}
void Assist::Save(User u)
{
	fstream outFile;
	outFile.open("D:\\UserList.txt",ios::app);
	outFile << u.name<<" "<< u.ID <<" "<< u.acc << " " << u.pass << " " << u.money << endl;
	outFile.close();
}
//类User的成员函数:
void User::Register(Assist a)
{
	string InAcc, InName, InID, InPass1, InPass2;
	cout << "----------------------------欢迎使用开户功能----------------------------" << endl;
	cout << setiosflags(ios::left);
	cout << "请按提示输入信息" << endl;
	cout << setw(52) << "请输入您的姓名:";
	cin >> InName;
	name = InName;
	while (1)//输入身份证号
	{
		cout << setw(52) << "请输入您的身份证号(18位，若有X请使用大写):";
		while (1)
		{
			cin >> InID;
			if (a.JudgeID(InID))
			{
				break;
			}
			else
			{
				cout << setw(52) << "!错误！请输入合法的身份证号(18位，若有X请使用大写)：";
			}
		}
		if (a.IDIsReg(InID))//确保用户提供的身份证未被使用
		{
			ID = InID;
			break;
		}
		else
		{
			cout << "!错误!此身份证已被注册" << endl;
		}
	}
	cout << setw(52) << "请输入您的帐号，为19位数字";
	while (1)
	{
		cin >> InAcc;
		if (!a.JudgeAcc(InAcc))
		{
			cout << "!错误!请确保账户为19位数字";
		}
		else
		{
			if (a.AccIsReg(InAcc))
			{
				acc = InAcc;
				break;
			}
			else
			{
				cout << setw(52) << "!错误!此帐号已经被注册,请重新输入:";
			}
		}
	}
	Flag2:
	cout << setw(52) << "请输入您的密码，为6位数字:";
	while (1)//设定密码
	{
		cin >> InPass1;
		if (a.JudgePass(InPass1))
		{
			cout << setw(52) << "请再次输入您的密码:";
			while (1)
			{
				cin >> InPass2;
				if (a.JudgePass(InPass2) && InPass2 == InPass1)
				{
					pass = InPass2;
					break;
				}
				else
				{
					cout << setw(52) << "！错误!请检查:1.密码是否为6位数字。2.两次输入密码是否相同。" << endl;
					cout << setw(52) << "请再次输入您的密码:" << endl;
					goto Flag2;
				}
			}
			break;
		}
		else
		{
			cout << setw(52) << "!错误!请输入格式符合要求的6位数字密码:";
		}
	}
	cout << "开户成功!您提供的信息如下:" << endl;
	this->ShowInform();
	a.Save(*this);
	cout << "您的信息已被保存至本地" << endl << "即将返回主菜单" << endl;
	Sleep(1000);
}
User::User()
{
	money = 10000;
}
void User::Read(string a, string b, string c, string d, int e,string f)
{
	name = a;
	ID = b;
	acc = c;
	pass = d;
	money = e;
	record = f;
}
void User::ShowInform()
{
	cout << setiosflags(ios::left) << endl;
	cout << setw(10) << "姓名:" <<  name << endl;
	cout << setw(10) << "身份证号:" << ID << endl;
	cout << setw(10) << "账户:" << acc << endl;
}
void User::ShowBalance()
{
	cout << "---------------------------------余额查询--------------------------------" << endl;
	cout << "您目前的余额为:" << this->money << endl << "即将返回主菜单"<<endl;
	Sleep(2000);
}
void User::TakeCash()
{
	ofstream ofr;
	ofr.open("D:\\UserList.txt", ios::out);
	cout << "------------------------------------取款-----------------------------------" << endl;
	int num = 0;
	while (1)
	{
		cout << setw(52) << "请输入您的取款金额(您单次可取款100000元):"; 
		cin >> num;
		if (num < 100 || num % 100 != 0)
		{
			cout << "!错误!取款额只能为100的整数倍" << endl;
		}
		else
		{
			if ((money - num) < 0)
			{
				cout << "余额不足!" << endl;
			}
			else
			{
				if (num > 100000)
				{
					cout << "您一次只可取款100000元" << endl;
				}
				else
				{
					money -= num;
					it->money -= num;
					string temp1 = itos(num);
					string temp2 = "-" + temp1;
					it->record += temp2;
					cout << "取款成功!" << endl << "您当前的余额为:" << this->money << "元" << endl;
					cout << "即将返回主页面";
					ofr.close();
					Sleep(1000);
					break;
				}
			}
		}
	}
}
void User::SaveCash()
{
	ofstream ofr;
	ofr.open("D:\\UserList.txt", ios::out);
	int num;
	cout << "------------------------------------存款-----------------------------------" << endl;
	while (1)
	{
		cout << setw(52) << "请输入您的存款金额(您单次可存款100000元):"; cin >> num;
		if (num < 100 || num % 100 != 0)
		{
			cout << "!错误!存款额只能为100的整数倍" << endl;
		}
		else if (num > 100000)
		{
			cout << "您一次只可存款100000元" << endl;
		}
		else
		{
			money += num;
			it->money += num;
			string temp1 = itos(num);
			string temp2 = "+" + temp1;
			it->record += temp2;
			cout << "存款成功!" << endl << "您当前的余额为:" << this->money << "元" << endl;
			cout << "即将返回主页面";
			Sleep(1000);
			ofr.close();
			break;
		}
	}
}
void User::Transfer()
{
	string InAcc1, InAcc2, temp; int num; bool out = false;
	cout << "------------------------------------转账-----------------------------------" << endl;
	ifstream input;
	input.open("D:\\UserList.txt", ios::in);
	cout << setw(52)<<"请输入需要转入的帐号:";
	cin >> InAcc1;
	for (it=Usr.begin(); it != Usr.end(); it++)
	{
		if (this->acc == InAcc1)
		{
			out = true;
			cout << "!错误!您不可以给自己转账" << endl << "即将返回主页面";
			Sleep(1000);
			break;
		}
		else
		{
			if (it->acc == InAcc1)
			{
				out = true;
				cout << setw(52) << "请再次输入需要转入的帐号:";
				while (1)
				{
					cin >> InAcc2;
					if (InAcc1 == InAcc2)
					{
						cout << setw(52) << "请输入您的转账金额(您单次可转账100000元):"; cin >> num;
						if ((money - num) < 0)
						{
							cout << "余额不足!" << endl;
							goto Flag;
						}
						else if (num > 100000)
						{
							cout << "您一次只可取款100000元" << endl;
							goto Flag;
						}
						else
						{
							money -= num;
							it->money += num;
							string temp1 = itos(num);
							it->record += "+" + temp1;
							cout << "转账成功!" << endl << "您当前的余额为:" << this->money << "元" << endl;
							cout << "即将返回主页面";
							Sleep(1000);
							for (it = Usr.begin(); it != Usr.end(); it++)
							{
								if (it->acc == this->acc)
								{
									it->money -= num;
									it->record = it->record+"-" + temp1;
								}
							}
							goto Flag;
						}
					}
					else
					{
						cout << setw(52) << "!错误!请检查两次输入帐号是否一致";
					}
				}
			}
		}
	}
	Flag:
	if (out == false)
	{
		cout << "没有此帐号" << endl << "即将返回主界面" << endl;
		Sleep(1000);
	}
}
void User::ChangePass(Assist a)
{
	string cPass1,cPass2;
	cout << setiosflags(ios::left);
	cout << "----------------------------------修改密码---------------------------------" << endl;
	cout << setw(52) << "请输入修改后的密码,为6位数字:";
	while (1)
	{
		cin >> cPass1;
		if (a.JudgePass(cPass1))
		{
			cout << setw(52) << "请再次输入您的密码:";
			while (1)
			{
				cin >> cPass2;
				if (a.JudgePass(cPass2) && cPass2== cPass1)
				{
					pass = cPass1;
					it->pass = cPass1;
					break;
				}
				else
				{
					cout << setw(52) << "！错误!请检查:1.密码是否为6位数字。2.两次输入密码是否相同。" << endl;
					cout << setw(52) << "请再次输入您的密码:";
				}
			}
			break;
		}
		else
		{
			cout << setw(52) << "!错误!请输入格式符合要求的6位数字密码:";
		}
	}
	cout << "修改成功" << endl << "即将返回主页面" << endl;
	Sleep(1000);
}
//主函数、主体功能
int main()
{
	User usr;
	Assist a;
	string c, d, e;
	while (1)
	{
		cout << setiosflags(ios::left);
		cout << "-----------------欢迎使用自助ATM机，请选择您的服务--------------------" << endl;
		cout << "    " << "1.开户" << endl
			<< "    " << "2.插卡" << endl
			<< "    " << "3.结束服务" << endl;
		cout << setw(52)<<"请在此输入您的选择:";
		cin >> d;
		if (d != "1" && d != "2" && d != "3")//输入异常处理
		{
			cout << "请输入有效选择";
			Sleep(1000);
			system("cls");
		}
		else
		{
			system("cls");
			if (d == "1")
			{
				usr.Register(a);
				system("cls");
			}
			else if (d == "2")
			{
				User ulogin = Login(a);
				system("cls");
				if (login)
				{
					while (1)
					{
						cout << "--------------欢迎使用自助ATM机，请在下列菜单中选择您的服务----------" << endl;
						cout << "    " << "1.查询余额" << endl
							<< "    " << "2.存取款" << endl
							<< "    " << "3.转账" << endl
							<< "    " << "4.修改密码" << endl
							<< "    " << "5.退卡" << endl
							<< setw(52)<<"请在此输入您的选择:";
						cin >> c;
						system("cls");
						if (c != "1" && c != "2" && c != "3" && c != "4"&&c!="5")
						{
							cout << "请输入有效选择";
							Sleep(1000);
							system("cls");
						}
						if (c == "1")
						{
							ulogin.ShowBalance();
							system("cls");
						}
						else if (c == "2")
						{
                            cout << "----------------------------------存取款----------------------------------" << endl;
							cout << "请选择:" << endl;
							cout << "    1.存款" << endl << "    2.取款" << endl;
							cout << setw(52) << "请输入您的选择:"; cin >> e;
							if (e != "1" && e != "2")
							{
								cout << "请输入有效选择";
								Sleep(1000);
								system("cls");
							}
							else if (e == "1")
							{
								system("cls");
								ulogin.SaveCash();
								Change();
							}
							else
							{
								system("cls");
								ulogin.TakeCash();
								Change();
							}
							Change();
							system("cls");
						}
						else if (c == "3")
						{
							ulogin.Transfer();
							Change();
							system("cls");
						}
						else if (c == "4")
						{
							ulogin.ChangePass(a);
							Change();
							system("cls");
						}
						else if (c == "5")
						{
							cout << "退卡成功" << endl;
							break;
						}
					}
				}
			}
			else if (d == "3")
			{
				system("cls");
				cout << "感谢您的使用" << endl;
				break;
			}
		}
	}
	return 0;
}
