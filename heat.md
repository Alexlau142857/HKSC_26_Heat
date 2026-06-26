# Solution for HKSC Heat Event
- This is not an official solution, and hence there will only be full solution of the task A to C, and partial solution of subtasks of task D & E.
- As most of you just solved the task A, you should focus more on tasks similar to task B and C (HKOI Level C and Level B)

## A Distributing Candies 分發糖果
- Trivial question, as all of you solved, no need to explain
- For some of you may have long code solving this question, here's a easy one-line solution : $\max(0, A - X) + \max(0, B - Y)$
```
#include <bits/stdc++.h>
using namespace std;
#define ll long long
int main(){
    ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
    int a, b, x, y;
    cin >> a >> b >> x >> y;
    cout << max(0, a - x) + max(0, b - y) << endl;
    return 0;
}
```

## B Bright Chemist 光明化學家

```

```

## C Fence 圍欄
- HKOI Level C question
- Required knowledge:
    - 2D partial sum (HKOI C201)
    - Binary Search : a technique very useful in many contest, definitely should do more exercise on this algorithm.
```
#include <bits/stdc++.h>
using namespace std;

#define ll long long

const int maxn = 3010;
const int maxm = 3010;
int seq[maxn][maxm];

int cal(int r1, int c1, int r2, int c2){
    return seq[r2][c2] - seq[r1 - 1][c2] - seq[r2][c1 - 1] + seq[r1 - 1][c1 - 1];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);
    int n, m, q;
    cin >> n >> m >> q;
    memset(seq, 0, sizeof seq);
    for(int i = 1; i <= n; i++){
        string s;
        cin >> s;
        for(int j = 1; j <= m; j++){
            seq[i][j] = s[j - 1] - '0';
        }
    }
    for(int  i = 1; i <= n; i++){
        for(int j = 1; j <= m; j++){
            seq[i][j] += seq[i - 1][j] + seq[i][j - 1] - seq[i - 1][j - 1];
        }
    }
    for(int i = 1; i <= q; i++){
        int r1, c1, r2, c2;
        cin >> r1 >> c1 >> r2 >> c2;
        int curr = cal(r1, c1, r2, c2);
        if(curr == 0){
            cout << "0\n";
            continue;
        }
        while(r1 + 1 <= n and cal(r1 + 1, c1, r2, c2) == curr) r1++; 
        while(c1 + 1 <= m and cal(r1, c1 + 1, r2, c2) == curr) c1++; 
        while(r2 - 1 >= 1 and cal(r1, c1, r2 - 1, c2) == curr) r2--;
        while(c2 - 1 >= 1 and cal(r1, c1, r2, c2 - 1) == curr) c2--;
        cout << 2 * ((r2 - r1 + 1) + (c2 - c1 + 1)) << "\n";
    }
    return 0;
}
```
## D No 67 不可以 67


## E Dungeon 地下城