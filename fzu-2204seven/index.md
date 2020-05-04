# [FZU 2204]Seven

n个有标号的球围成一个圈。每个球有两种颜色可以选择黑或白染色。问有多少种方案使得没有出现连续白球7个或连续黑球7个。

对方案数mod 2015，球最多有100000个。

# 分析
考虑对于非环状球的答案计算，可以设$sum(i,k)$表示第i个球为k色时的方案数。其计算非常显然

<div>$$
sum(i,k)=\sum_{1\leq j \leq 6}{sum(i-j,1-k)}
$$</div>

接下来考虑收尾相接后需要排除的情况，即收尾同色球长度相加超过6的情况，这可以直接枚举。

首取i个末取j个同色，从答案中删除此时剩下球的方案数，注意剩下的球的首末球颜色**不能**和已经枚举的颜色同色。鉴于这种要求，我们退回到sum的递推公式处，决定sum的边界条件为首个球固定为黑色，这样就能很方便的确定球的颜色，且根据对称性答案可以直接x2得到。

题就做完了。

# 代码
> 淦，为什么当时没写。

```cpp
#include <iostream>
using namespace std;
const int MAXN=100010;
const int P=2015;

int sum[MAXN][2];
int main(){
    int kase;cin>>kase;
    sum[0][1]=1;
    for(int i=1;i<=100000;i++){
        for(int j=1;j<=min(i,6);j++){
            (sum[i][1]+=sum[i-j][0])%=P;
            (sum[i][0]+=sum[i-j][1])%=P;
        }
    }
    sum[0][1]=0;
    int cnt=0;
    while(kase--){
        int nlen;cin>>nlen;
        int ans=(sum[nlen][0]+sum[nlen][1])%P;
        if(nlen>=7)
            for(int i=1;i<=6;i++)
                for(int j=1;j<=6;j++)
                    if(i+j>=7 && nlen-i-j>=0)
                        ans=(ans-sum[nlen-i-j][0])%P;
        cout<<"Case #"<<++cnt<<": "<<((ans*2)%P+2015)%P<<endl;
    }
    return 0;
}
```

