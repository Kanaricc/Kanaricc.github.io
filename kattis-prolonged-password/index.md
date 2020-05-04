# Prolonged Password


给一个字符串，现有这么一种操作，将字符串内每一个字母替换成一给定字符串。例如$T_a=abc,T_b=ee$，那么$ab$就会被替换成$abcee$。

给出初始的字符串$S$，和26个字母所对应的字符串$T_i$，应用操作$K$次，询问第$i$个位置的字符。

$$|S| \leq 1000000$$

$$2 \leq |T_i| \leq 50,K \leq 10^{15}$$

## 分析

头秃。开始以为关键是在于数出一个字符串经过K次操作后达到的长度，然后然后用类似于二分的方法去递归地分层找到需要的位置。于是就统计每个字母的数目，用矩阵快速幂拿到长度……

现在看着过于智障。这1e15的指数和1e15的下标，你怎么敢快速幂。

由替换字符串的长度可以得到，这种操作顶多执行50次（$2^{50}$）。因此字符串的长度部分计算实际上不占大头，用个倍增就能处理出来。

现在剩下的问题是操作次数$K$过大。方法仍然是基于50次。一个字符扩展50次，就足够把所有的询问包含在内。因此要计算出第一个字符替换的循环节，将操作次数降到接近50。此后通过倍增法去一层层定位询问下标所处的位置。

最开始方向错了是一码事…感觉K的范围也挺无聊的…代码等到之后再补上。

## 代码

实际实现时，降K可能浮动26左右，所以多处理这么一点。

```cpp
#include <vector>
#include <algorithm>
#include <iostream>
#include <stack>
#include <cstring>
#include <cassert>
using namespace std;
using ll=long long;
const int MAXN=30;

string inp;
ll K;
string rep[30];
ll f[26][81];

void upd(ll &a,ll b){
    if(a==-1)return;
    if(b==-1){
        a=-1;
        return;
    }
    a+=b;
    if(a>1e15)a=-1;
}

char reduceK(){
    //找到循环节
    char chr=inp[0];
    bool vis[30];
    stack<char> st;
    memset(vis,0,sizeof(vis));
    while(!vis[chr-'a']){
        st.push(chr);
        vis[chr-'a']=1;
        chr=rep[chr-'a'][0];
    }

    int cycle=1;
    while(st.top()!=chr){
        cycle++;
        st.pop();
    }
    st.pop();

    int pre=st.size();

    //将K削到54-80
    if(K<=54)return '\0';
    if(K-pre<=54)return '\0';
    K-=pre;
    K=K-(K-54)/cycle*cycle;
    return chr;
}

char solve(ll idx,string str,int p){
    if(p==0){
        return str[idx-1];
    }
    for(int i=0;i<str.size();i++) {
        if (f[str[i] - 'a'][p] == -1 || f[str[i] - 'a'][p] >= idx)return solve(idx, rep[str[i] - 'a'],p-1);
        idx-=f[str[i] - 'a'][p];
    }
    //不该出现这种情况
    assert(false);
}

int main(){
    ios::sync_with_stdio(false);
    cin>>inp;//

    for(int i=0;i<26;i++){
        cin>>rep[i];
        f[i][1]=rep[i].size();
    }
    for(int i=0;i<26;i++)f[i][0]=1;
    cin>>K;
    for(int p=2;p<=80;p++){
        for(int i=0;i<26;i++){
            for(int k=0;k<rep[i].size();k++){
                upd(f[i][p],f[rep[i][k]-'a'][p-1]);
            }
        }
    }
    char start=reduceK();

    int qlen;cin>>qlen;
    while(qlen--){
        ll idx;cin>>idx;
        if(start=='\0')cout<<solve(idx,inp,K)<<endl;
        else cout<<solve(idx,rep[start-'a'],K-1)<<endl;//-1
    }

}
```


