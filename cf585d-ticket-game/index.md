# [CF585D] Ticket Game

Monocarp and Bicarp live in Berland, where every bus ticket consists of n digits (n is an even number). During the evening walk Monocarp and Bicarp found a ticket where some of the digits have been erased. The number of digits that have been erased is even.

Monocarp and Bicarp have decided to play a game with this ticket. Monocarp hates happy tickets, while Bicarp collects them. A ticket is considered happy if the sum of the first $\frac n2$ digits of this ticket is equal to the sum of the last $\frac n2$ digits.

Monocarp and Bicarp take turns (and Monocarp performs the first of them). During each turn, the current player must replace any erased digit with any digit from 0 to 9. The game ends when there are no erased digits in the ticket.

If the ticket is happy after all erased digits are replaced with decimal digits, then Bicarp wins. Otherwise, Monocarp wins. You have to determine who will win if both players play optimally.

## 分析

在做这道题的时候，我发现了实际上影响答案的只和两边的 已知数的和 以及未知位的数目有关，发现了获胜的根本是前后两部分和的差值大小，发现了先手的目的是**扩大差值**使得后手无法通过某种方式追赶。

但是到这我就卡住了。淦，为什么我这么菜。

接下来要考虑的是**优势**，在这道题里优势就是差值。先手的目的是扩大优势，在大的一边增大会加大优势，而在小的一边置0不会扩大优势；后手要**减小优势**，同样的办法就是在小的一侧置数来追赶另一侧，在大的一侧置0不会更好。

那么，又可以得出先手扩大优势最好的办法是置9（最大）。那么游戏最开始的几步，就是双方在两侧填写9。直到某侧不再有空位。

此时，如果空位在大的一侧，先手必赢。如果空位在小的一侧，继续讨论如下

<div>$$
x_1+x_2+\cdots+x_n=C
$$</div>

首先$2|n$显然。

当C不是9的倍数时，先手总有办法让最终结果偏离。因此该情况只有当$\frac n2 \times 9 = n$时后手才能获胜。

## 代码

嫖了队友的…我瞎xx搞的结论WA23了…

```cpp
#include<bits/stdc++.h>
using namespace std;
char s[200005];
int main()
{
	int n;
	scanf("%d",&n);
	scanf("%s",s+1);
	int cnt=0;
	int sum=0;
	for(int i=1;i<=n/2;i++)
	{
		if(s[i]==&#039;?&#039;) cnt++;
		else sum+=s[i]-&#039;0&#039;;
	}
	for(int i=n/2+1;i<=n;i++)
	{
		if(s[i]==&#039;?&#039;) cnt--;
		else sum-=s[i]-&#039;0&#039;;
	}
	//cout<<cnt<<" "<<sum<<endl;
	if(sum*2==-cnt*9) puts("Bicarp");
	else puts("Monocarp");
}
```
