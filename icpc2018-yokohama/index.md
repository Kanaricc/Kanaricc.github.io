# ICPC2018 Yokohama


## C. Emergency Evacuation

当没有人碰在一起时，耗时就是离门最远的人需要的耗时。当有人碰在一起时，才会发生其他情况。

人碰在一起的条件是他们距离门的距离一样，可以让人反着跑看出来。当碰在一起时，必定会有一个人等待一秒。而这个等的人到底是谁并没有区别，原本会在下一秒到达该格子的人总是会被等待的人卡住。然后就你卡我，我卡他，他再卡后面的人。

将逃跑需要时间相同的人往后匀。时间的值域不大所以直接用桶。

## D. Shortest Common Non-Subsequence

> 补

先考虑怎么构造一个字符串的最小非子序列。用类似于序列自动机的方法构造。

为字符串添加起始节点，令$f(i,x)$表示在该自动机上第i状态填x得到的最短非子序列长度。转移是
<div>$$
f(i,x)=\min \{f(\text{next}(i,x),y)|y\}+1
$$</div>
其中$\text{next}(i,x)$为第i位后最近的x的位置。只要最短长度时，扔掉x也无所谓
<div>$$
f(i)=min\{f(\text{next}(i,x))|x\}+1
$$</div>
初始状态为$f(n+1)=0$。

改造为该题，设状态$f(i,j)$为在俩自动机上跑到$i,j$状态的最短XX长度，有
<div>$$
f(i,j)=\min\{f(n_1(i,x),n_2(j,x))|x\}+1
$$</div>
初始状态有$f(n_1+1,n_2+1)=0$。

----

感觉有一个值得思考的问题是最长公共子串的写法…毕竟转移方程看起来几乎一样来着…
<div>$$
f(i,j)=\max\{f(n_1(i,x),n_2(j,x))|x\}+1
$$</div>
那重点就只能在于初始状态了。

看起来应该是$f(n_1+1,n_2+1)=-\infty,f(n_1, * )=f( * ,n_2)=0$，主要考虑的

* 不能转移到不存在n+1状态，这意味着填写的字母不存在。
* n状态无字符，0。

## E. Eulerian Flight Tour

……

看起来是把边选/不选对度造成的影响抽象成方程组来解。复杂度看起来没问题，估计解完还要再处理一堆东西…回头再说。

## K. Sixth Sense

算最大得分可以贪心。对方和自己的牌升序排序后，拿正好能对抗的牌去和对方牌一一打。

剩下的是怎么求字典序最大的答案。能够发现，当当前这张牌打得太大时，就可能导致后续某一轮牌输掉，让总比分减少。这看起来可以二分。不过写着写着把样例3给wa了。

之后意识到这个东西并不是整个区间单调，需要在得分和输分的两个出牌区间分别做两次。

```cpp
#include <cstdio>
#include <cmath>
#include <algorithm>
#include <unordered_map>
#include <vector>
#include <map>
#include <cmath>
#include <iostream>
using namespace std;
const int XN=5050;

int my[XN],you[XN];
int myrnk[XN],yournk[XN];
int used[XN];
int n;
int maxans=0;

//从第n轮开始
int check(int round){
    int ptr=1;
    while(ptr<=n && yournk[ptr]<round)ptr++;
    if(ptr>n)return 0;
    int res=0;
    for(int i=1;i<=n;i++){
        if(used[i])continue;
        if(ptr>n)break;
        if(my[myrnk[i]]>you[yournk[ptr]]){
            res++;
            ptr++;
            while(ptr<=n &&yournk[ptr]<round)ptr++;
        }
    }
    return res;
}

vector<int> rest;
int main(){
    ios::sync_with_stdio(false);cin.tie(0);cout.tie(0);
    cin>>n;
    for(int i=1;i<=n;i++){
        cin>>you[i];
    }
    for(int i=1;i<=n;i++){
        cin>>my[i];
    }
    for(int i=1;i<=n;i++)myrnk[i]=yournk[i]=i;
    sort(myrnk+1,myrnk+1+n,[](auto a,auto b){
        return my[a]<my[b];
    });
    sort(yournk+1,yournk+1+n,[](auto a,auto b){
        return you[a]<you[b];
    });
    for(int i=1;i<=n;i++)rest.push_back(i);

    maxans=check(1);
    int app=0;
    for(int i=1;i<=n;i++){
        //赢的
        {
            int l=0,r=rest.size();
            while(l<r && my[myrnk[rest[l]]]<=you[i])l++;
            while(l+1<r){
                int mid=(l+r)/2;
                int chosen=rest[mid];
                used[chosen]=1;
                if(app+1+check(i+1)>=maxans)l=mid;
                else r=mid;
                used[chosen]=0;
            }
            if(l<r){
                used[rest[l]]=1;
                if(app+1+check(i+1)>=maxans){
                    cout<<my[myrnk[rest[l]]]<<" ";
                    rest.erase(rest.begin()+l);
                    app++;
                    continue;
                }else{
                    used[rest[l]]=0;
                }
            }
        }
        {
            int l=0,r=(int)rest.size()-1;
            while(l<r && my[myrnk[rest[r]]]>you[i])r--;
            r++;
            while(l+1<r){
                int mid=(l+r)/2;
                int chosen=rest[mid];
                used[chosen]=1;
                if(app+check(i+1)>=maxans)l=mid;
                else r=mid;
                used[chosen]=0;
            }
            used[rest[l]]=1;
            cout<<my[myrnk[rest[l]]]<<" ";
            rest.erase(rest.begin()+l);
        }

    }
    cout<<endl;
    return 0;
}
```

## Extra

在群里看到一道题，在$1 \leq x \leq m$的n个数中找到一个字典序最小的序列且为1-m的一个排列。

最开始想维护每个数字出现的次数，然后从第一位开始向后扫，当前数字正好是还缺少的最小数字或者此后再也没有该数字出现的话就输出。后面的条件是对的，的确当后方没有数字时，就不得不在该位输出，但是前面的条件有些问题。如果一个更大的数字结束的比小数字更快，且下一个缺少的最小数字并非“小数字”，就会翻车。此时的情况最优应该是以不得不输出的位置开始，划分为两个问题，前面尽可能找字典序小的一段。

维护单调栈，当新加入的数字比前一个更优（字典序小）时，如果前一个数字在后面还有，就可以把它踢掉，否则前面的答案就定了，输出答案后再从该位重新开始。

```cpp
#include<cstdio>
#include<cmath>
#include<algorithm>
#include<unordered_map>
#include <vector>
#include <map>
#include <cmath>
#include <iostream>
#include <string>
#include <deque>
using namespace std;
const int XN=30;

int cnt[XN];
bool used[XN];
deque<char> q;
int main(){
    string inp;cin>>inp;
    for(auto chr:inp)cnt[chr-'a']++;
    for(auto chr:inp){
        cnt[chr-'a']--;
        if(used[chr-'a'])continue;
        while(!q.empty() && q.back()>chr && cnt[q.back()-'a']>0){
            used[q.back()-'a']=0;
            q.pop_back();
        }
        q.push_back(chr);
        used[chr-'a']=1;
        if(cnt[chr-'a']==0){
            while(!q.empty()){
                cout<<q.front();
                q.pop_front();
            }
        }
    }
    while(!q.empty()){
        cout<<q.front();
        q.pop_front();
    }

    cout<<endl;

    return 0;
}
```


