# [HDU 6704] Kth Occurrence

You are given a string S consisting of only lowercase english letters and some queries.

For each query (l,r,k), please output the starting position of the k-th occurence of the substring SlSl+1...Sr in S.

## 分析
第一个问题是快速找出所有出现的子串的位置,可以使用后缀数组.这些字串出现在sa的一个连续的区间中.

<!--more-->

第二个问题是找出这些出现位置中的第k大,可以使用主席树,以sa建树.


## 代码
思路清晰,但这代码它不好写

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cstring>
#include <string>
#include <cstdlib>
#include <cmath>
using namespace std;
const int MAXN=100060;
using ull=unsigned long long;

int n;
int sa[MAXN], x[MAXN], c[MAXN], y[MAXN];
char a[MAXN];

inline void SA()
{
    int m = 128;
    for (int i = 0; i <= m; i++)
        c[i] = 0;
    for (int i = 1; i <= n; i++)
        c[x[i]]++;
    for (int i = 1; i <= m; i++)
        c[i] += c[i - 1];
    for (int i = n; i; i--)
        sa[c[x[i]]--] = i;

    for (int k = 1, p; k <= n; k <<= 1)
    {
        p = 0;
        for (int i = n; i > n - k; i--)
            y[++p] = i;
        for (int i = 1; i <= n; i++)
            if (sa[i] > k)
                y[++p] = sa[i] - k;

        for (int i = 0; i <= m; i++)
            c[i] = 0;
        for (int i = 1; i <= n; i++)
            c[x[i]]++;
        for (int i = 1; i <= m; i++)
            c[i] += c[i - 1];
        for (int i = n; i; i--)
            sa[c[x[y[i]]]--] = y[i];

        p = y[sa[1]] = 1;
        for (int i = 2, a, b; i <= n; i++)
        {
            a = sa[i] + k > n ? -1 : x[sa[i] + k];
            b = sa[i - 1] + k > n ? -1 : x[sa[i - 1] + k];
            y[sa[i]] = (x[sa[i]] == x[sa[i - 1]]) && (a == b) ? p : ++p;
        }
        swap(x, y);
        m = p;
    }
}

int tot;
int sum[(MAXN << 5) + 10], rt[MAXN + 10], ls[(MAXN << 5) + 10],
    rs[(MAXN << 5) + 10];

int build(int l, int r) //建树
{
    int root = ++tot;
    if (l == r)
        return root;
    int mid = l + r >> 1;
    ls[root] = build(l, mid);
    rs[root] = build(mid + 1, r);
    return root; //返回该子树的根节点
}
int update(int k, int l, int r, int root) //插入操作
{
    int dir = ++tot;
    ls[dir] = ls[root], rs[dir] = rs[root], sum[dir] = sum[root] + 1;
    if (l == r)
        return dir;
    int mid = l + r >> 1;
    if (k <= mid)
        ls[dir] = update(k, l, mid, ls[dir]);
    else
        rs[dir] = update(k, mid + 1, r, rs[dir]);
    return dir;
}
//left root, right root, querying l,r, the k-th
int query(int u, int v, int l, int r, int k) //查询操作
{
    int mid = l + r >> 1,
        x = sum[ls[v]] - sum[ls[u]]; //通过区间减法得到左儿子的信息
    if (l == r){
        return l;
    }
    if (k <= x) //说明在左儿子中
        return query(ls[u], ls[v], l, mid, k);
    else //说明在右儿子中
        return query(rs[u], rs[v], mid + 1, r, k - x);
}

int height[MAXN];
int st[20][MAXN];
inline void get_height() {
    int k = 0;
    for (int i = 1; i <= n; ++i) {
        if (x[i] == 1) continue;
        if (k) --k;
        int j = sa[x[i] - 1];
        while (j + k <= n && i + k <= n && a[i + k] == a[j + k]) ++k;
        height[x[i]] = k;
    }
}
void build_st() {
    for (int i = 1; i <= n; i++) st[0][i] = height[i];
    for (int k = 1; k <= 19; k++) {
        for (int i = 1; i + (1 << k) - 1 <= n; i++) {
            st[k][i] = min(st[k - 1][i], st[k - 1][i + (1 << k - 1)]);
        }
    }
}
int lcp(int ll, int rr) {
    int l = x[ll], r = x[rr];
    if (l > r) swap(l, r);
    if (l == r) return n - sa[l]+1;
    int t = log2(r - l);
    return min(st[t][l + 1], st[t][r - (1 << t) + 1]);
}

int main(){
    int kase;cin>>kase;
    while(kase--){
        int nlen,qlen;cin>>nlen>>qlen;
        scanf("%s",a+1);
        for(int i=0;i<MAXN;i++)x[i]=a[i];
        n=nlen;
        SA();
        n=nlen;

        get_height();
        build_st();
        
        /*
        for(int i=1;i<=nlen;i++)cout<<sa[i]<<" ";
        cout<<endl;
        for(int i=1;i<=nlen;i++)cout<<x[i]<<" ";
        cout<<endl;
        */

        tot=0;
        memset(sum,0,sizeof(sum));
        rt[0] = build(1, nlen);
        for (int i = 1; i <= n; ++i)
            rt[i] = update(sa[i], 1, nlen, rt[i - 1]);

        while(qlen--){
            int ql,qr,qk;
            scanf("%d%d%d",&ql,&qr,&qk);
            int sublen=qr-ql+1;

            int ex_l,ex_r;
            //binary search
            {
                int l=1,r=x[ql];
                while(r-l>1){
                    int mid=(l+r)/2;
                    if(lcp(sa[mid], ql) >= sublen){
                        r=mid;
                    }else l=mid+1;
                }
                for(l;l<=r;l++){
                    if(lcp(sa[l], ql) >= sublen){
                        ex_l=l;
                        break;
                    }
                }
            }
            {
                int l=x[ql],r=nlen;
                while(r-l>1){
                    int mid=(l+r)/2;
                    if(lcp(sa[mid], ql) >= sublen){
                        l=mid;
                    }else r=mid-1;
                }
                for(r;r>=l;r--){
                    if(lcp(sa[r], ql) >= sublen){
                        ex_r=r;
                        break;
                    }
                }
            }
            //cout<<ex_l<<" "<<ex_r<<endl;
            if(ex_r-ex_l+1<qk){
                cout<<-1<<endl;
            } else cout<<query(rt[ex_l - 1], rt[ex_r], 1, nlen, qk)<<endl;
        }
    }

    return 0;
}
```
