# [CCPC2019 秦皇岛] Angel Beats

给出一些点，询问对于点$(x,y)$，求其能和已知点形成多少个直角三角形。

## 分析

首先，对于询问的点，直接加入集合离线操作就可以。这道题就变成了单纯的求三角形。一开始我跟求锐角三角形那道题一样做，然后写得又恶心又不知道哪里出了bug改不出来。

后来，实际上可以事先直接枚举2点按照斜率统计边数，对每个查询$lg(n^2)$查询，会省很多事。

之后的就是很显然的：查询点作为直角扫一遍，查询点作为非直角扫一遍。

## 代码

淦。

```cpp
#include <iostream>
#include <map>
#include <vector>
#include <cstring>
using namespace std;
using ll = long long;
const int MAXN = 2010;

struct Point
{
    ll x, y;
    Point(ll x, ll y) : x(x), y(y){};
    Point(){}

    Point normalize() const
    {
        if (x < 0 || (x == 0 && y < 0))
            return Point(-x, -y);
        return *this;
    }

    Point operator-(const Point &other) const
    {
        return Point(x - other.x, y - other.y);
    }
    bool operator<(const Point &other) const
    {
        //将线按照斜率排序，以在map中统计到一起。
        Point a = normalize(), b = other.normalize();
        return a.y * b.x < b.y * a.x;
    }
};
using Vec = Point;

Point origin[MAXN],queries[MAXN];
int ans[MAXN];
map<Vec,int> mp;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);
    int nlen, qlen;
    while (cin >> nlen >> qlen)
    {
        memset(ans,0,sizeof(ans));
        for (int i = 0; i < nlen; i++)
        {
            cin>>origin[i].x>>origin[i].y;
        }
        for (int i = 0; i < qlen; i++)
        {
            cin>>queries[i].x>>queries[i].y;
        }

        for (int i = 0; i < nlen; i++)
        {
            mp.clear();
            for (int j = 0; j < nlen; j++)
            {
                if (i == j)
                    continue;
                Vec temp = origin[j] - origin[i];
                mp[temp]++;
            }
            for (int j = 0; j < qlen; j++)
            {
                Vec temp = queries[j] - origin[i];
                ans[j] += mp[Vec(-temp.y, temp.x)];
            }
        }
        /*
        cout<<"====not===="<<endl;
        for(int i=0;i<qlen;i++){
            cout<<ans[i]<<"\t";
        }
        cout<<endl<<"====not===="<<endl;
        */

        for (int i = 0; i < qlen; i++)
        {
            mp.clear();
            for (int j = 0; j < nlen; j++)
            {
                Vec temp = origin[j] - queries[i];
                ans[i] += mp[Vec(-temp.y, temp.x)];
                mp[temp]++;
            }
        }

        for (int i = 0; i < qlen; i++)
        {
            cout << ans[i] << "\n";
        }
    }

    return 0;
}
```

