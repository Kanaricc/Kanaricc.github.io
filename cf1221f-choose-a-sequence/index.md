# [CF1221F] Choose a Sequence

Petya recently found a game "Choose a Square". In this game, there are nn points numbered from 11 to nn on an infinite field. The ii-th point has coordinates (xi,yi) and cost ci.

You have to choose a square such that its sides are parallel to coordinate axes, the lower left and upper right corners belong to the line y=x, and all corners have integer coordinates.

The score you get is the sum of costs of the points covered by the selected square minus the length of the side of the square. Note that the length of the side can be zero.

Petya asks you to calculate the maximum possible score in the game that can be achieved by placing exactly one square.

{% asset_img a.png %}

## 分析

这道题其实和多校的Snowy Smile（题文无关）挺像的。

<!--more-->

需要转化条件。正方形全部位于坐标系第一象限的平分线上，可以表示为$(l,r)$这种形式。一个在正方形内的点$(x,y)$(假设$x < y$,由于这个题目的特性，不满足时可以直接调换)满足的条件为

<div>$$l \leq x \leq y \leq r$$</div>

由此，这道题就变成了二维偏序。本来的本来，上个长得像树状数组一类的东西就解决了。

将点按照第一维排序，从大到小枚举，在数据结构中维护结尾到r的前缀和最大值，并查询，更新全局答案。不过这个题目还要求权值要减去正方形边长w。

<div>$$f(r)-(r-l)=l+[f(r)-r]$$</div>

维护最大值，y分开，只在pushup时合在一起。

然后还要离散化……

## 代码

这题给我整自闭了，我想用线段树单点修改来魔幻维护我需要的数据，加加减减。结果发现2种维护方式都有无法解决的问题。最后只能打tag。

线段树维护$f(n)$为到$n$为止的前缀和。每次加点以$x$为组，对已有点减去新增线段的长度，再加上新点的影响。那么在线段树上维护的就是答案。取出最大值即可。

代码写得又长又丑，不过过了。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
using ll=long long;
const int MAXN=500010*4;

ll tag[MAXN];
ll dMax[MAXN];
int dy[MAXN];
int lc[MAXN],rc[MAXN],idx=0;
void build(int &n,int l,int r){
    if(!n)n=++idx;
    dy[n]=r;
    if(l>=r){
        tag[n]=0;
        return;
    }

    int mid=(l+r)/2;
    build(lc[n],l,mid);
    build(rc[n],mid+1,r);
}
void pushdown(int n,int l,int r){
    if(!tag[n])return;
    tag[lc[n]]+=tag[n];
    tag[rc[n]]+=tag[n];
    dMax[lc[n]]+=tag[n];
    dMax[rc[n]]+=tag[n];

    tag[n]=0;
}
void collect(int n){
    if(dMax[lc[n]]>dMax[rc[n]]){
        dMax[n]=dMax[lc[n]];
        dy[n]=dy[lc[n]];
    }else{
        dMax[n]=dMax[rc[n]];
        dy[n]=dy[rc[n]];
    }
}
void modify(int l,int r,int x,int L,int R,int n){
    if(l<=L && R<=r){
        tag[n]+=x;
        dMax[n]+=x;
        return;
    }
    pushdown(n,L,R);

    int mid=(L+R)/2;
    if(l<=mid)modify(l,r,x,L,mid,lc[n]);
    if(mid<r)modify(l,r,x,mid+1,R,rc[n]);
    collect(n);
}

bool operator<(const pair<ll,int> &a,const pair<ll,int> &b){
    return a.first<b.first;
}

pair<ll,int> query(int l,int r,int L,int R,int n){
    if(l<=L && R<=r){
        return make_pair(dMax[n],dy[n]);
    }
    pushdown(n,L,R);
    
    int mid=(L+R)/2;
    pair<ll,int> res=make_pair(-0x3f3f3f3f,0);
    if(l<=mid)res=max(res,query(l,r,L,mid,lc[n]));
    if(mid<r)res=max(res,query(l,r,mid+1,R,rc[n]));

    return res;
}
struct Point{
    int x,y,w;
} points[MAXN];
vector<int> nums;
int root=0;
int main(){
    /*
    build(root,1,10);
    int opt;
    while(cin>>opt){
        if(opt==1){
            int l,r,x;cin>>l>>r>>x;
            modify(l,r,x,1,10,root);
        }
        if(opt==2){
            int l,r;cin>>l>>r;
            cout<<query(l,r,1,10,root).first<<" "<<query(l,r,1,10,root).second<<endl;
        }
    }
    */
    

    int nlen;cin>>nlen;

    for(int i=1;i<=nlen;i++){
        Point &p=points[i];
        cin>>p.x>>p.y>>p.w;
        if(p.x>p.y)swap(p.x,p.y);
        nums.push_back(p.y);
        nums.push_back(p.x);
    }
    sort(nums.begin(),nums.end());
    auto rit=unique(nums.begin(),nums.end());
    for(int i=1;i<=nlen;i++){
        points[i].x=lower_bound(nums.begin(),rit,points[i].x)-nums.begin();
        points[i].y=lower_bound(nums.begin(),rit,points[i].y)-nums.begin();
    }

    sort(points+1,points+1+nlen,[](const Point &a,const Point &b){
        if(a.x!=b.x)return a.x>b.x;
        return a.y<b.y;
    });

    int len=rit-nums.begin()-1;
    build(root,0,len);
    vector<int>::iterator lastit=rit-1;
    pair<ll,int> ans=make_pair(-0x3f3f3f3f,0);
    int pairl=-0x3f3f3f3f;
    for(int i=1;i<=nlen;i++){
        int thisx=points[i].x;
        //auto it=lower_bound(nums.begin(),rit,points[i].x);
        auto it=nums.begin()+thisx;
        for(;lastit!=it;lastit--){
            modify(lastit-nums.begin(),len,*(lastit-1)-*lastit,0,len,root);
            //cout<<"cost "<<*(lastit-1)-*lastit<<" from "<<lastit-nums.begin()<<" to "<<len<<endl;
        }
        /*
        for(int j=0;j<=len;j++){
            cout<<query(j,j,0,len,root).first<<"\t";
        }
        cout<<endl;
        */
        while(i<=nlen && points[i].x==thisx){
            const Point &p=points[i];
            modify(p.y,len,p.w,0,len,root);
            //cout<<"add "<<p.w<<" from "<<p.y<<" to "<<len<<endl;
            i++;
        }
        i--;
        /*
        for(int j=0;j<=len;j++){
            auto temp2=query(j,j,0,len,root);
            cout<<temp2.first<<","<<temp2.second<<"\t";
        }
        cout<<endl;
        */

        pair<ll,int> temp=query(0,len,0,len,root);
        if(ans<=temp){
            ans=temp;
            pairl=nums[thisx];
        }
    }

    if(ans.first>0){
        cout<<ans.first<<endl;
        cout<<pairl<<" "<<pairl<<" "<<nums[ans.second]<<" "<<nums[ans.second]<<endl;
    }else{
        cout<<0<<endl;
        cout<<*nums.rbegin()+1<<" "<<*nums.rbegin()+1<<" "<<*nums.rbegin()+1<<" "<<*nums.rbegin()+1<<endl;
    }
    return 0;
}
```
