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

First, let's look at the data constraints provided in the problem:

* Length of S <= 100,000
* Total N <= 1,000,000,000

From this, we can deduce that our code's time complexity should be around **O(N)**.

The core challenge of this problem is handling **nested brackets** and **number multipliers**. Brackets in a chemical formula can be nested infinitely, and a number following a bracket multiplies the count of all atoms inside that bracket block.

Whenever we encounter a "nested bracket" problem, the fundamental logic is an **inside-out** process: we must calculate the total count of the innermost bracket first, and then pass that result back to the outer layer.

We generally use two standard approaches for this: **Stack** and **Recursion**.

---

## Approach 1: Stack

If you are not familiar with what a stack is, feel free to Google it or ask your friendly AI assistant! (⁠ ⁠╹⁠▽⁠╹⁠ ⁠)

* [https://en.wikipedia.org/wiki/Stack_(abstract_data_type)](https://www.google.com/search?q=https://en.wikipedia.org/wiki/Stack_(abstract_data_type))
* Other standard data structure resources...

We can imagine the string as an array and create a `cnt` array to record the "number of atoms represented at each position". A stack has a "Last-In-First-Out (LIFO)" property, which makes it absolutely perfect for finding the most recent left bracket `(` that matches a given right bracket `)`.

### Stack Logic Breakdown

1. **Uppercase letters:** Represents a new atom. We record the count at the current position as 1 and mark it as an "entity" that can be multiplied later (by updating our `idx` tracker).
2. **Lowercase letters:** These are attached to the preceding uppercase letter. They don't create a new atom type, so the count at this current position is 0.
3. **Left bracket `(`:** Marks the beginning of a new scope. We push the current index into the stack.
4. **Right bracket `)`:** Marks the end of the innermost scope. We pop the corresponding left bracket index from the top of the stack, sum up all the atomic counts between these two brackets, store this sum at the current right bracket's position, and reset the values in that intermediate range to zero. Finally, we update `idx` because this entire bracket block can act as an entity multiplied by a trailing number.
5. **Numbers `0-9`:** Read the number and multiply the count of the previous "entity" (which could be an uppercase letter or a right bracket, located at `idx`) by this multiplier.

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long; // Use long long uniformly to prevent overflow and save typing

int main() {
    ios::sync_with_stdio(0); cin.tie(0); cout.tie(0); // Fast I/O
    
    string s;
    cin >> s;
    
    vector <ll> cnt((ll)s.length(), 0);
    stack <ll> st;
    ll idx = -1; 
    
    // s.length() returns an unsigned integer type. 
    // We cast it to long long (ll) to safely compare it with our loop variable.
    for (ll i = 0; i < (ll)s.length(); i++) { 
        char c = s[i];
        
        if (c <= 'z' && c >= 'a') {
            cnt[i] = 0; 
        } else if (c <= 'Z' && c >= 'A') {
            cnt[i] = 1;
            idx = i; 
        } else if (c == '(') {
            st.push(i);
        } else if (c == ')') {
            ll temp = st.top();
            ll t = 0;
            st.pop();
            
            for (ll j = temp; j < i; j++) {
                t += cnt[j];
                cnt[j] = 0;
            } 
            
            cnt[i] = t;
            idx = i;
        } else if (c <= '9' && c >= '0') {
            ll to = c - '0';
            if (idx != -1) {
                cnt[idx] *= to;
            }
            cnt[i] = 0; 
        }
    }
    
    ll c = 0;
    for (ll i = 0; i < (ll)s.length(); i++) {
        c += cnt[i];
    }
    
    cout << c << '\n';
    return 0;
}

```

---

## Approach 2: Recursion

The structure inside a pair of brackets is exactly the same as the structure of the overall chemical formula. Because of this, we can easily treat "calculating the value inside the brackets" as a smaller **subproblem** and hand it over to a recursive function.

### Recursion Logic Breakdown

We design a `parse()` function and use a pointer `i` to scan through the string:

1. **Uppercase letters:** The base count is recorded as 1.
2. **Lowercase letters:** These don't affect the count, so we just skip over them.
3. **Left bracket `(`:** We are entering a substructure. We **recursively call `parse()**` to get the total number of atoms inside this new bracket pair.
4. **Right bracket `)`:** The current substructure has ended. We **return** the accumulated total of atoms in this current layer back to the outer layer.
5. **Numbers:** We take the "previously parsed count" (which might be a single atom or a fully evaluated bracket block) and multiply it by this number.

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long; // Use long long uniformly to prevent overflow and save typing

ll parse(const string &s, ll &i) {
    ll total = 0;
    ll last_val = 0; 
    
    // s.length() returns an unsigned integer type. 
    // We cast it to long long (ll) to safely compare it with our pointer.
    while (i < (ll)s.length()) { 
        char c = s[i];
        
        if (c <= 'Z' && c >= 'A') {
            total += last_val; 
            last_val = 1;       
            i++;
        } else if (c <= 'z' && c >= 'a') {
            i++;                
        } else if (c == '(') {
            total += last_val;  
            i++;                
            last_val = parse(s, i); 
        } else if (c == ')') {
            total += last_val;  
            i++;                
            return total;       
        } else if (c <= '9' && c >= '0') {
            ll to = c - '0';
            last_val *= to;    
            i++;
        }
    }
    
    total += last_val; 
    return total;
}

int main() {
    ios::sync_with_stdio(0); cin.tie(0); cout.tie(0); // Fast I/O
    
    string s;
    cin >> s;
    
    ll i = 0;
    cout << parse(s, i) << '\n';
    
    return 0;
}

```

---

### Recommended Practice Problems (HKOI Judge)

* **Stack:** [https://judge.hkoi.org/task/X0801](https://judge.hkoi.org/task/X0801)
* **Stack:** [https://judge.hkoi.org/task/X0804](https://judge.hkoi.org/task/X0804)
* **Recursion:** [https://judge.hkoi.org/task/01046](https://judge.hkoi.org/task/01046)

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
### Before start

There are some graph and data structures in this solution. Although some key ideas are taught in this solution, it is highly recommended you learn some knowledge about graphs before you read this solution.

Graph (I) 2026 : [https://assets.hkoi.org/training2026/g-i.pdf](https://assets.hkoi.org/training2026/g-i.pdf)

DFS (HKOI judge) : [https://judge.hkoi.org/task/C800](https://judge.hkoi.org/task/C800)

Data Structures (III) 2026 (Sparse Table) : [https://assets.hkoi.org/training2026/ds-iii.pdf#page=5](https://assets.hkoi.org/training2026/ds-iii.pdf#page=5)

Data Structures (III) 2026 (Segment Tree) : [https://assets.hkoi.org/training2026/ds-iii.pdf#page=35](https://assets.hkoi.org/training2026/ds-iii.pdf#page=35)

Data Structures (IV) 2026 (Lazy Propagation on Segment Tree) : [https://assets.hkoi.org/training2026/ds-iv.pdf#page=25](https://assets.hkoi.org/training2026/ds-iv.pdf#page=25)

Data Structures (IV) 2026 (Binary Search on Segment Tree + Persistent Segment Tree) : [https://assets.hkoi.org/training2026/ds-iv.pdf#page=80](https://assets.hkoi.org/training2026/ds-iv.pdf#page=80)

Graph (V) 2026 : [https://assets.hkoi.org/training2026/g-v.pdf#page=80](https://assets.hkoi.org/training2026/g-v.pdf#page=80)

Sparse Table (HKOI judge) : [https://judge.hkoi.org/task/B110](https://judge.hkoi.org/task/B110)

Segment Tree (Point Update, Range Query) (HKOI judge) : [https://judge.hkoi.org/task/B111](https://judge.hkoi.org/task/B111)

Segment Tree (Range Maximum Query) (HKOI judge) : [https://judge.hkoi.org/task/M0921](https://judge.hkoi.org/task/M0921)

Lazy Propagation on Segment Tree (HKOI judge) : [https://judge.hkoi.org/task/A101](https://judge.hkoi.org/task/A101)

Persistent Segment Tree (HKOI judge) : [https://judge.hkoi.org/task/A103](https://judge.hkoi.org/task/A103)

Heavy-light Decomposition (HKOI judge) : [https://judge.hkoi.org/task/A222](https://judge.hkoi.org/task/A222)

---

### Core Concept: What are we solving for?

In the game, you travel from a starting point $S$ to a destination $T$. Every room on this path contains a monster with a level equal to the room number $i$. You must consume your level $i$ sword to defeat it.

**This means: If room $i$ is on the path from $S$ to $T$, your sword of level $i$ will be consumed.**

Therefore, the "highest sword level" the player can keep is simply the **largest room number that is NOT on the path from $S$ to $T$.** If all rooms are on the path, we output 0.

---

### Subtask 2 $M = N-1$ (Graph is a Chain)

The graph is a chain. It looks just like a single straight line.

Here is an example of the graph in this subtask ($N = 4, M = 3$):

```text
3 4
4 2
2 1

```

The physical graph structure connects the nodes like this:
`3 - 4 - 2 - 1`

Because a chain only has one simple path between any two nodes, we can treat the entire graph like an array. However, the node IDs given in the input are not in a neat order. Therefore, we can use a single DFS traversal to flatten the graph into a neat 1D array.

We will create two helper arrays:

* `arr[i]`: Tells us the actual room ID at the $i$-th position on the chain.
* `pos[u]`: Tells us the 1D position (index) of room ID $u$.

```cpp
void dfs(int u, int p, int d) {
    arr[d] = u;
    pos[u] = d;
    for (int v : adj[u]) {
        if (v != p) {
            dfs(v, u, d + 1);
        }
    }
}

```

After conversion, if we print `arr` from index 1 to 4, it looks like this: `3, 4, 2, 1`.

Once flattened, any path from a starting room $S$ to a target room $T$ simply covers a continuous segment on our 1D array. We have two main approaches to solve this subtask.

---

#### Approach 1: Range Maximum Query (Sparse Table / Segment Tree)

Assume we want to travel from $S$ to $T$. Let's look at their positions: `pos[S]` and `pos[T]`.
To make things easier, if `pos[S] > pos[T]`, we can swap them so that $S$ always appears before $T$ on our chain (meaning `pos[S] < pos[T]`).

The path from $S$ to $T$ consumes all swords in the range `[pos[S], pos[T]]`. This implies the surviving, unused swords belong to the remaining two parts of the chain:

1. The left part: `[1, pos[S] - 1]`
2. The right part: `[pos[T] + 1, N]`

To find the maximum sword we can keep, we just need to find the **maximum room ID** across these two segments! We can do this efficiently using a Sparse Table or a Segment Tree. We must also carefully handle cases where these segments are empty (e.g., if $S$ is at the very beginning of the chain, the left part does not exist).

**Sparse Table Code:**

```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m, q;
vector<int> adj[200100];
int arr[200100], pos[200100];
int st[200100][20];

void dfs(int u, int p, int d) {
    arr[d] = u;
    pos[u] = d;
    for (int v : adj[u]) {
        if (v != p) {
            dfs(v, u, d + 1);
        }
    }
}

void build_st() {
    for (int i = 1; i <= n; i++) {
        st[i][0] = arr[i];
    }
    for (int j = 1; (1 << j) <= n; j++) {
        for (int i = 1; i + (1 << j) - 1 <= n; i++) {
            st[i][j] = max(st[i][j - 1], st[i + (1 << (j - 1))][j - 1]);
        }
    }
}

int query_st(int l, int r) {
    if (l > r) {
        return 0;
    }
    int k = log2(r - l + 1);
    return max(st[l][k], st[r - (1 << k) + 1][k]);
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    cin >> n >> m >> q;
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    
    // Find one endpoint of the chain to start DFS
    int start_node = 1;
    for (int i = 1; i <= n; i++) {
        if (adj[i].size() == 1) {
            start_node = i;
            break;
        }
    }
    
    dfs(start_node, 0, 1);
    build_st();
    
    while (q--) {
        int s, t;
        cin >> s >> t;
        int p_s = pos[s];
        int p_t = pos[t];
        if (p_s > p_t) {
            swap(p_s, p_t);
        }
        
        int ans = 0;
        if (p_s > 1) {
            ans = max(ans, query_st(1, p_s - 1));
        }
        if (p_t < n) {
            ans = max(ans, query_st(p_t + 1, n));
        }
        cout << ans << "\n";
    }
    return 0;
}

```

**Time Complexity (Sparse Table):**

* Precompute: $O(N \log N)$
* Per query: $O(1)$
* Overall: $O(N \log N + Q)$

**Segment Tree Code:**
*(The logic is the same, but we use a segment tree for the range maximum queries.)*

```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m, q;
vector<int> adj[200100];
int arr[200100], pos[200100];
int tree[800100];

void dfs(int u, int p, int d) {
    arr[d] = u;
    pos[u] = d;
    for (int v : adj[u]) {
        if (v != p) {
            dfs(v, u, d + 1);
        }
    }
}

void build(int idx, int l, int r) {
    if (l == r) {
        tree[idx] = arr[l];
        return;
    }
    int mid = l + (r - l) / 2;
    build(idx * 2, l, mid);
    build(idx * 2 + 1, mid + 1, r);
    tree[idx] = max(tree[idx * 2], tree[idx * 2 + 1]);
}

int query(int idx, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) {
        return tree[idx];
    }
    int mid = l + (r - l) / 2;
    int res = 0;
    if (ql <= mid) {
        res = max(res, query(idx * 2, l, mid, ql, qr));
    }
    if (qr > mid) {
        res = max(res, query(idx * 2 + 1, mid + 1, r, ql, qr));
    }
    return res;
}

int main() {
    // Input and DFS logic is identical to Sparse Table above...
    // ...
    build(1, 1, n);
    while (q--) {
        int s, t;
        cin >> s >> t;
        int p_s = pos[s];
        int p_t = pos[t];
        if (p_s > p_t) {
            swap(p_s, p_t);
        }
        
        int ans = 0;
        if (p_s > 1) {
            ans = max(ans, query(1, 1, n, 1, p_s - 1));
        }
        if (p_t < n) {
            ans = max(ans, query(1, 1, n, p_t + 1, n));
        }
        cout << ans << "\n";
    }
    return 0;
}

```

**Time Complexity (Segment Tree):**

* Precompute: $O(N)$
* Per query: $O(\log N)$
* Overall: $O(N + Q \log N)$

---

#### Approach 2: Binary Search on Persistent Segment Tree

In this approach, instead of storing the maximum room IDs in the array positions, we use a **Persistent Segment Tree (PST)**.
Version $i$ of the tree will keep track of the frequencies of all sword levels (room IDs) present in the chain from position 1 up to position $i$.

When querying the path from `pos[S]` to `pos[T]`, we want to find the **maximum sword level** that has an occurrence count of 0 inside this range. By traversing down the Persistent Segment Tree and checking the right child (larger sword levels) first, we can find this answer in $O(\log N)$ time.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int ls, rs, cnt;
};

int n, m, q;
vector<int> adj[200100];
int arr[200100], pos[200100];
int root[200100];
Node tree[10000100];
int tot = 0;

void dfs(int u, int p, int d) {
    arr[d] = u;
    pos[u] = d;
    for (int v : adj[u]) {
        if (v != p) {
            dfs(v, u, d + 1);
        }
    }
}

void update(int& idx, int pre, int l, int r, int val) {
    tot++;
    idx = tot;
    tree[idx] = tree[pre];
    if (l == r) {
        tree[idx].cnt++;
        return;
    }
    int mid = l + (r - l) / 2;
    if (val <= mid) {
        update(tree[idx].ls, tree[pre].ls, l, mid, val);
    } else {
        update(tree[idx].rs, tree[pre].rs, mid + 1, r, val);
    }
    tree[idx].cnt = tree[tree[idx].ls].cnt + tree[tree[idx].rs].cnt;
}

int query(int idx_r, int idx_l, int l, int r) {
    int current_cnt = tree[idx_r].cnt - tree[idx_l].cnt;
    if (current_cnt == r - l + 1) {
        // Every sword in this range is consumed on the path
        return 0; 
    }
    if (current_cnt == 0) {
        // No swords in this range are consumed, we can safely pick the largest one
        return r;
    }
    int mid = l + (r - l) / 2;
    
    // We want the MAXIMUM possible sword, so check the right side first
    int res = query(tree[idx_r].rs, tree[idx_l].rs, mid + 1, r);
    if (res == 0) {
        // If nothing is available on the right, check the left side
        res = query(tree[idx_r].ls, tree[idx_l].ls, l, mid);
    }
    return res;
}

int main() {
    // ... Input and DFS logic same as above ...
    
    for (int i = 1; i <= n; i++) {
        update(root[i], root[i - 1], 1, n, arr[i]);
    }
    
    while (q--) {
        int s, t;
        cin >> s >> t;
        int p_s = pos[s];
        int p_t = pos[t];
        if (p_s > p_t) {
            swap(p_s, p_t);
        }
        
        cout << query(root[p_t], root[p_s - 1], 1, n) << "\n";
    }
    return 0;
}

```

**Time Complexity (Persistent Segment Tree):**

* Precompute: $O(N \log N)$
* Per query: $O(\log N)$
* Overall: $O(N \log N + Q \log N)$

---

### Subtask 3 $M = N - 1$ (Graph is a Tree)

The graph is now a standard Tree. The simple path from $S$ to $T$ is still unique, but the graph is no longer a single straight line.
However, we can still use Approach 2 from Subtask 2!

We will use a technique called **Heavy-Light Decomposition (HLD)**. HLD cleverly cuts any tree into a collection of straight chains. The beautiful property of HLD is that any path between two nodes $u$ and $v$ can be broken down into at most $O(\log N)$ continuous segments across these chains.

Once we break the query path into $O(\log N)$ segments, we can query our Persistent Segment Tree just like before. Instead of checking a single interval `[L, R]`, we sum up the occurrences of swords across *all* the $O(\log N)$ intervals simultaneously as we traverse down the PST.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int ls, rs, cnt;
};

int n, m, q;
vector<int> adj[200100];
int si[200100], fa[200100], son[200100], dep[200100];
int timer = 0, id[200100], top[200100], rid[200100];
int root[200100];
Node tree[10000100];
int tot = 0;

void dfs1(int u, int pre, int d) {
    si[u] = 1;
    fa[u] = pre;
    dep[u] = d;
    int maxsi = -1;
    for (int v : adj[u]) {
        if (v == pre) {
            continue;
        }
        dfs1(v, u, d + 1);
        si[u] += si[v];
        if (si[v] > maxsi) {
            son[u] = v;
            maxsi = si[v];
        }
    }
}

void dfs2(int u, int t) {
    timer++;
    top[u] = t;
    id[u] = timer;
    rid[timer] = u; // reverse mapping: DFS time to actual node ID
    if (!son[u]) {
        return;
    }
    dfs2(son[u], t);
    for (int v : adj[u]) {
        if (v == son[u] || v == fa[u]) {
            continue;
        }
        dfs2(v, v);
    }
}

void update(int& idx, int pre, int l, int r, int val) {
    tot++;
    idx = tot;
    tree[idx] = tree[pre];
    if (l == r) {
        tree[idx].cnt++;
        return;
    }
    int mid = l + (r - l) / 2;
    if (val <= mid) {
        update(tree[idx].ls, tree[pre].ls, l, mid, val);
    } else {
        update(tree[idx].rs, tree[pre].rs, mid + 1, r, val);
    }
    tree[idx].cnt = tree[tree[idx].ls].cnt + tree[tree[idx].rs].cnt;
}

int query_path(vector<pair<int, int>>& roots, int l, int r) {
    if (l == r) {
        return l;
    }
    int mid = l + (r - l) / 2;
    int right_cnt = 0;
    
    // Sum up occurrences in the right child for all HLD segments
    for (int i = 0; i < roots.size(); i++) {
        right_cnt += tree[tree[roots[i].second].rs].cnt - tree[tree[roots[i].first].rs].cnt;
    }
    
    int right_capacity = r - mid;
    if (right_cnt < right_capacity) {
        // We have missing swords on the right side, move right
        vector<pair<int, int>> next_roots;
        for (int i = 0; i < roots.size(); i++) {
            next_roots.push_back({tree[roots[i].first].rs, tree[roots[i].second].rs});
        }
        return query_path(next_roots, mid + 1, r);
    } else {
        // Right side is full, we must move left
        vector<pair<int, int>> next_roots;
        for (int i = 0; i < roots.size(); i++) {
            next_roots.push_back({tree[roots[i].first].ls, tree[roots[i].second].ls});
        }
        return query_path(next_roots, l, mid);
    }
}

int get_ans(int u, int v) {
    vector<pair<int, int>> intervals;
    int total_nodes_on_path = 0;
    
    // Break the path into continuous intervals using HLD top arrays
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) {
            swap(u, v);
        }
        intervals.push_back({id[top[u]], id[u]});
        total_nodes_on_path += (id[u] - id[top[u]] + 1);
        u = fa[top[u]];
    }
    if (dep[u] > dep[v]) {
        swap(u, v);
    }
    intervals.push_back({id[u], id[v]});
    total_nodes_on_path += (id[v] - id[u] + 1);
    
    // If the path length is strictly N, all rooms are visited
    if (total_nodes_on_path == n) {
        return 0;
    }
    
    vector<pair<int, int>> roots;
    for (int i = 0; i < intervals.size(); i++) {
        int l = intervals[i].first;
        int r = intervals[i].second;
        roots.push_back({root[l - 1], root[r]});
    }
    
    return query_path(roots, 1, n);
}

int main() {
    ios::sync_with_stdio(0);
    cin.tie(0);
    cin >> n >> m >> q;
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    
    dfs1(1, 0, 1);
    dfs2(1, 1);
    
    // Insert actual room IDs by their HLD mapped time
    for (int i = 1; i <= n; i++) {
        update(root[i], root[i - 1], 1, n, rid[i]);
    }
    
    while (q--) {
        int s, t;
        cin >> s >> t;
        cout << get_ans(s, t) << "\n";
    }
    
    return 0;
}

```

**Time Complexity (HLD + Persistent Segment Tree):**

* Precompute: $O(N \log N)$
* Per query: $O(\log^2 N)$ (Because HLD provides $O(\log N)$ segments, and iterating through them while traversing the $O(\log N)$ tree depth gives $O(\log^2 N)$).
* Overall: $O(N \log N + Q \log^2 N)$

---

### Practice Questions
- [M2243 Yet Another RMQ(https://judge.hkoi.org/task/M2243)](https://judge.hkoi.org/task/M2243)
