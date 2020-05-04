# [CCPC2017杭州] Heathrock

这题意太长了，我懒得复制。

## 分析

额……其实就是爆搜模拟。但是状态比较多，需要记双方生命值，3个随从的生命，当前场数，魔力，体力，以及一些玄学优化。

* 自选目标的牌一定会放在回合的较后出，因为其不会累加魔力。
* 如果剩下的牌全踢脸也踢不死，直接返回。

之后就能跑得飞快。

<!--more-->

{% asset_img "1.png" %}


## 代码

一开始输出了"YES"，淦。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include <cstring>
#include <stack>
#include <queue>
using namespace std;
const int MAXN=10;
struct Card{
    int type;
    int damage;
    int cost;
    int power;

    Card(){}
    Card(int type,int damage,int cost,int power):type(type),damage(damage),cost(cost),power(power){}
} cards[MAXN];
bool used[MAXN];
int nlen;
int md1,md2,md3;
int mx1,mx2,mx3;

stack<int> ans_cards,ans_target;
bool dfs(int stone,int power,int bl_me,int bl_en,int m1,int m2,int m3,int depth,bool nearend=false){
    if(stone<0)return false;
    if(depth>=3)return false;
    if(bl_en<=0)return true;

    if(depth>=1 && stone==10){
        if(m1>0)bl_me-=md1;
        if(m2>0)bl_me-=md2;
        if(m3>0)bl_me-=md3;

        if(m1<=0)m1=mx1;
        if(m2<=0)m2=mx2;
        if(m3<=0)m3=mx3;
    }
    if(bl_me<=0)return false;

    //full attacking to owner cannot kill it.
    int fullattack=0;
    int fakepower=power;
    int unusd_lian=0;
    for(int i=0;i<nlen;i++){
        if(used[i])continue;
        fakepower+=cards[i].power;
        if(cards[i].type==1)continue;
        unusd_lian++;
        fullattack+=cards[i].damage;
    }
    if(fullattack+fakepower*(unusd_lian)<bl_en)return false;
    

    for(int i=0;i<nlen;i++){
        if(used[i])continue;
        const Card &card=cards[i];
        int reald=card.damage+power;

        if(card.cost>stone)continue;

        used[i]=1;
        if(!nearend && card.type==1){
            if(stone-card.cost>=0 && dfs(stone-card.cost,power+card.power,bl_me,bl_en,m1-reald,m2-reald,m3-reald,depth)){
                ans_cards.push(i+1);
                ans_target.push(-1);
                return true;
            }
        }
        if(!nearend && card.type==2){
            if(stone-card.cost>=0 && dfs(stone-card.cost,power+card.power,bl_me,bl_en-reald,m1,m2,m3,depth)){
                ans_cards.push(i+1);
                ans_target.push(-1);
                return true;
            }
        }
        if(card.type==3){
            if(stone-card.cost>=0 && dfs(stone-card.cost,power+card.power,bl_me,bl_en-reald,m1,m2,m3,depth,1)){
                ans_cards.push(i+1);
                ans_target.push(0);
                return true;
            }

            if(m1>0 && stone-card.cost>=0 && dfs(stone-card.cost,power+card.power,bl_me,bl_en,m1-reald,m2,m3,depth,1)){
                ans_cards.push(i+1);
                ans_target.push(1);
                return true;
            }

            if(m2>0 && stone-card.cost>=0 && dfs(stone-card.cost,power+card.power,bl_me,bl_en,m1,m2-reald,m3,depth,1)){
                ans_cards.push(i+1);
                ans_target.push(2);
                return true;
            }

            if(m3>0 && stone-card.cost>=0 && dfs(stone-card.cost,power+card.power,bl_me,bl_en,m1,m2,m3-reald,depth,1)){
                ans_cards.push(i+1);
                ans_target.push(3);
                return true;
            }
        }
        used[i]=0;
    }

    if(dfs(10,0,bl_me,bl_en,m1,m2,m3,depth+1)){
        ans_cards.push(-1);
        ans_target.push(-1);
        return true;
    }

    return false;
}
int mc1,mc2,mc3;
int bl_me,bl_en;
int main(){
    ios::sync_with_stdio(false);
    cin.tie(0);cout.tie(0);
    int kase;cin>>kase;
    while(kase--){
        memset(used,0,sizeof(used));
        while(!ans_cards.empty())ans_cards.pop();
        while(!ans_target.empty())ans_target.pop();

        cin>>nlen;
        cin>>bl_me>>bl_en;

        cin>>md1>>mc1>>mx1;
        cin>>md2>>mc2>>mx2;
        cin>>md3>>mc3>>mx3;
        for(int i=0;i<nlen;i++){
            cards[i].cost=cards[i].damage=cards[i].power=cards[i].type=0;
            int type;cin>>type;
            if(type==1 || type==2){
                int cost,x,y;
                cin>>cost>>x>>y;
                cards[i].type=type;
                cards[i].cost=cost;
                cards[i].damage=x;
                cards[i].power=y;
            }else{
                int cost,x;
                cin>>cost>>x;
                cards[i].type=type;
                cards[i].cost=cost;
                cards[i].damage=x;
            }
        }
        ans_cards.push(-1);
        ans_target.push(-1);
        if(dfs(10,0,bl_me,bl_en,mc1,mc2,mc3,0)){
            cout<<"Yes\n";
        }else{
            cout<<"No\n";
            continue;
        }
        
        int block=0;
        while(!ans_cards.empty()){
            queue<int> pack_ans,pack_target;
            while(!ans_cards.empty() && ans_cards.top()!=-1){
                pack_ans.push(ans_cards.top());
                ans_cards.pop();
                pack_target.push(ans_target.top());
                ans_target.pop();
            }

            if(!ans_cards.empty()){
                ans_cards.pop();
                ans_target.pop();
            }

            if(pack_ans.empty())continue;

            cout<<pack_ans.size()<<"\n";
            while(!pack_ans.empty()){
                cout<<pack_ans.front()<<" ";
                pack_ans.pop();
            }
            cout<<"\n";
            while(!pack_target.empty()){
                cout<<pack_target.front()<<" ";
                pack_target.pop();
            }
            cout<<"\n";
            block++;            
        }
        while(block<3){
            cout<<"0\n";
            block++;
        }
    }

    return 0;
}
```
