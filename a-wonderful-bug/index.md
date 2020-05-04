# 记一个bug (HDU 6703)

You are given an array a1,a2,...,an(∀i∈[1,n],1≤ai≤n). Initially, each element of the array is **unique**. 

Moreover, there are m instructions. 

Each instruction is in one of the following two formats: 

1. (1,pos),indicating to change the value of apos to apos+10,000,000; 
2. (2,r,k),indicating to ask the minimum value which is **not equal** to any ai ( 1≤i≤r ) and **not less ** than k. 

Please print all results of the instructions in format 2. 

## 分析
这题强制在线.首先1操作相当于删除了这个数.

dalao自闭了一会get到了它的正确做法,我就直接拿来用了.

维护一权值线段树,位置i存其在a中出现的位置.那么当1到r区间内出现位置的最大值超过了r,根据鸽巢原理,至少有一个数未被限制.

加上不小于k的条件,就是k到r中,找到最小的一个r,使得它满足上面的条件,输出这个r.

## 代码
一个奇葩的bug...

当使用了fread这种先读完缓冲区再处理的快速读入而删漏了cin时...会显而易见的遇到bug.

但是,因为缓冲区的存在,**小范围数据被快乐的读入了缓冲区,cin并不会实际影响什么**.一旦遇到大范围数据,cin提前读入了接下来的数据,导致第一个缓冲区之外的数据全部出错...

艹.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cstring>
using namespace std;
const int MAXN = 400000;

namespace IO
{
const int MAXSIZE = 1 << 20;
char buf[MAXSIZE], *p1, *p2;
#define gc()                                                                 \
    (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, MAXSIZE, stdin), p1 == p2) \
         ? EOF                                                               \
         : *p1++)
inline int rd()
{
    int x = 0, f = 1;
    char c = gc();
    while (!isdigit(c))
    {
        if (c == '-')
            f = -1;
        c = gc();
    }
    while (isdigit(c))
        x = x * 10 + (c ^ 48), c = gc();
    return x * f;
}
char pbuf[1 << 20], *pp = pbuf;
inline void push(const char &c)
{
    if (pp - pbuf == 1 << 20)
        fwrite(pbuf, 1, 1 << 20, stdout), pp = pbuf;
    *pp++ = c;
}
inline void write(int x)
{
    static int sta[35];
    int top = 0;
    do
    {
        sta[top++] = x % 10, x /= 10;
    } while (x);
    while (top)
        push(sta[--top] + '0');
}
} // namespace IO
///////////////////////////////////////////////////////////////////////////////////////

int dat[MAXN];
int a[MAXN];
int lc[MAXN], rc[MAXN], idx = 0;

inline int imax(int a,int b){
    if(a>b)return a;
    return b;
}
void collect(int n)
{
    dat[n] = max(dat[lc[n]], dat[rc[n]]);
}

int build(int &n, int l, int r)
{
    if (!n)
        n = ++idx;
    dat[n] = a[l];
    if (l == r)
        return dat[n];
    int mid = (l + r) / 2;
    return dat[n] = imax(build(lc[n], l, mid), build(rc[n], mid + 1, r));
}

void modify(int x, int l, int r, int L, int R, int n)
{
    if (l <= L && R <= r)
    {
        dat[n] = x;
        return;
    }
    int mid = (L + R) / 2;
    if (l <= mid)
        modify(x, l, r, L, mid, lc[n]);
    if (mid < r)
        modify(x, l, r, mid + 1, R, rc[n]);

    collect(n);
}

int query(int l, int r,int target, int L, int R, int n)
{
    if(L>r || R<l || dat[n]<=target)return -1;
    if(L==R)return L;

    int mid = (L + R) / 2;
    int res=query(l,r,target,L,mid,lc[n]);
    return ~res?res:query(l,r,target,mid+1,R,rc[n]);
}
int root;
int num[MAXN];
int main()
{
    int kase=IO::rd();
    while (kase--)
    {
        int nlen, qlen;
        nlen=IO::rd();
        qlen=IO::rd();
        for (int i = 1; i <= nlen; i++)
        {
            num[i] = IO::rd();
            a[num[i]] = i;
        }
        build(root, 1, nlen);
        int lastans = 0;
        while (qlen--)
        {
            int opt=IO::rd();
            if (opt == 1)
            {
                int pos=IO::rd();
                pos ^= lastans;
                if(num[pos]==0 || num[pos]>nlen)continue;
                modify(0x3f3f3f3f, num[pos], num[pos], 1, nlen, root);
            }
            else
            {
                int t2, t3;
                t2=IO::rd();
                t3=IO::rd();
                //cin >> t2 >> t3;
                int r = t2 ^ lastans, k = t3 ^ lastans;

                int t=query(k,nlen,r,1,nlen,root);
                cout<<(lastans=(~t?t:nlen+1))<<endl;
            }
        }
    }
    return 0;
}
```
