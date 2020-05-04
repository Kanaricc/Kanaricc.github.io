# [CF 1321C] Remove Adjacent 拓展

没注意到数据范围是100。

You are given a string 𝑠 consisting of lowercase Latin letters. Let the length of 𝑠 be |𝑠|. You may perform several operations on this string.

In one operation, you can choose some index 𝑖 and remove the 𝑖-th character of 𝑠 (𝑠𝑖) if at least one of its adjacent characters is the previous letter in the Latin alphabet for 𝑠𝑖. For example, the previous letter for b is a, the previous letter for s is r, the letter a has no previous letters. Note that after each removal the length of the string decreases by one. So, the index 𝑖 should satisfy the condition 1≤𝑖≤|𝑠| during each operation.

For the character 𝑠𝑖 adjacent characters are 𝑠𝑖−1 and 𝑠𝑖+1. The first and the last characters of 𝑠 both have only one adjacent character (unless |𝑠|=1).

Consider the following example. Let 𝑠= bacabcab.

During the first move, you can remove the first character 𝑠1= b because 𝑠2= a. Then the string becomes 𝑠= acabcab.
During the second move, you can remove the fifth character 𝑠5= c because 𝑠4= b. Then the string becomes 𝑠= acabab.
During the third move, you can remove the sixth character 𝑠6='b' because 𝑠5= a. Then the string becomes 𝑠= acaba.
During the fourth move, the only character you can remove is 𝑠4= b, because 𝑠3= a (or 𝑠5= a). The string becomes 𝑠= acaa and you cannot do anything with it.
Your task is to find the maximum possible number of characters you can remove if you choose the sequence of operations optimally.

## 分析

换个视角来看消除，当有b时，我们就能保证所有能接触到这个b的c被消除。

所以从大往小了做，枚举每一个字母$s$，然后枚举s的位置去查询它左右是否有能够碰到它的$s+1$。

接下来是如何维护信息。当上一个字母使得上上个字母能够被消除后，就有可能使得原本无法接触到本次字母的上一个字母转而能够接触到该字母。使用并查集来做这种事情，并查集维护连续消除的区间l和r，以及这个区间存在字母（int状压即可）。

## 代码

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include <map>
using namespace std;
using pii=pair<int,int>;
using ll=long long;
const int MAXN=1000;

struct UT {
    int fa[MAXN];
    int dat[MAXN];
    int l[MAXN], r[MAXN];

    UT() {
        for (int i = 0; i < MAXN; i++)fa[i] = i;
        for (int i = 0; i < MAXN; i++)l[i] = r[i] = i;
    }

    int find(int u) {
        return fa[u] == u ? u : fa[u] = find(fa[u]);
    }

    void con(int u, int v) {
        int uf = find(u), vf = find(v);
        dat[vf] |= dat[uf];
        l[vf] = min(l[vf], l[uf]);
        r[vf] = max(r[vf], r[uf]);
        fa[uf] = vf;
    }

    bool isc(int u, int v) {
        return find(u) == find(v);
    }
};
string inp;
vector<int> pos[30];
UT ut;
bool vis[MAXN];
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    int nlen;
    cin >> nlen;
    cin >> inp;
    inp = " " + inp;
    for (int i = 1; i <= nlen; i++)
        pos[inp[i] - &#039;a&#039;].push_back(i);
    for (int i = 1; i <= nlen; i++)
        ut.dat[i] |= (1 << (inp[i] - &#039;a&#039;));

    for (int i = 25; i >= 0; i--) {
        for (auto p:pos[i]) {
            if (p - 1 >= 1) {
                if ((ut.dat[ut.find(p - 1)] & (1 << (i + 1))) || (ut.dat[ut.find(p - 1)] & (1 << i))) {
                    ut.con(p - 1, p);
                }
            }
            if (p + 1 <= inp.size()) {
                if ((ut.dat[ut.find(p + 1)] & (1 << (i + 1))) || (ut.dat[ut.find(p + 1)] & (1 << i))) {
                    ut.con(p, p + 1);
                }
            }
        }
    }
    int ans = 0;
    for (int i = 1; i <= nlen; i++) {
        int uf = ut.find(i);
        if (!vis[uf]) {
            vis[uf] = 1;

            int minn = 0;
            for (int i = 0; i < 26; i++) {
                if (ut.dat[uf] & (1 << i)) {
                    minn = i;
                    break;
                }
            }
            int off = 0;
            for (int i = ut.l[uf]; i <= ut.r[uf]; i++) {
                off += (inp[i] == &#039;a&#039; + minn);
            }

            ans += max(0, ut.r[uf] - ut.l[uf] + 1 - off);
        }
    }
    cout << ans << endl;


    return 0;
}
```

## Extra

这场比赛的B题挺好玩的…要求选出的数字满足

<div>$$c_i-c_{i+1}=p_{c_i}-p_{c_{i+1}}$$</div>

…一上来傻了不知道咋做，其实只要把公式左右交换一下就完事了。
