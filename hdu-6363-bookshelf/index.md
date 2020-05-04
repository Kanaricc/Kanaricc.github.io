# [HDU 6363] bookshelf

Patrick Star bought a bookshelf, he named it ZYG !!

Patrick Star has N book .

The ZYG has K layers (count from 1 to K) and there is no limit on the capacity of each layer !

Now Patrick want to put all N books on ZYG :

1. Assume that the i-th layer has cnti(0≤cnti≤N) books finally.

2. Assume that f[i] is the i-th fibonacci number (f[0]=0,f[1]=1,f[2]=1,f[i]=f[i−2]+f[i−1]).

3. Define the stable value of i-th layers stablei=f[cnti].

4. Define the beauty value of i-th layers beautyi=2stablei−1.

5. Define the whole beauty value of ZYG score=gcd(beauty1,beauty2,...,beautyk)(Note: gcd(0,x)=x).

Patrick Star wants to know the expected value of score if Patrick choose a distribute method randomly !

## 分析

我tm这辈子学不会数论了。

先是两个【——】结论

* $\gcd(2^a-1,2^b-1)=2^{\gcd(a,b)}-1$
* $\gcd(\text{fib}(a),\text{fib}(b))=\text{fib}(\gcd(a),\gcd(b))$

然后原题的score就变成

<div>$$2^{\text{fib}(\gcd(a_1,a_2,\cdots))}-1$$</div>

接下来才有机会瞎搞。枚举gcd，$d|n$，$a$的分配方式有$C^{n/d+k-1}_{k-1}$种。这种分配包括了d'是d的倍数的所有情况的可能。令$F(d)=C^{n/d+k-1}_{k-1}$，$f(d)$为真正的数目，于是有

<div>$$F(d)=\sum_{d|d'}f(d')$$</div>

依照莫比乌斯反演，有

<div>$$f(d)=\sum_{d|d'}\mu(\frac{d'}{d})F(d')$$</div>

于是原题得以解决。

