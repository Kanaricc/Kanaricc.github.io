# 咸鱼数论

## 一些结论
* $N!$的质因数分解中某质数的指数为$\sum_{r=1}^{\inf}n/p^r $
* 约数个数为质因数指数+1的乘积，和为质因数枚举指数次和的乘积。
* 费马小定理要求p是质数

## 欧拉函数
小于x且与其互质的数的个数
<div>$$
\phi(x)=x\prod_{k=1}^n(1-\frac{1}{p_k})
$$</div>

* $phi(1)=1$
* $phi(p)=p-1$,当p为质数
* $phi(2n)=phi(n)$
* $phi(phi(phi...))))=1$

对于任意积性函数$f(xy)=f(x)f(y)$，可以筛。欧拉函数非完全积性函数。

* $phi(xy)=phi(x)(y-1)$,当x与y互质
* $phi(xy)=phi(x)y$,当x与y不互质

```cpp
for(int i=2;i<=n;i++){
    if(!no[i]){
        p[++cnt]=i;
        phi[i]=i-1;
    }
    for(int j=1;j<=cnt&&p[j]*i<=n;j++){
        no[p[j]*i]=1;
        if(i%p[j]==0){
            phi[p[j]*i]=phi[i]*p[j];
            break;
        }
        phi[p[j]*i]=phi[i]*(p[j]-1);
    }
}
```

## 扩展欧几里得
* 存在x，y使得ax+by=gcd（a，b）
* 求逆元，要求x与模数互质

```cpp
void exgcd(ll a,ll b,ll &x,ll &y){
    if(!b){
        x=1,y=0;
    }else{
        exgcd(b,a%b,y,x);
        y-=(a/b)*x;
    }
}
```

## 递推逆元
inv(i) = inv(mod % i) * (mod-mod/i) % mod;

* 阶乘的逆元：inv(i)=inv(i+1)*(i+1)

