# [CF703D] Mishka and Interesting Sum


Little Mishka enjoys programming. Since her birthday has just passed, her friends decided to present her with array of non-negative integers a1, a2, ..., an of n elements!

Mishka loved the array and she instantly decided to determine its beauty value, but she is too little and can't process large arrays. Right because of that she invited you to visit her and asked you to process m queries.

Each query is processed in the following way:

Two integers l and r (1 ≤ l ≤ r ≤ n) are specified — bounds of query segment.
Integers, presented in array segment [l,  r] (in sequence of integers al, al + 1, ..., ar) even number of times, are written down.
XOR-sum of written down integers is calculated, and this value is the answer for a query. Formally, if integers written down in point 2 are x1, x2, ..., xk, then Mishka wants to know the value , where  — operator of exclusive bitwise OR.
Since only the little bears know the definition of array beauty, all you are to do is to answer each of queries presented.

## 分析

> 被大哥们秒切了…都没来得及想💔

利用了异或的性质。

想找到一个区间内出现奇数次的数的异或和非常简单，只要维护一个前缀和就可以了。那么知道了奇数次的数字，就能从所有数字的异或中求到补——出现偶数次的数的异或。

值得一写的是求某区间内**不同元素**相关信息的一个思路。之前有好几道题都是这么干的（一个钥匙的题，一个书的题）。即维护元素的下一次出现位置，将左区间排序后，使用数据结构查询右端点。当左端点越过了某数后，就将其下一次出现的位置加入到结构中。整体为$O(n\lg n)$。

如果题目的范围小一点，可以莫队。

## 代码

没上快读T了发。

```cpp
#include <vector>
#include <algorithm>
#include <iostream>
#include <stack>
#include <unordered_map>
using namespace std;
const int XN=1000010;
const int FTXN=4000010;
namespace IO {
    const int IN=1e6;
    char in[IN],*ip=in,*ie=in;
#define getchar() (ip==ie && (ie=(ip=in)+fread(in,1,IN,stdin),ip==ie)?EOF:*ip++)
    struct Istream {
        template <class T>
        Istream &operator >>(T &x) {
            static char ch;static bool neg;
            for(ch=neg=0;ch<'0' || '9'<ch;neg|=ch=='-',ch=getchar());
            for(x=0;'0'<=ch && ch<='9';(x*=10)+=ch-'0',ch=getchar());
            x=neg?-x:x;
            return *this;
        }
    }fin;

    const int OUT=1e6;
    char out[OUT],*op=out,*oe=out+OUT;
#define flush() fwrite(out,1,op-out,stdout)
#define putchar(x) ((op==oe?(flush(),op=out,*op++):*op++)=(x))
    struct Ostream {
        ~Ostream() {
            flush();
        }
        template <class T>
        Ostream &operator <<(T x) {
            x<0 && (putchar('-'),x=-x);
            static char stack[233];static int top;
            for(top=0;x;stack[++top]=x%10+'0',x/=10);
            for(top==0 && (stack[top=1]='0');top;putchar(stack[top--]));
            return *this;
        }
        Ostream &operator <<(char ch) {
            putchar(ch);
            return *this;
        }
    }fout;
}

using IO::fin;
using IO::fout;

int ft[FTXN];
int lowbit(int x){
    return x&-x;
}
void ftadd(int pos,int x){
    for(;pos<FTXN;pos+=lowbit(pos)){
        ft[pos]^=x;
    }
}
int ftget(int pos){
    int res=0;
    for(;pos;pos-=lowbit(pos))res^=ft[pos];
    return res;
}

int num[XN];
int xo[XN];
struct Q{
    int l,r;
    int i;
} qs[XN];
int ans[XN];
unordered_map<int,int> curpos;
int nxt[XN];
int main(){
    int n;fin>>n;
    //cout<<"st"<<endl;
    for(int i=1;i<=n;i++)fin>>num[i];
    for(int i=0;i<=n;i++)xo[i]=xo[i-1]^num[i];
    //cout<<"done"<<endl;
    for(int i=n;i>=1;i--){
        nxt[i]=curpos[num[i]];
        curpos[num[i]]=i;
    }
//    for(int i=1;i<=n;i++){
//        cout<<nxt[i]<<" ";
//    }
//    cout<<endl;

    int qlen;fin>>qlen;
    for(int i=0;i<qlen;i++){
        fin>>qs[i].l>>qs[i].r;
        qs[i].i=i;
    }


    for(auto [x,pos]:curpos){
        ftadd(pos,x);
    }

    sort(qs,qs+qlen,[](const Q &a,const Q &b){
        return a.l<b.l;
    });

    int la=1;

    //cout<<"done"<<endl;

    for(int i=0;i<qlen;i++){
        Q &q=qs[i];
        while(q.l>la){
            ftadd(la,num[la]);
            //cout<<"remove "<<la<<endl;
            if(nxt[la]){
                ftadd(nxt[la],num[la]);
                //cout<<"add "<<nxt[la]<<endl;
            }
            la++;
        }
        ans[q.i]=ftget(q.r)^(xo[q.r]^xo[q.l-1]);
    }
    for(int i=0;i<qlen;i++)fout<<ans[i]<<'\n';

    return 0;
}
```


