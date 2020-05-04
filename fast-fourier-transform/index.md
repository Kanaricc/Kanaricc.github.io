# 快速傅里叶变换

> 写代码是不可能写代码的,今下午是不想写代码的.不想写代码,又不想咸鱼,就只能靠学点新东西来假装自己在工作的样子,心里才能好受些.
>
> > 窃格码拉

几乎可以肯定,下面的内容肯定会出锅.

<!--more-->

## 傅里叶变换的实际意义
### 从电压说起
被模电折磨的同学都知道,有种东西叫做傅里叶级数,可以将成周期性变化的电压分解为数个三角函数波的叠加.

在这里,我们提出另一个问题,如果不知道周期,该如何将这些叠加在一起的信号拆分为单纯的三角波?

> 三角波叠加图象周期并不那么显然,也许你可以试一试.

### 一种缠绕机
有一种奇特的方法,我们将一段`时域图象`在笛卡尔坐标系上以原点为圆心**绕**起来,一圈一圈缠起来,然后调整源图象缠绕的速率(几秒一圈),观察整个图形的**质心**变化.

这个质心会随着图象缠绕的频率而发生位移.取质心的x坐标为y轴,以缠绕的频率为横轴,作出图象`B`.在这个图象上,会观测到一个现象:(假设我们已知原图像的分解三角波频率)当缠绕频率接近某个源三角波的频率时,缠绕图象出现**重合**,质心相对原点出现较大位移,图象出现一个**峰值**.

继续调整缠绕频率,峰值消失,图象回归到小范围波动.

通过观察图象`B`,可以认为,出现峰值的频率对应着一个频率的源三角波.如此,就将叠加图象还原了回去.

## 傅里叶变换的数学实现
现在,考虑如何通过的数学的方法来实现这个缠绕.

### 如何缠绕
将$g(t)$的图象缠绕到圆上听起来挺奇怪的,有这样的数学方法吗.

有一个东西,叫做$y=e^{ix}$.当其图象画在复平面时,就出现了有趣的事情:一个圆.对这个公式做一些加工.

<div>$$
y=g(t)e^{-2\pi i ft}
$$</div>

如此,就能够将$g(t)$以$f$频率缠绕.

### 关于质心
上文我们取质心的x坐标作图,现在需要稍微修改一下.

实际上,我们关心的是质心相对于原点的偏移**距离**.同样,以复平面的方式来表示质心位置就能够同时保留x和y坐标信息.

关于如何求取质心,其实也很直观.选取缠绕图象上的数个点,取平均,就是质心的大约位置.当点的数目达到极限,求和公式化为积分,所求即为质心.

<div>$$
\hat g(f)=\frac{1}{t_2-t_1} \int_{t_1}^{t_2}{g(t)e^{-2\pi i ft}}
$$</div>

### ?
目前为止,这几乎已经是傅里叶变换了.在数学应用时,傅里叶变换会去掉取均值,即

<div>$$
\hat g(f)=\int_{t_1}^{t_2}{g(t)e^{-2\pi i ft}}
$$</div>

也就是说,取样的时域信号越长,该质心的偏移倾向越大,这和我们想要的效果一致.

这就是傅里叶变换,实现了`时域信号`到`频域信号`的转换.

此外,还有方法将频域信号再次逆变换为时域信号的方法.

## 应用
傅里叶变换在很多领域都有重要作用.只要问题能转换为时域频域之间的变化,就有傅里叶变换的用武之处.

比如,在音频处理软件中,常常有一个功能叫做`消除人声`.基于傅里叶变换我们可以设计一个(至少理论上有用)的算法.

首先,任何声音都是相应频率的波对气压变化引起的,也就是不同**频率**的波在**时间**上叠加在一起,产生了声音.将源波使用傅里叶变换拆分到多个三角波上去.**删除人声所在的频域**,再将频域信号逆变换为时域信号.人声便消失了.

## 离散傅里叶变换
连续意义下的傅里叶变换先到此为止.在计算机中所处理的数据一般都是离散的.我们需要的是离散傅里叶变换.

离散意义下的时域信号和频域信号就都变成了点集.当从连续向离散过渡时,可以这样思考:

> 在连续的图象上以一定间隔**取样**得到离散点集.使用该点集进行傅里叶变换.

这也是我们一开始采取的质心求解方法,只不过,这次我们从缠绕时就取样.(质点依然是真正的质点)
对于点集$g(0\leq n < N)$,它的傅里叶变化就是

<div>$$
\hat g(k)=\sum_{n=0}^{N-1}g(n)e^{-i\frac {2\pi}{N}nk}
$$</div>

简单粗暴.

## 快速傅里叶变换
在了解了关于傅里叶变换的一系列背景与一个应用后,我们再回来解决一些重要的问题.

根据傅里叶变换的公式定义

<div>$$
\hat g(k)=\sum_{n=0}^{N-1}g(n)e^{-i\frac {2\pi}{N}nk}
$$</div>

其朴素算法的时间复杂度为$O(N^2)$ 这个复杂度还不够优秀.一种快速傅里叶变换算法利用$e^{ix}$的性质,将复杂度降低到了$O(N\lg N)$.

### 单位根
给$e^{ix}$个名字.

在数学上, $n$次**单位根**是 $n$次幂为1的复数.它们位于复平面的单位圆上,构成正n边形的顶点,其中一个顶点是1.

记

<div>$$
\omega_{n,k}=-e^{i\frac{2\pi}{n}k}
$$</div>

其几何意义为单位圆上的n等分点的顺时针第k个.

> 一般来说,单位根取逆时针,不过这里为了方便,取顺时针.

如同三角函数一样,单位根存在一些显然的定理.

**折半:**$\omega_{2n,2k}=\omega_{n,k}$

**化简:**$\omega_{n,k+\frac 2n}=-\omega_{n,k}$

#### 修改公式
来看原本的离散傅里叶变换公式

<div>$$
\hat g(k)=\sum_{n=0}^{N-1}g(n)e^{-i\frac {2\pi}{N}nk}
$$</div>

使用单位根来替换一下

<div>$$
\hat g(k)=\sum_{n=0}^{N-1}\omega_{N,nk} g(n)
$$</div>

按照化简公式的指引,将求和公式按照单位根奇偶拆分为2部分.

<div>$$
\begin{aligned}
\hat g(k) &= \sum_{n=0}^{N-1}\omega_{N,nk} g(n) \\
&= \sum_{0 \leq n < N}\omega_{N,nk} g(n) \\
&= \sum_{0 \leq 2n < N}\omega_{N,2nk} g(2n) + \sum_{0 \leq 2n+1 < N}\omega_{N,(2n+1)k} g(2n+1) \\
&=\sum_{0 \leq 2n < N}\omega_{N,2nk} g(2n) + \sum_{0 \leq 2n+1 < N}\omega_{N,2nk+k} g(2n+1) \\
&=\sum_{0 \leq 2n < N}\omega_{\frac N2,nk} g(2n) + \omega_{N,k}\sum_{0 \leq 2n+1 < N}\omega_{\frac N2,nk} g(2n+1) \\\\
&=\hat g_{even}(k)+\omega_{N,k} \hat g_{odd}(k)
\end{aligned}
$$</div>

注意到不管是$\hat g_{even}(k)$还是$\hat g_{even}(k)$,它们都以$N/2$为周期.接下来,应用化简定理

<div>$$
\hat g(k+\frac N2)=\hat g_{even}(k)-\omega_{N,k} \hat g_{odd}(k)
$$</div>

将这2个式子放在一起

<div>$$
\hat g(k)=\hat g_{even}(k)+\omega_{N,k} \hat g_{odd}(k) \\
\hat g(k+\frac N2)=\hat g_{even}(k)-\omega_{N,k} \hat g_{odd}(k)
$$</div>

当k取遍原问题规模的一半时,可以直接由第二个式子得到另一半.问题的规模减半.递归求解,最终的复杂度就降到了$O(N \lg N)$.

这就是`Cooley-Turkey`快速傅里叶变换算法.

## 快速乘法
定义多项式$A(x)=\sum_k a_kx^{k+1}$,$B(x)$同理,求解$C(x)=A(x)B(x)$.

很容易看出,朴素算法的复杂度为$O(N^2)$.

现在,来看如何使用FFT快速计算.

### 点值表示
> 对于一个次数为$n-1$的多项式,其图象上互不相同的$n$个点可以唯一确定该多项式.
> ...
> 如同确定混合在一起的几个波一样.

至于为什么是对的,大可在Google上搜索一番.

取$x$为数个单位根,在$A(x)$和$B(x)$上利用单位根的性质得到$A$和$B$的点值表示,将点值相乘得到$C$的点值表示.之后,将$C$的点值表示再转换为系数表示.

嗯?FFT在哪?

其实在这里,

<div>$$
\hat g(\omega)=A(\omega)=\sum_{n}\omega A[x^{n+1}]
$$</div>

### 离散卷积
有一种**数学运算**,叫做**卷积**.现在只讨论它的离散情况.

<div>$$
(f * g)(n)=\sum_{\tau=-\inf}^{inf}f(\tau)g(n-\tau)
$$</div>

这玩意的意义...实在是不怎么明显.不过好在我们只是想算个多项式乘法,也就是把多项式的**系数**算来算去:

$C[x^n]$ 表示多项式$C$中$x^n$项的系数.

<div>$$
C[x^{n}]=\sum_{\tau=0}^{n}A[x^\tau]B[x^{n-\tau}]
$$</div>

嗯?

如果我们设多项式中不存在的项的系数为0的话.

<div>$$
C[x^{n}]=\sum_{\tau=-\inf}^{\inf}A[x^\tau]B[x^{n-\tau}]
$$</div>

哈,

<div>$$
C[x^n]=(A*B)[x^n]
$$</div>
### 卷积定理
卷积定理指出:

> 一个域中的卷积对应于另一个域中的乘积.

这意味着,上面这个计算(卷积)对应着另一个域里的乘积.也就是

<div>$$
F(C[x^n])=F(A[x^n]) \cdot F(B[x^n])
$$</div>

这便是深层原理.对A和B的取样(频域)称为A和B的点值表示,最终以乘积的方式得到了C的点值表示(频域).用FFT来计算乘法的说法是对的.

## 傅里叶逆变换
如何从一个频域信号再得到时域信号?

<div>$$
g(n)=\frac 1N\sum_{k=0}^{N-1}e^{i\frac{2\pi}{N}nk} \hat g(k)
$$</div>

> 注意:此处的1/N与上面的变换是相匹配的.

这个式子可以理解成对变换后的式子再变换,意味着它同样可以用变换时的思想来加速.

## FFT的C++实现
一个值得注意的问题就是,对于单位根的运算涉及到了精度的问题.但目前还不需要讨论.

### 翻转操作
可以观察到,按照上面的算法实现,我们需要在每次递归按照奇和偶将取样分组.且每次递归都会分组.每次分组都会涉及到数组的复制,常数较大.

观察分组操作中下标的变化.

```
(表示下标)
0 1 2 3 4 5 6 7
0 2 4 6 | 1 3 5 7
0 4 | 2 6 | 1 5 | 3 7
```

将其转换为二进制
```
000 001 010 011 100 101 110 111
000 100 010 110 001 101 011 111
```

可以发现,最终的分组结果就是将原下标二进制翻转.所以可以直接一次完成分组.

注意,**这要求取样为2的幂次**.

### 🦋蝴蝶操作
解决了递归中由顶至底的分组后,接下来优化子问题合并时的数组复制.

观察原来的合并式子

<div>$$
\hat g(k)=\hat g_{even}(k)+\omega_{N,k} \hat g_{odd}(k) \\
\hat g(k+\frac N2)=\hat g_{even}(k)-\omega_{N,k} \hat g_{odd}(k)
$$</div>

按照算法中的实现方法,其为

<div>$$
\hat g(k)=\hat g(k)+\omega_{N,k} \hat g(k+\frac N2) \\
\hat g(k+\frac N2)=\hat g(k)-\omega_{N,k} \hat g(k+\frac N2)
$$</div>

想要省略数组复制,进行原地合并,问题出在新数值太早地替换掉了我们需要的数值.

取辅助变量,修改原式

<div>$$
t=\omega_{N,k} \hat g(k+\frac N2)\\
\hat g(k+\frac N2)=\hat g(k)-t\\
\hat g(k)=\hat g(k)+t
$$</div>

这个操作被称为"蝴蝶操作",名字很有意思.

### 代码
在这段代码中同时去掉了递归.

```cpp
const double PI=acos(-1);

inline complex<double> gomega(int n,int k,bool rev=false){
    complex<double> res(cos(2*PI/n*k),sin(2*PI/n*k));
    if(rev)return conj(res);
    else return res;
}
const int MAXN=10;
struct FFT{
    complex<double> omega[MAXN],omegaI[MAXN];

    FFT(int n){
        for(int i=0;i<n;i++){
            omega[i]=gomega(n,i);
            omegaI[i]=gomega(n,i,1);
        }
    }
    void transform(complex<double> *a,int n,const complex<double> *omega){
        for(int i=0,j=0;i<n;i++){
            if(i>j)swap(a[i],a[j]);
            //二进制换位
            for(int l=n/2;(j^=l)<l;l>>=1);
        }
        for(int l=2;l<=n;l<<=1){
            int m=l/2;
            for(complex<double> *p=a;p!=a+n;p+=l){
                for(int i=0;i<m;i++){
                //蝴蝶操作
                    complex<double> t=omega[n/l*i]*p[m+i];
                    p[m+i]=p[i]-t;
                    p[i]+=t;
                }
            }
        }
    }
    void dft(complex<double> *a,int n){
        transform(a,n,omega);
    }
    void idft(complex<double> *a,int n){
        transform(a,n,omegaI);
        for(int i=0;i<n;i++)a[i]/=n;
    }
};
```

如果想要实现快速乘法,只要将2个多项式的系数函数传入进行变换,变换结果相乘并逆变换即可.

## 应用

### 快速乘法
指快速大数乘法。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <complex>
#include <cmath>
using namespace std;

const int MAXN=300000;
const double PI=acos(-1);

inline complex<double> gomega(int n,int k,bool rev=false){
    complex<double> res(cos(2*PI/n*k),sin(2*PI/n*k));
    if(rev)return conj(res);
    else return res;
}
struct FFT{
    complex<double> omega[MAXN],omegaI[MAXN];

    FFT(){
    }
    void init(int n){
        for(int i=0;i<n;i++){
            omega[i]=gomega(n,i);
            omegaI[i]=gomega(n,i,1);
        }
    }

    void transform(complex<double> *a,int n,const complex<double> *omega){
        for(int i=0,j=0;i<n;i++){
            if(i>j)swap(a[i],a[j]);
            //二进制换位
            for(int l=n/2;(j^=l)<l;l>>=1);
        }
        for(int l=2;l<=n;l<<=1){
            int m=l/2;
            for(complex<double> *p=a;p!=a+n;p+=l){
                for(int i=0;i<m;i++){
                    //蝴蝶操作
                    complex<double> t=omega[n/l*i]*p[m+i];
                    p[m+i]=p[i]-t;
                    p[i]+=t;
                }
            }
        }
    }
    void dft(complex<double> *a,int n){
        transform(a,n,omega);
    }
    void idft(complex<double> *a,int n){
        transform(a,n,omegaI);
        for(int i=0;i<n;i++)a[i]/=n;
    }
};

complex<double> a[2][MAXN];
int ans[MAXN];
FFT fft;
int main(){

    int nlen;cin>>nlen;
    int n=1;
    //根据原理，n必须取大于2nlen的数，才能满足取样要求和反转操作要求
    while(n<2*nlen)n*=2;
    fft.init(n);


    for(int i=0;i<2;i++){
        string inp;cin>>inp;
        for(int j=0,k=inp.size()-1;j<inp.size();j++,k--){
            a[i][j]=complex<double>(inp[j]-&#039;0&#039;,0);
        }
        fft.dft(a[i],n);
    }
    for(int i=0;i<n;i++)a[0][i]=a[0][i]*a[1][i];
    fft.idft(a[0],n);
    int reslen=nlen+nlen-1;
    for(int i=reslen-1,k=0;i>=0;i--,k++)
        ans[k]=(int)floor(a[0][i].real()+0.5);
    /*
    for(int i=0;i<reslen;i++)cout<<ans[i]<<" ";
    cout<<endl;
    */
    for(int i=0;i<MAXN;i++){
        ans[i+1]+=ans[i]/10;
        ans[i]%=10;
    }
    int ptr=MAXN-1;
    while(ans[ptr]==0)ptr--;
    for(;ptr>=0;ptr--)cout<<ans[ptr];
    cout<<endl;

    return 0;
}
```

### 实际意义

但是更加值得关心的是其广义上的物理意义。N个采样点拿到的N个值到底对应着什么。

设采样率为$F$，采样间隔为$T=\frac{1}{F}$。设采样次数$N$。那么显然不管是数据下标还是结果下标，都不是其原有意义。

有如下对应关系

* 采样时长为$TN$或者$N/F$（采样区间长度）
* 数据下标很显然

结果略有一点点绕。对于「在$N$个点中出现了$k$个周期」的基信号，一个周期$N/k$个点，一个周期时长$\frac{N}{kF}$。如果不想把你的频域图像画爆那就倒过来取周期频率$\frac{F}{N}k$。

即结果的点下标$x$对应着频率为$\frac{F}{N}x$的基信号。
