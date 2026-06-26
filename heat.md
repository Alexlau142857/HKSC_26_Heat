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

Stack illustration:


### Stack Logic Breakdown

* **Uppercase letters:** Represents a new atom. We record the count at the current position as 1 and mark it as an "entity" that can be multiplied later (by updating our `idx` tracker).
* **Lowercase letters:** These are attached to the preceding uppercase letter. They don't create a new atom type, so the count at this current position is 0.
* **Left bracket `(`:** Marks the beginning of a new scope. We push the current index into the stack.
* **Right bracket `)`:** Marks the end of the innermost scope. We pop the corresponding left bracket index from the top of the stack, sum up all the atomic counts between these two brackets, store this sum at the current right bracket's position, and reset the values in that intermediate range to zero. Finally, we update `idx` because this entire bracket block can act as an entity multiplied by a trailing number.
* **Numbers `0-9`:** Read the number and multiply the count of the previous "entity" (which could be an uppercase letter or a right bracket, located at `idx`) by this multiplier.

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
