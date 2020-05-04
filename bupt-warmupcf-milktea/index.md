# [BUPT WARMUP|CF] 珍珠奶茶

给出一个$N \times N$的非负整数矩阵，要求找到一条从左上角数字到右下角数字的路线，且

* 只能向右或者下走。
* 将经过数字相乘后得到的结果，使其末尾的“0”最少。

<div>$$
N \leq 1000
$$</div>

<!--more-->

## 分析
大概是因为末尾的0长得像珍珠。

思考0是怎么出现的，可以发现结果中因数10的指数越小越好，即，使得经过的路上凑出的因数10最少即可。10的质因数分解为$2 \times 5$，以矩阵中每个数所含因数2和5的数目分别DP一遍求路径，再在两次DP的结果中取小。

一个特殊情况是数字里有0，那么经过0的路的末尾0一定是1个……一开始脑袋抽了以为是0个。如果其他情况的路径末尾0都多于1个的话，就特判走0。

## 代码 

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
constexpr int MAXN=1010;

int game[MAXN][MAXN];
int num[2][MAXN][MAXN];
int cal(int x,int fac){
    if(x==0)return 0;
    int res=0;
    while(x%fac==0){
        x/=fac;
        res++;
    }
    return res;
}
int n;
int dp[MAXN][MAXN];
int from[MAXN][MAXN];
void dodp(int fac){
    for(int i=0;i<MAXN;i++){
        for(int j=0;j<MAXN;j++){
            dp[i][j]=0x3f3f3f3f;
        }
    }
    dp[0][1]=dp[1][0]=0;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(dp[i-1][j]<dp[i][j-1]){
                from[i][j]=2;
                dp[i][j]=dp[i-1][j];
            }else{
                from[i][j]=1;
                dp[i][j]=dp[i][j-1];
            }
            dp[i][j]+=num[fac][i][j];
        }
    }
}
string genpath(){
    string res="";
    int i=n,j=n;
    for(int p=from[i][j];i!=1 || j!=1 ;p=from[i][j]){
        if(p==2){
            res+="D";
            i--;
        }else{
            res+="R";
            j--;
        }
    }
    reverse(res.begin(),res.end());
    return res;
}
int main(){
    ios::sync_with_stdio(false);
    cin>>n;
    bool haszero=false;
    int zeroi,zeroj;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            cin>>game[i][j];
            if(game[i][j]==0){
                haszero=1;
                zeroi=i,zeroj=j;
            }
        }
    }
    
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            num[0][i][j]=cal(game[i][j],2);
            num[1][i][j]=cal(game[i][j],5);
        }
    }
    int ans=0x7f7f7f7f;
    dodp(0);
    string P;
    if(ans>dp[n][n]){
        ans=dp[n][n];
        P=genpath();
    }
    dodp(1);
    if(ans>dp[n][n]){
        ans=dp[n][n];
        P=genpath();
    }

    if(haszero && ans>1){
        cout<<1<<endl;
        for(int i=1;i<zeroi;i++)cout<<"D";
        for(int j=1;j<zeroj;j++)cout<<"R";
        for(int i=zeroi+1;i<=n;i++)cout<<"D";
        for(int j=zeroj+1;j<=n;j++)cout<<"R";
        cout<<endl;
        return 0;
    }

    cout<<ans<<endl;
    cout<<P<<endl;

    return 0;
}
```

