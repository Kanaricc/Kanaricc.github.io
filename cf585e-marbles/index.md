# [CF585E] Marbles

Monocarp has arranged n colored marbles in a row. The color of the i-th marble is ai. Monocarp likes ordered things, so he wants to rearrange marbles in such a way that all marbles of the same color form a contiguos segment (and there is only one such segment for each color).

In other words, Monocarp wants to rearrange marbles so that, for every color j, if the leftmost marble of color j is l-th in the row, and the rightmost marble of this color has position r in the row, then every marble from l to r has color j.

To achieve his goal, Monocarp can do the following operation any number of times: choose two neighbouring marbles, and swap them.

You have to calculate the minimum number of operations Monocarp has to perform to rearrange the marbles. Note that the order of segments of marbles having equal color does not matter, it is only required that, for every color, all the marbles of this color form exactly one contiguous segment.

## 分析

这道题的翻译就是“重新确定每个颜色的权值，使得逆序对最少”。

然后我就卡了，现在我也不是特别清楚为啥可以dp，只是知道能dp。

选择状态$f(S)$表示集合$S$已经全部分配权值时产生的最少逆序对。枚举下一个颜色，给予下一个（较小）权值，并计算代价。

<div>$$
f(S+\{i\})=min\{f(S)+w\}
$$</div>

接下来是w的处理。我们需要的是在当前颜色的所有点前的**已经拥有新权值**的点数目，这可以事先预处理。预处理颜色i前颜色j的pair数即可。

## 代码
不小心把ptr扔到了里层循环里T了发…

明明第一步完事都很直白了，就是非得被卡一下。

```cpp
#include <iostream>
#include <algorithm>
#include <iomanip>
#include <vector>
#include <stack>
#include <cstring>
#include <string>
using namespace std;
using ll=long long;
const int MAXN = 400010;
const int MAXC=21;
 
int nlen;
int num[MAXN];
vector<int> pos[MAXC];
ll cnt[MAXC][MAXC];
int tick=0;
int newColor[MAXN];
 
ll dp[1<<MAXC];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
 
    cin>>nlen;
    for(int i=0;i<nlen;i++){
        cin>>num[i];
    }
    memset(newColor,-1,sizeof(newColor));
    for(int i=0;i<nlen;i++){
        if(newColor[num[i]]==-1)newColor[num[i]]=tick++;
        pos[newColor[num[i]]].push_back(i);
    }
 
    for(int i=0;i<tick;i++){
        for(int j=0;j<tick;j++){
            if(i==j)continue;
            int ptr=0;
            for(int k=0;k<pos[i].size();k++){
                if(ptr<0)ptr=0;
                while(ptr<pos[j].size() && pos[j][ptr]<pos[i][k])ptr++;
                ptr--;
                if(ptr>=0 && pos[j][ptr]<pos[i][k]){
                    cnt[i][j]+=ptr+1;
                }
            }
        }
    }
 
 
    memset(dp,0x3f,sizeof(dp));
    dp[0]=0;
    for(int i=0;i<(1<<tick);i++){
        for(int j=0;j<tick;j++){
            if((i>>j)&1)continue;
            ll diff=0;
            for(int k=0;k<tick;k++){
                if((i>>k)&1)diff+=cnt[j][k];
            }
            dp[i|(1<<j)]=min(dp[i|(1<<j)],dp[i]+diff);
        }
    }
 
    cout<<dp[(1<<tick)-1]<<endl;
 
    return 0;
}
```
