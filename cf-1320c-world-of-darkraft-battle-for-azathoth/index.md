# [CF 1320C] World of Darkraft: Battle for Azathoth

Roma is playing a new expansion for his favorite game World of Darkraft. He made a new character and is going for his first grind.

Roma has a choice to buy exactly one of 𝑛 different weapons and exactly one of 𝑚 different armor sets. Weapon 𝑖 has attack modifier 𝑎𝑖 and is worth 𝑐𝑎𝑖 coins, and armor set 𝑗 has defense modifier 𝑏𝑗 and is worth 𝑐𝑏𝑗 coins.

After choosing his equipment Roma can proceed to defeat some monsters. There are 𝑝 monsters he can try to defeat. Monster 𝑘 has defense 𝑥𝑘, attack 𝑦𝑘 and possesses 𝑧𝑘 coins. Roma can defeat a monster if his weapon's attack modifier is larger than the monster's defense, and his armor set's defense modifier is larger than the monster's attack. That is, a monster 𝑘 can be defeated with a weapon 𝑖 and an armor set 𝑗 if 𝑎𝑖>𝑥𝑘 and 𝑏𝑗>𝑦𝑘. After defeating the monster, Roma takes all the coins from them. During the grind, Roma can defeat as many monsters as he likes. Monsters do not respawn, thus each monster can be defeated at most one.

Thanks to Roma's excessive donations, we can assume that he has an infinite amount of in-game currency and can afford any of the weapons and armor sets. Still, he wants to maximize the profit of the grind. The profit is defined as the total coins obtained from all defeated monsters minus the cost of his equipment. Note that Roma must purchase a weapon and an armor set even if he can not cover their cost with obtained coins.

Help Roma find the maximum profit of the grind.

## 分析

这题代表了一类基本的思路。枚举其中一维，用数据结构维护另一维。

假设我们枚举武器攻击力，首先能够挑出所有被击杀的怪物。当我们的防具为k时，所有攻击力小于k的都可以计算收益。反过来说，对于攻击力为k的怪物，只要我们的防具大于k，就可以计算收益。

所以每枚举一个主角攻击力，我们把怪物按照其攻击力为下标作收益前缀和，就可以获取到当主角防具为k时所能得到的收益。为了计算入防具的花费，预处理每个防具等级k所需要的最少的钱，预先在相应下标扣除。此时还需要一个结构来求去所有前缀和中最大。这些操作都可以用线段树完成。

## 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <limits>
using namespace std;
using ll=long long;
const int MAXN=1000100*4;

int idx=0;
int lc[MAXN],rc[MAXN];
ll award[MAXN];
int cost[MAXN];
ll flag[MAXN];
int root;

int wlen,alen,mlen;
struct Weapon {
    int rnk, cost;

    bool operator<(const Weapon &b) const {
        //if (rnk == b.rnk)return cost < b.cost;实际没用
        return rnk < b.rnk;
    }
} weapons[MAXN];
struct Armor {
    int rnk, cost;

    bool operator<(const Armor &b) const {
        if (rnk == b.rnk)return cost < b.cost;
        return rnk < b.rnk;
    }
} armors[MAXN];

void collect(int n){
    award[n]=max(award[lc[n]],award[rc[n]]);
}

void pushdown(int n){
    if(flag[n]){
        flag[lc[n]]+=flag[n];
        flag[rc[n]]+=flag[n];
        award[lc[n]]+=flag[n];
        award[rc[n]]+=flag[n];
        flag[n]=0;
    }
}

//枚举武器，维护防具
void build(int &n,int l,int r,int init=-0x3f3f3f3f) {
    if (!n)n = ++idx;
    if (l == r) {
        award[n]=-cost[l];
        return;
    }

    int mid = (l + r) / 2;
    build(lc[n], l, mid,init);
    build(rc[n], mid + 1, r,init);
    collect(n);
}

void modify(ll x,int l,int r,int L,int R,int n) {
    if (l <= L && R <= r) {
        flag[n] += x;
        award[n] += x;
        return;
    }
    pushdown(n);

    int mid = (L + R) / 2;
    if (l <= mid)modify(x, l, r, L, mid, lc[n]);
    if (mid < r)modify(x, l, r, mid + 1, R, rc[n]);

    collect(n);
}

ll query(){
    return award[root];
}

ll query(int l,int r,int L,int R,int n){
    if(l<=L && R<=r){
        return award[n];
    }
    pushdown(n);

    ll res=-numeric_limits<ll>::max();
    int mid=(L+R)/2;
    if(l<=mid)res=max(res,query(l,r,L,mid,lc[n]));
    if(mid<r)res=max(res,query(l,r,mid+1,R,rc[n]));
    return res;
}

struct Monster{
    int att,de;
    int cost;
} monsters[MAXN];

void debug(int n){
    for(int i=1;i<=n;i++){
        cout<<query(i,i,1,n,root)<<" ";
    }
    cout<<endl;
}

int main() {
    ios::sync_with_stdio(false);

    cin >> wlen >> alen >> mlen;
    int vallim = 0;
    for (int i = 0; i < wlen; i++)cin >> weapons[i].rnk >> weapons[i].cost;
    for(int i=0;i<alen;i++)cin>>armors[i].rnk>>armors[i].cost;
    for(int i=0;i<mlen;i++)cin>>monsters[i].de>>monsters[i].att>>monsters[i].cost;

    sort(weapons,weapons+wlen);
    sort(monsters,monsters+mlen,[](auto a,auto b){
        return a.de<b.de;
    });

    fill(cost,cost+MAXN,numeric_limits<int>::max());
    int maxarm=0;
    for(int i=0;i<alen;i++){
        cost[armors[i].rnk]=min(cost[armors[i].rnk],armors[i].cost);
        maxarm=max(maxarm,armors[i].rnk);
    }
    for(int i=maxarm-1;i>=1;i--){
        cost[i]=min(cost[i],cost[i+1]);
    }

    build(root,1,maxarm);
    //debug(maxarm);

    ll ans=-numeric_limits<ll>::max();
    int ptr=0;
    for(int i=0;i<wlen;i++){
        while(ptr<mlen && monsters[ptr].de<weapons[i].rnk){
            if(monsters[ptr].att<maxarm){
                modify(monsters[ptr].cost,monsters[ptr].att+1,maxarm,1,maxarm,root);
            }
            ptr++;
        }
        ans=max(ans,query()-weapons[i].cost);
    }
    cout<<ans<<endl;


    return 0;
}
```
