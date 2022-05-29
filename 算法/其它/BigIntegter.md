# 计算 2 ^ 4000

编写任意精度整数运算包。要求使用类似于多项式的运算方法。计算 2 ^ 4000 的结果，并统计 0~9的分布。

首先考虑模仿10进制整数计数方式，构造链表描述多项式 $a_nX^n + \cdots +a_1X + a_0$ 其中 $X=10$ ，首先将底数存入 基础的多项式中（具体方法不断除10去余数），随后使用for循环指数次，每次使用多项式与底数相乘，计算方法为 本位的数字×底数+进位 除 $X$ 将余数保留，商存入进位。

因为每个节点使用一个int类型存储，10进制会对空间照成巨大的浪费，同时会大量增加内循环的次数，增加时间开销。所以可以通过基数的大小来使得空间利用更加充分，并且提供运行速度。我这里采用1000作为基础进制，当让你也可以采用10000，10000000等，不过要注意进制的选择要保证进制与底数相乘不会溢出，所以要对输入的底数加以限制，例如，我才用1000作为进制，那么对于int类型，底数就要限制在 -4294967~4294967之间，否者在计算过程中就可能造成结果溢出。

下边上代码：

bigintegter.h

```c
#pragma once

#include <iostream>
#include <vector>
#include <iomanip>

using namespace std;

vector<int> BigInteger(int base, int exp);

void ShowList(const vector<int>& list);

void StatList(const vector<int>& list);
```

bigintegter.cpp

```c
#include "bigineger.h"


#define BASE	1000	//基础进制为1000进制
#define	SIZE	3		//基础长度
#define MAXBASE	4294967	//最大基数

//计算高幂次 例 2^4000
vector<int> BigInteger(int base, int exp)
{	

	
	vector<int> list(0);  //初始化用于储存计算的大数容器
	vector<int>::iterator it; //list的迭代器
	int carry;	//进位
	int flag = 1;  //符号控制

	if (base < 0 && exp % 2) {
		base *= -1;
		flag *= -1;
	}

	int tmp = base;
	//如果指数小于0 失败
	if (exp < 0 ) {
		cout << "exp < 0 !" << endl;
		return list;
	}
	//如果指数等于0 返回1
	if (exp == 0) {
		list.push_back(1);
		return list;
	}

	//如果底数过大 计算失败返回空容器
	if (base > MAXBASE) {
		cout << "Base is too large!" << endl;
		return list;
	}

	//将底数放入容器	
	while (tmp) {
		list.push_back(tmp % BASE);
		tmp /= BASE;
	}

	//计算幂次
	for (int i = 1; i < exp; i++) {
		carry = 0;

		for (it = list.begin(); it < list.end(); it++) {
			//计算当前位结果
            tmp = *it * base + carry;
			*it = tmp % BASE;
            
            //计算进位
			carry = tmp / BASE;
		}
		
        //如果最后存在进位添加新节点
		if (carry) {
			list.push_back(carry);
		}
	}

	//符号控制
	*list.rbegin() *= flag;
	
	return list;
}

//显示结果
void ShowList(const vector<int>& list)
{
	//反向迭代器
	vector<int>::const_reverse_iterator it;

	//反向输出list
	for (it = list.rbegin(); it != list.rend(); it++)
	{
		if (it != list.rbegin())
			cout << setw(3) << setfill('0');
		cout << *it;
	}
	cout << endl;
}

void StatList(const vector<int>& list)
{
	int count = 0;		//总数
	int tmp;		//中间变量
	int nums[10];	//分布
	for (int i = 0; i < 10; i++) {
		nums[i] = 0;
	}

	//反向迭代器
	vector<int>::const_reverse_iterator it;

	//反向遍历list
	for (it = list.rbegin(); it != list.rend(); it++)
	{
		tmp = *it;

		if (it == list.rbegin()) {
			while (tmp) {
				tmp = abs(tmp);
				nums[tmp % 10]++;
				count++;
				tmp /= 10;
			}
		}
		else
		{
			for (int j = 0; j < SIZE; j++) {
				nums[tmp % 10]++;
				count++;
				tmp /= 10;
			}
		}
	}

	//输出统计结果
	cout << "共有数字：" << count << "\t个。" << endl;
	for (int k = 0; k < 10; k++) {
		cout << "数字 \"" << k << "\" 共有：" << nums[k] << " 个";
		cout << "\t占比：" << nums[k] * 100.0 / count << "  %" << endl;
	}



}

```



main.c

```c
#include <iostream>
#include <vector>
#include <iomanip>

#include "bigineger.h"

using namespace std;

void BigIntegerTest();

int main(int argc, char *argv[]) 
{

	cout << "Program begin!" << endl;

	BigIntegerTest();

	cout << "Program end!" << endl;

	return 0;
}

void BigIntegerTest()
{
	//计算 2^4000
	vector<int> list(BigInteger(2,4000));

	//显示结果
	ShowList(list);
	
	//字符统计
	StatList(list);
}


```

时间复杂度分析：

假设幂次为 $N$ ，最终结果的多项式有 $M$ 个节点，则时间复杂度为 $O(NM)$ ，由于采用了1000进制故缩减了 $M$ 的大小为  $M/3$ ，虽然时间复杂仍度为 $O(NM)$ 从而提高了运行速度。



结果

```txt
Program begin!
13182040934309431001038897942365913631840191610932727690928034502417569281128344551079752123172122033140940756480716823038446817694240581281731062452512184038544674444386888956328970642771993930036586552924249514488832183389415832375620009284922608946111038578754077913265440918583125586050431647284603636490823850007826811672468900210689104488089485347192152708820119765006125944858397761874669301278745233504796586994514054435217053803732703240283400815926169348364799472716094576894007243168662568886603065832486830606125017643356469732407252874567217733694824236675323341755681839221954693820456072020253884371226826844858636194212875139566587445390068014747975813971748114770439248826688667129237954128555841874460665729630492658600179338272579110020881228767361200603478973120168893997574353727653998969223092798255701666067972698906236921628764772837915526086464389161570534616956703744840502975279094087587298968423516531626090898389351449020056851221079048966718878943309232071978575639877208621237040940126912767610658141079378758043403611425454744180577150855204937163460902512732551260539639221457005977247266676344018155647509515396711351487546062479444592779055555421362722504575706910949376
共有数字：1205  个。
数字 "0" 共有：126 个   占比：10.4564  %
数字 "1" 共有：112 个   占比：9.29461  %
数字 "2" 共有：127 个   占比：10.5394  %
数字 "3" 共有：109 个   占比：9.04564  %
数字 "4" 共有：129 个   占比：10.7054  %
数字 "5" 共有：115 个   占比：9.54357  %
数字 "6" 共有：129 个   占比：10.7054  %
数字 "7" 共有：124 个   占比：10.2905  %
数字 "8" 共有：125 个   占比：10.3734  %
数字 "9" 共有：109 个   占比：9.04564  %
Program end!
```

当让也可以考虑使用递归的方法，但是考虑到在大量使用递归会对运行栈造成巨大压力甚至为u法计算出结果，我觉得还是上述方法更为实用。