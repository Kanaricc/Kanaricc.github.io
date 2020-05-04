# CF1285D Dr.Evil Underscores

Today, as a friendship gift, Bakry gave Badawy n integers a1,a2,…,an and challenged him to choose an integer X such that the value max1≤i≤n(ai⊕X) is minimum possible, where ⊕ denotes the bitwise XOR operation.

As always, Badawy is too lazy, so you decided to help him and find the minimum possible value of max1≤i≤n(ai⊕X).

<!--more-->

## 分析

将输入按照二进制位从高位开始建树,就能看出来,一旦确定了高位$i$填1还是0后,只会有一棵子树影响答案.另一棵子树的所有数字都因为$i$位上的异或结果为0而定小于另一个子树.

树不用真的建出来,这样就可以做了.

## 代码

```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <limits>
#include <vector>
using namespace std;
using ll=long long;
const ll LMAX=numeric_limits<ll>::max();
const int MAXN=100010;
 
//int nlen;
//int num[MAXN];
vector<int> num;
 
int dfs(int ptr,const vector<int> &vec){
    if(ptr<0)return 0;
    vector<int> one,zero;
    for(auto i:vec){
        if((i>>ptr)&1==1)one.push_back(i);
        else zero.push_back(i);
    }
    /*
    cout<<"====vec===="<<endl;
    for(auto i:one)cout<<i<<",";
    cout<<endl;
    for(auto i:zero)cout<<i<<",";
    cout<<endl;
    cout<<"==========="<<endl;
    */
    if(one.size()==0) return dfs(ptr-1,zero);
    if(zero.size()==0) return dfs(ptr-1,one);
    //cout<<"add"<<(1<<ptr)<<endl;
 
    return (1<<ptr)+min(dfs(ptr-1,zero),dfs(ptr-1,one));
}
 
 
int main(){
    ios::sync_with_stdio(false);
 
    int nlen;cin>>nlen;
    for(int i=0;i<nlen;i++){
        int x;cin>>x;
        num.push_back(x);
    }
 
    cout<<dfs(30,num)<<endl;
 
    return 0;
}
```
