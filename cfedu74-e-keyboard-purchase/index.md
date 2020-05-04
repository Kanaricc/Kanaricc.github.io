# [CFEDU74 E] Keyboard Purchase

当在使用一指禅键入一个字符串的时候，你需要不停的移动手指。比如说输入“af”，手指需要跨过3个键（包括终点）输入“a”和“f”。

针对一个经常输入的字符串，你可以定制一个长条键盘，使得在这个键盘上输入字符串需要移动的距离最短。

给出字符串长度$n$与字典大小$k$（小写字母的前$k$个），给出最优情况下手指需要移动的距离。

## 分析

这道题和之前那道marbles很像……或者说状压都挺像的。

<!--more-->

题目即依次决定键盘的下一个按键装啥，并在原字符串里计算需要移动的次数。设$f(S)$表示已经考虑了集合$S$里所有的按键。不过仔细思考一下发现这个代价需要知道先前安装的按键的顺序，而这个顺序显然不能加入到状态里，这是无法接受的。必须要通过别的方式来计算。

然后我又卡了。

这是没见过的转移方式……考虑本轮没有排入键A，那么一定就**排入了别的键B**，那么，这次对于事件“排A”的**一次delay**会导致原串里A和相邻**在键盘上已确定位置的**字母间的转移多**一个**代价。

有种模糊的感觉。之前那个题目是计算当前元素和已有元素之间的代价；这个是计算已有元素和未有元素之间的累进式代价。搞不太清楚这个$f(S)$现在代表的是什么了…不知可否有人指导。

## 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>
#include <string>
using namespace std;
using ll=long long;
const int MAXK=21;
int nlen,klen;
string inp;
 
int cost[MAXK][MAXK];
ll dp[1<<20];
int main(){
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin>>nlen>>klen;
    cin>>inp;
 
    for(int i=1;i<nlen;i++){
        cost[inp[i-1]-&#039;a&#039;][inp[i]-&#039;a&#039;]++;
        cost[inp[i]-&#039;a&#039;][inp[i-1]-&#039;a&#039;]++;
    }
 
 
    memset(dp,0x3f,sizeof(dp));
    dp[0]=0;
 
    for(int i=0;i<(1<<klen);i++){
        ll scost=0;
        for(int j=0;j<klen;j++){
            if(!((i>>j)&1))continue;
            for(int k=0;k<klen;k++){
                if((i>>k)&1)continue;
                scost+=cost[j][k];
            }
        }
        for(int j=0;j<klen;j++){
            if((i>>j)&1)continue;
            dp[i|(1<<j)]=min(dp[i|(1<<j)],dp[i]+scost);
        }
    }
 
    cout<<dp[(1<<klen)-1]<<endl;
 
    
 
    return 0;
}
```
