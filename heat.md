Solution for HKSC Heat Event

This isn't an official solution, so I'll just provide full solutions for tasks A to C, and partial solutions for some subtasks of D and E.
Since most of you managed to solve task A, I'd suggest focusing more on problems like B and C (around HKOI Level C and Level B).
The solutions for D and E are quite tough — feel free to ask if anything is unclear.

這不是官方題解，所以只會提供 A 到 C 題的完整解答，以及 D 和 E 題部分子任務的解法。因為大部分同學都解出了 A 題，建議大家多花時間在像 B 和 C 這種程度的題目上（大概 HKOI Level C 和 Level B）。D 跟 E 的題解比較難，看不懂的話隨時發問。

---

A Distributing Candies 分發糖果

A really trivial problem, basically everyone solved it, so no deep explanation needed. If you wrote a long solution, here's a clean one-liner that does the job:
$\max(0, A - X) + \max(0, B - Y)$

這題很簡單，大家都解出來了，不用多解釋。如果你寫了很長的程式碼，這裡有一個乾淨的一行解法：
$\max(0, A - X) + \max(0, B - Y)$

```cpp
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


—



B Bright Chemist 光明化學家

Let's start by checking the data constraints given in the problem: $|S| \le 100{,}000$, $N \le 1{,}000{,}000{,}000$. This tells us our code should run in about $O(N)$ time.
先來看一下題目給的數據範圍：$|S| \le 100{,}000$，$N \le 1{,}000{,}000{,}000$。這表示我們的程式時間複雜度大約要落在 $O(N)$。

The main challenge here is dealing with nested brackets and number multipliers. Brackets in a chemical formula can be nested arbitrarily deep, and a number right after a bracket multiplies all the atoms inside that bracket block.
這題的核心難點在於處理嵌套括號和數字倍數。化學式裡的括號可以無限嵌套，而括號後面的數字會把裡面所有原子的數量都乘上去。

When you run into a "nested bracket" problem, the core idea is to work inside-out: figure out the total count for the innermost bracket first, then pass that result back to the outer layer.
遇到這種「嵌套括號」的問題，基本的想法都是從內層往外層處理：先把最內層括號的總數算出來，再把結果交回給外層。

Two standard ways to handle this are Stack and Recursion.
處理這種問題有兩種常見方法：Stack（堆疊）和Recursion（遞迴）。

---

Approach 1: Stack（棧）

If you're not sure what a stack is, you can always Google it or ask a friendly AI assistant!
如果你對 stack 不太熟悉，隨時可以上網搜尋或問問 AI ！

· https://en.wikipedia.org/wiki/Stack_(abstract_data_type)
· (other standard data structure resources...)

Think of the string as an array, and create a cnt array to record "the number of atoms represented at each position". A stack works in a Last-In-First-Out (LIFO) way, which makes it perfect for finding the most recent ( that matches a given ).
我們可以把字串想像成一個陣列，並建立一個 cnt 陣列來記錄「每個位置代表的原子數量」。Stack 具有後進先出 (LIFO) 的特性，非常適合用來找到最近的 ( 與 ) 配對。

Stack Logic Breakdown:

· Uppercase letters: A new atom appears. We record the count at the current position as 1 and mark this spot as an "entity" that can be multiplied later (by updating idx).
    大寫字母： 代表一個新原子。我們把當前位置的數量記為 1，並標記這個位置是「可以被後面數字乘上的單位」（透過更新 idx）。
· Lowercase letters: These just extend the previous uppercase letter, so they don't create a new atom — the count at this position stays 0.
    小寫字母： 只是前面大寫字母的延伸，不產生新原子，當前位置的數量保持 0。
· Left bracket (: Marks the start of a new scope. We push the current index onto the stack.
    左括號 (： 一個新範圍的開始，把當前的索引推入 stack。
· Right bracket ): Marks the end of the innermost scope. We pop the matching left bracket index from the stack, sum up all atomic counts between those two brackets, store that sum at the right bracket's position, and clear the values in between to zero. Then we update idx because this whole bracket block can now act as an entity that a trailing number may multiply.
    右括號 )： 最內層範圍的結束。從 stack 頂端取出對應的左括號索引，把這兩個括號之間所有原子數量加總起來，存到右括號的位置，並把中間那段清成零。然後更新 idx，因為整個括號區塊現在也能被後面的數字乘上。
· Numbers 0-9: Read the digit, then multiply the count of the previous entity (which could be a single uppercase letter or a whole bracket block, pointed to by idx) by that number.
    數字 0-9： 讀取數字，然後把前一個實體（可能是大寫字母或一個完整的括號區塊，由 idx 指向）的數量乘上這個數字。

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

Approach 2: Recursion（遞歸）


The structure inside a pair of brackets is exactly the same as the overall chemical formula. That means we can treat "calculate the value inside the brackets" as a smaller subproblem and hand it to a recursive function.
一對括號裡面的結構，其實跟整個化學式的結構一模一樣。所以我們可以很自然地，把「計算括號內部的值」當成一個較小的子問題，交給遞迴函數去處理。

Recursion Logic Breakdown:

We design a parse() function and use a pointer i to walk through the string:
我們設計一個 parse() 函數，用一個指針 i 掃過字串：

· Uppercase letters: The base count is 1.
    大寫字母： 基本數量就是 1。
· Lowercase letters: They don't affect the count, so we just skip over them.
    小寫字母： 不影響數量，直接跳過。
· Left bracket (: We're entering a new substructure. We recursively call parse() to get the total atom count inside this bracket pair.
    左括號 (： 進入一個子結構，我們遞迴呼叫 parse()，取得這一對括號裡面的原子總數。
· Right bracket ): The current substructure ends. We return the accumulated total of atoms in this layer back to the outer call.
    右括號 )： 當前子結構結束，我們回傳這層累積的原子總數給上一層。
· Numbers: Take the "previously parsed count" (which could be a single atom or a full bracket block) and multiply it by the digit.
    數字： 把「剛剛解析出來的數量」（可能是一個原子或一個完整的括號區塊）乘上這個數字。

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

Recommended Questions 

· Stack: https://judge.hkoi.org/task/X0801
· Stack: https://judge.hkoi.org/task/X0804
· Recursion: https://judge.hkoi.org/task/01046
· Recursion: https://judge.hkoi.org/task/C601 

—



C Fence 圍欄

· HKOI Level C question
    HKOI Level C 的題目
· Required knowledge: 
2D prefix sum (HKOI C201)
Binary Search (a technique very useful in many contests; definitely worth practicing more).

    需要掌握的知識：
二維前綴和 (HKOI C201)
二分搜尋（在很多比賽中都很實用，絕對值得多練習）。

· **Binary search**  ALSO VERY OFTEN ASKED IN DSE ICT !!!
    DSE ICT 也很常考 !!!

---

Data Range and Time Complexity Analysis
數據範圍與時間複雜度分析

First, observe the data ranges given in the problem:
先看一下題目給的數據範圍：

· $1 \le N, M \le 3000$
· $1 \le Q \le 2 \times 10^5$

In a typical programming contest, around $10^8$ basic operations can be performed per second. The time limit for this problem is 1000ms.
一般程式比賽中，一秒大約可以執行 $10^8$ 次基本操作，這題的時間限制是 1000ms。

If we use the most straightforward brute‑force method, scanning the entire rectangle with a double loop for every query, each query would take $O(N \times M)$ in the worst case. The total time complexity would be $O(Q \times N \times M)$. Plugging in the maximum values: $2 \times 10^5 \times 3000 \times 3000 = 1.8 \times 10^{12}$, which will obviously cause a Time Limit Exceeded (TLE) error.
如果我們用最直接的暴力法，每次查詢都用兩層迴圈掃過整個矩形，最壞情況下每次查詢要 $O(N \times M)$，總時間複雜度會是 $O(Q \times N \times M)$。代入最大值：$2 \times 10^5 \times 3000 \times 3000 = 1.8 \times 10^{12}$，這很明顯會 TLE。

To keep the total runtime within 1000ms, the total number of operations must be reduced below $10^8$. Therefore, we need to compress the time complexity per query from $O(N \times M)$ down to $O(\log N)$ or $O(1)$. The expected overall time complexity for a full‑score solution should be $O(N \times M + Q \log(\max(N, M)))$.
為了讓整體執行時間壓在 1000ms 內，總操作量必須降到 $10^8$ 以下，所以我們需要把每次查詢的時間複雜度從 $O(N \times M)$ 壓到 $O(\log N)$ 或 $O(1)$，滿分解的整體時間複雜度大概是 $O(N \times M + Q \log(\max(N, M)))$。

---

Analysis of the Example’s Key Misleading Point
範例中最容易誤導人的地方

The most deceptive part of this problem lies in the statement and the example diagram.
這題最騙人的地方在於題目敘述和那張範例圖。

The problem says “build a fence of the shortest possible total length to enclose all the sheep,” and the example picture draws an irregular concave polygon in yellow. This can easily mislead contestants into thinking they need to write complex geometric algorithms to dynamically construct a polygon.
題目說「建造一個總長度最短的籬笆把所有羊圍起來」，然後範例圖片畫了一個不規則的凹多邊形用黃色標示，很容易讓人誤以為要寫複雜的幾何演算法去動態構建多邊形。

However, on a grid where the fence can only run along cell edges:
不過，在一個柵欄只能沿著格線走的網格上：

· For any valid shape enclosing a set of points, its horizontal projection length must be at least $c_{max} - c_{min} + 1$, and its vertical projection length must be at least $r_{max} - r_{min} + 1$.
    任何一個合法圍住一群點的形狀，它的水平投影長度至少是 $c_{max} - c_{min} + 1$，垂直投影長度至少是 $r_{max} - r_{min} + 1$。
· Because edges can only travel in straight lines along the grid, the perimeter of an irregular polygon, in the best case, is exactly the same as the perimeter of its minimum bounding rectangle (bounding box).
    因為邊只能沿著格線走直線，不規則多邊形的周長，最好的情況下就等於它最小包圍矩形的周長。

Thus, the problem essentially only requires computing the outermost rectangular boundary that encloses all the sheep within the given query range. The formula is:
所以這題基本上只需要找出查詢範圍內所有羊的最外圍矩形邊界，公式就是：

\text{Perimeter} = 2 \times ((r_{max} - r_{min} + 1) + (c_{max} - c_{min} + 1))

The problem simplifies to: for each query rectangle, how can we quickly locate the top, bottom, left, and right extreme boundaries of the group of sheep inside it?
問題就簡化成：每次查詢一個矩形，要怎麼快速找出裡面那群羊的上下左右最遠邊界？

---

Step‑by‑Step Breakdown and Implementation of Each Subtask
各子任務逐步拆解與實現

In a contest, if you cannot immediately think of the full solution, using if-else to write separate code paths based on different data range characteristics is an important scoring strategy.
比賽中如果沒辦法馬上想到滿分解，用 if-else 根據不同數據範圍特性去寫不同段的 code 也是很重要的搶分策略。

---

Subtask 1 (7%): Exactly 1 cell is occupied by sheep

There is only one sheep on the whole map. Simply record its coordinates $(sr, sc)$ during input. For each query, just check whether this sheep falls inside the query rectangle. If it does, the minimum fence length surrounding a single sheep is $2 \times (1 + 1) = 4$; if not, it is $0$.
整張地圖只有一隻羊，輸入時直接記下牠的座標 $(sr, sc)$。每次查詢只要判斷這隻羊有沒有落在查詢矩形內，有的話一隻羊的最小籬笆長度就是 $2 \times (1 + 1) = 4$，沒有就是 $0$。

· Time complexity: $O(N \times M + Q)$
    時間複雜度：$O(N \times M + Q)$

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

---

Subtask 2 (8%): Exactly 2 cells are occupied by sheep

Record the coordinates of the two sheep. For each query, separately determine whether each sheep lies inside the query rectangle. Then find the extreme coordinates of all sheep inside the rectangle to calculate the perimeter.
記錄兩隻羊的座標。每次查詢分別判斷這兩隻羊是否在矩形內，再找出在矩形內的所有羊的極值座標，計算周長。

· Time complexity: $O(N \times M + Q)$
    時間複雜度：$O(N \times M + Q)$

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

---

Subtask 3 (15%): $N = 1$

The height is fixed to 1, so the 2D grid degenerates into a 1D array. We only care about the leftmost boundary $L$ and rightmost boundary $R$ in the horizontal direction.
高度固定為 1，二維網格退化成一維陣列，我們只需要關心水平方向的最左邊界 $L$ 和最右邊界 $R$。

· Guided thinking: When repeatedly querying the boundaries of elements within an interval, we can first use a 1D prefix sum to determine, in $O(1)$ time, whether the interval contains any sheep. Because the prefix sum array is monotonic (non‑decreasing), if there are sheep inside the interval, we can use Binary Search to quickly cut out the left and right boundaries.
    引導思考：當需要反覆查詢一個區間內有元素的邊界時，可以先做一維前綴和，在 $O(1)$ 時間內判斷該區間有沒有羊。因為前綴和陣列具有單調性（非遞減），如果有羊在區間內，就可以用二分搜尋快速切出左右邊界。
· Time complexity: $O(M + Q \log M)$
    時間複雜度：$O(M + Q \log M)$

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

---

Subtask 4 (15%): $Q = 1$

Since there is only one query, no complex optimization is needed. Directly traverse the rectangular region $[X_1, X_2] \times [Y_1, Y_2]$ specified by the query.
因為只有一次查詢，不需要做複雜的優化，直接遍歷該次查詢指定的矩形區域 $[X_1, X_2] \times [Y_1, Y_2]$ 即可。

· Time complexity: $O(N \times M)$
    時間複雜度：$O(N \times M)$

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

---

Subtask 5 (20%): $Q \le 3000$

When $Q$ reaches 3000, scanning with a double loop for every query will cause a TLE.
當 $Q$ 到 3000 時，每次查詢都用雙層迴圈掃會 TLE。

· Guided thinking: How can we quickly obtain the total number of sheep in any sub‑rectangle? We need to introduce 2D Prefix Sum. After obtaining the total sheep count in the current region, we can shrink the four boundaries of the query rectangle inward step by step (using while loops). As long as the total number of sheep inside the rectangle does not decrease after shrinking, it means the trimmed edge is empty and can be safely removed.
    引導思考：要怎麼快速得到任意子矩形的羊總數？我們需要引入二維前綴和。在取得當前區域的羊總數後，可以一步步（用 while 迴圈）將查詢矩形的四邊向內縮，只要縮了之後矩形的羊總數沒有變少，就代表那條邊是空的，可以放心拔掉。
· Time complexity: $O(N \times M + Q(N + M))$
    時間複雜度：$O(N \times M + Q(N + M))$

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

---

Subtask 6 (35%): No additional constraints ($Q \le 2 \times 10^5$)

· Guided thinking: Facing 200,000 queries, the linear while‑loop that shrinks the rectangle row by row will time out. When we fix three of the boundaries and move the fourth, the change in the number of contained sheep is monotonic. Since there is monotonicity, what method can we use to compress the complexity of finding the boundary from linear to logarithmic? The answer is Binary Search. Instead of stepping one cell at a time, we directly cut the range in half to verify whether the sheep count remains unchanged.
    引導思考：面對 200,000 筆查詢，用線性的 while 一步一步縮矩形會超時。我們固定三條邊、移動第四條邊時，包含的羊數量變化是單調的。既然有單調性，能用什麼方法把找邊界的複雜度從線性壓到對數級？答案就是二分搜尋。不再一格一格移動，而是直接對範圍切半去驗證羊的數量有沒有保持不變。
· Time complexity: $O(N \times M + Q \log(\max(N, M)))$
    時間複雜度：$O(N \times M + Q \log(\max(N, M)))$

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

---

Full solution:

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);
    return 0;
}
```

Recommended Questions：

…



—




D No 67 不可以 67

In this question, I will only explain how to get 51/67 marks.
這題我只會講解怎麼拿到 51/67 分。

· Solution Strategy: Split by subtask characteristics
    解題策略：根據子任務特性拆分處理
· Core Knowledge: Basic string operations (std::string::find and erase), 1D Prefix Sum
    核心知識：基本字串操作（std::string::find 和 erase）、一維前綴和

---

Data Range and Time Complexity Analysis
數據範圍與時間複雜度分析

The string length $|S|$ can be up to $10^5$, and the number of queries $Q$ up to $10^5$. Time limit is 1000ms.
字串長度 $|S|$ 最多可以到 $10^5$，查詢次數 $Q$ 也最多到 $10^5$，時間限制是 1000ms。

· Simulation method: Extract the substring and repeatedly find and delete "67". Each deletion requires shifting characters in the string; in the worst case the time complexity is related to the square of the query interval length. When $Q$ is small (e.g. $Q \le 10$), this method can pass.
    模擬法：擷取子字串，然後反覆尋找並刪除 "67"。每次刪除都要搬移字元，最壞情況下的時間複雜度和查詢區間長度的平方有關。當 $Q$ 很小（例如 $Q \le 10$）時，這個方法可以通過。
· Efficient solution: When $Q$ reaches $10^5$, each query must be answered in $O(1)$ or $O(\log |S|)$ time, otherwise it will exceed the time limit.
    高效解法：當 $Q$ 達到 $10^5$，每筆查詢必須在 $O(1)$ 或 $O(\log |S|)$ 內完成，否則會超時。

---

Subtask Analysis

Subtask 1 (12%): $|S| \le 20, Q \le 10$

· Characteristic: Very short string, very few queries.
    特性：字串很短，查詢很少。
· Solution: Directly extract the interval substring and simulate, or any brute‑force method will pass.
    解法：直接擷取區間子字串並模擬，或是任何暴力方法都會過。

---

Subtask 2 (18%): $Q \le 10$, and $S$ only contains digits 6 and 7
子任務 2 (18%)：$Q \le 10$，且 $S$ 只包含數字 6 和 7

· Characteristic: String consists only of 6 and 7, and there are at most 10 queries.
    特性：字串只由 6 和 7 構成，而且最多只有 10 筆查詢。
· Solution: Because the number of queries is small, we can use find to locate "67" and erase to delete it. The string shrinks automatically after deletion; repeat until no "67" is found.
    解法：因為查詢次數很少，可以直接用 find 找到 "67"，再用 erase 把它刪掉。刪完後字串會自己變短，重複直到找不到 "67" 為止。

---

Subtask 4 (21%): $Q \le 10$, no additional restrictions
子任務 4 (21%)：$Q \le 10$，無額外限制

· Characteristic: The string also contains other characters (1‑5, 8, 9, etc.), but the number of queries is still at most 10.
    特性：字串中還夾雜了其他字元（1‑5, 8, 9 之類的），但查詢次數一樣最多 10 筆。
· Solution: Although other characters are present, they do not form "67". We can still use the find + erase simulation from Subtask 2. Characters other than 6 and 7 stay in place and act as separators.
    解法：雖然有其他字元，但那些字元不會構成 "67"。我們還是可以沿用子任務 2 的 find + erase 模擬，6 和 7 以外的字元就留在原地當分隔符。

---

Subtask 3 (16%): $S$ only contains digits 6 and 7, and all 6's appear before any 7
子任務 3 (16%)：$S$ 只包含數字 6 和 7，且所有 6 都在 7 的前面

· Characteristic: The number of queries $Q$ can be as high as $10^5$, so simulation is not possible. However, the string format is fixed: 66...677...7.
    特性：查詢次數 $Q$ 可以高達 $10^5$，模擬一定不行。不過字串格式是固定的：66...677...7。
· Solution: Under this condition, any sub‑interval is also of the form 66...677...7. The most efficient way to eliminate all "67" is to delete all 6’s or delete all 7’s in the interval. Hence the answer is the minimum of the count of 6’s and the count of 7’s in the interval. This can be computed in $O(1)$ using 1D prefix sums.
    解法：在這種條件下，任意子區間也會是 66...677...7 的形式。要消掉所有 "67"，最直接的方法就是把區間內的 6 全部刪掉，或是把 7 全部刪掉，所以答案就是區間內 6 和 7 的數量的最小值。這個可以用一維前綴和在 $O(1)$ 內算出來。

---

Subtask 5 (33%): No additional restrictions
子任務 5 (33%)：無額外限制

· Characteristic: Large data range; requires a segment tree with DP matrix merging to answer each query in $O(\log |S|)$.
    特性：數據範圍大；需要用線段樹搭配 DP 矩陣合併，才能在 $O(\log |S|)$ 內回答每次查詢。

---

51 Points (Subtasks 1 + 2 + 4)
51 分（子任務 1 + 2 + 4）

These three subtasks share the characteristic $Q \le 10$. We can simulate using find and erase:
這三個子任務都有 $Q \le 10$ 的特性，我們可以直接用 find 和 erase 模擬：

1. For each query, obtain the substring with s.substr(l, r - l + 1).
      每次查詢時，用 s.substr(l, r - l + 1) 取出子字串。
2. Use a while loop with temp.find("67") to locate the target.
      用 while 迴圈搭配 temp.find("67") 找到目標。
3. If found, increment a counter and remove that segment with temp.erase(pos, 2).
      找到後就把計數器加一，並用 temp.erase(pos, 2) 把那一段刪掉。
4. Repeat until "67" is no longer found, then output the counter.
      重複直到找不到 "67"，然後輸出計數器的值。



Code (51 marks)

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);

    return 0;
}
```

---

67 Points (Adding Subtask 3)
67 分（加入子任務 3）

At the beginning of the program, check whether the whole string satisfies Subtask 3’s characteristic (only 6 and 7, and all 6’s appear before any 7):
程式一開始先判斷整條字串是否滿足子任務 3 的特性（只有 6 和 7，而且 6 全在 7 前面）：

· If it does, use prefix sum queries for all queries, outputting $\min(\text{count}_6, \text{count}_7)$ in the interval.
    如果滿足，就對所有查詢都用前綴和處理，輸出區間內 $\min(\text{count}_6, \text{count}_7)$。
· If it does not, fall back to the 51‑point find + erase simulation.
    如果不滿足，就退回 51 分的 find + erase 模擬做法。

---



Code (67 marks)

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
int main() {
    ios::sync_with_stdio(0); cin.tie(0);

    return 0;
}
```

Recommended Questions:




—



E Dungeon 地下城

Before start

We'll be using some graph and data structure concepts in this solution. While I'll explain the key ideas, it's a good idea to brush up on some basic graph knowledge before diving in.
這題解會用到一些圖論和資料結構的概念。雖然我會講解核心想法，但開始讀之前最好先補充一些基礎圖論知識。

Graph (I) 2026 : https://assets.hkoi.org/training2026/g-i.pdf
DFS (HKOI judge) : https://judge.hkoi.org/task/C800
Data Structures (III) 2026 (Sparse Table) : https://assets.hkoi.org/training2026/ds-iii.pdf#page=5
Data Structures (III) 2026 (Segment Tree) : https://assets.hkoi.org/training2026/ds-iii.pdf#page=35
Data Structures (IV) 2026 (Lazy Propagation on Segment Tree) : https://assets.hkoi.org/training2026/ds-iv.pdf#page=25
Data Structures (IV) 2026 (Binary Search on Segment Tree + Persistent Segment Tree) : https://assets.hkoi.org/training2026/ds-iv.pdf#page=80
Graph (V) 2026 : https://assets.hkoi.org/training2026/g-v.pdf#page=80
Sparse Table (HKOI judge) : https://judge.hkoi.org/task/B110
Segment Tree (Point Update, Range Query) (HKOI judge) : https://judge.hkoi.org/task/B111
Segment Tree (Range Maximum Query) (HKOI judge) : https://judge.hkoi.org/task/M0921
Lazy Propagation on Segment Tree (HKOI judge) : https://judge.hkoi.org/task/A101
Persistent Segment Tree (HKOI judge) : https://judge.hkoi.org/task/A103
Heavy-light Decomposition (HKOI judge) : https://judge.hkoi.org/task/A222

---

Core Concept: What are we solving for?

In the game, you travel from a starting point $S$ to a destination $T$. Every room on this path contains a monster with a level equal to the room number $i$. You must consume your level $i$ sword to defeat it.
遊戲中你從起點 $S$ 走到終點 $T$，這條路上的每個房間都有一隻等級等於房間編號 $i$ 的怪物，你必須消耗等級 $i$ 的劍才能打敗它。

This means: If room $i$ is on the path from $S$ to $T$, your sword of level $i$ will be consumed.
也就是說：如果房間 $i$ 在 $S$ 到 $T$ 的路徑上，你那把等級 $i$ 的劍就會被用掉。

Therefore, the "highest sword level" the player can keep is simply the largest room number that is NOT on the path from $S$ to $T$. If all rooms are on the path, we output 0.
所以，玩家能保留的「最高劍等級」就等於 不在 $S$ 到 $T$ 路徑上的最大房間編號。如果所有房間都在路徑上，就輸出 0。

---

Subtask 1: $N, M \leq 1000, Q = 1$

By looking at the constraints, the data size is small enough that an inefficient brute force solution can handle this subtask.
看一下限制，數據量夠小，一個不太有效率的暴力解就能搞定這個子任務。

So the question is how? How can we use a simple method to get the answer?
所以問題是怎麼做？怎麼用簡單的方法求出答案？

Think about keeping the largest sword only during the journey from $S$ to $T$. We can ignore one node (the highest possible sword we want to keep) and try to find a simple path from $S$ to $T$. Since we want the highest possible sword, we can try ignoring nodes from $N$ down to $1$, and simply use DFS from $S$ to $T$ to check if a simple path exists.
想像一下，我們從 $S$ 走到 $T$ 時想保留最大的劍，那麼我們可以忽略圖中的某個節點，再看看能不能找到一條從 $S$ 到 $T$ 的簡單路徑。因為我們要最大可能的劍，所以可以從 $N$ 一路往下試到 $1$，每次忽略掉那個節點，然後用 DFS 檢查 $S$ 到 $T$ 有沒有簡單路徑。

Code:

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2e5 + 100;
const int maxm = 2e5 + 100;

vector<int> e[maxn];
bool vis[maxn];

bool dfs(int s, int t, int esp){
    vis[s] = 1;
    if(s == t) return true;
    bool flag = false;
    for(int v : e[s]){
        if(v == esp) continue;
        if(!vis[v]) flag = flag or dfs(v, t, esp);
    }
    return flag;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);
    int n, m, q;
    cin >> n >> m >> q;
    set<pair<int, int>> s;
    for(int i = 0; i < m; i++){
        int u, v;
        cin >> u >> v;
        if(u == v) continue;
        if(u > v) swap(u, v);
        if(s.count({u, v}) != 0) continue;
        else {
            e[u].emplace_back(v);
            e[v].emplace_back(u);
            s.insert({u, v});
        }
    }
    if(n <= 1000 and m <= 1000 and q == 1){
        int s, t;
        cin >> s >> t;
        for(int i = n; i >= 1; i--){
            if(i == s or i == t) continue;
            memset(vis, 0, sizeof vis);
            if(dfs(s, t, i)){
                cout << i << "\n";
                return 0;
            }
        }
        cout << "0\n";
        return 0;
    }
    return 0;
}
```

Time Complexity: $O(QN^2)

---

Advanced subtasks

Subtask 2: $M = N-1$ (Graph is a Chain)

The graph is a chain. It looks just like a single straight line.
這張圖是一條鏈，看起來就像一條直線。

Here is an example of the graph in this subtask ($N = 4, M = 3$):
以下是這個子任務的一個例子 ($N = 4, M = 3$)：

```text
3 4
4 2
2 1
```

The physical graph structure connects the nodes like this:
圖的實際連接長這樣：

3 - 4 - 2 - 1

Because a chain only has one simple path between any two nodes, we can treat the entire graph like an array. However, the node IDs given in the input are not in a neat order. Therefore, we can use a single DFS traversal to flatten the graph into a neat 1D array.
因為鏈的任意兩點間只有一條簡單路徑，我們可以把整張圖當成一個陣列來處理。但輸入給的節點編號不一定是漂亮的順序，所以我們可以用一次 DFS 把圖攤平成一個整齊的一維陣列。

We will create two helper arrays:
我們會建立兩個輔助陣列：

· arr[i]: Tells us the actual room ID at the $i$-th position on the chain.
    arr[i]：記錄鏈上第 $i$ 個位置的實際房間編號。
· pos[u]: Tells us the 1D position (index) of room ID $u$.
    pos[u]：記錄房間 $u$ 在這個一維陣列中的位置（索引）。

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

After conversion, if we print arr from index 1 to 4, it looks like this: 3, 4, 2, 1.
轉換後，如果我們印出索引 1 到 4 的 arr，會得到：3, 4, 2, 1。

Once flattened, any path from a starting room $S$ to a target room $T$ simply covers a continuous segment on our 1D array. We have two main approaches to solve this subtask.
攤平之後，從起點 $S$ 到終點 $T$ 的路徑就只會對應到一維陣列上的一段連續區間。我們有兩種主要方法來解這個子任務。

---

Approach 1: Range Maximum Query (Sparse Table / Segment Tree)

方法一：區間最大值查詢（稀疏表 / 線段樹）

Assume we want to travel from $S$ to $T$. Let's look at their positions: pos[S] and pos[T].
假設我們要從 $S$ 走到 $T$，先看它們在一維陣列上的位置：pos[S] 和 pos[T]。

To make things easier, if pos[S] > pos[T], we can swap them so that $S$ always appears before $T$ on our chain (meaning pos[S] < pos[T]).
為了方便，如果 pos[S] > pos[T]，我們可以把兩個交換一下，讓 $S$ 在鏈上總是排在 $T$ 前面（也就是 pos[S] < pos[T]）。

The path from $S$ to $T$ consumes all swords in the range [pos[S], pos[T]]. This implies the surviving, unused swords belong to the remaining two parts of the chain:
從 $S$ 到 $T$ 的路徑會消耗掉區間 [pos[S], pos[T]] 內所有的劍。這表示還能保留、沒被用到的劍會落在鏈的另外兩段：

1. The left part: [1, pos[S] - 1]
      左段：[1, pos[S] - 1]
2. The right part: [pos[T] + 1, N]
      右段：[pos[T] + 1, N]

To find the maximum sword we can keep, we just need to find the maximum room ID across these two segments! We can do this efficiently using a Sparse Table or a Segment Tree. We must also carefully handle cases where these segments are empty (e.g., if $S$ is at the very beginning of the chain, the left part does not exist).
要找出我們能保留的最大劍，就只要在這兩段裡面找出最大的房間編號就行了！我們可以用稀疏表或是線段樹來高效率地查詢。當然也要小心處理這些區間可能為空的情況（例如 $S$ 剛好是鏈的起點，左段就不存在）。

Sparse Table Code:

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

Time Complexity (Sparse Table):

· Precompute: $O(N \log N)$
· Per query: $O(1)$
· Overall: $O(N \log N + Q)$

Segment Tree Code:
(The logic is the same, but we use a segment tree for the range maximum queries.)
(邏輯相同，只是用線段樹做區間最大值查詢。)

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

Time Complexity (Segment Tree):

· Precompute: $O(N)$
    預處理：$O(N)$
· Per query: $O(\log N)$
    每次查詢：$O(\log N)$
· Overall: $O(N + Q \log N)$
    整體：$O(N + Q \log N)$

---

Approach 2: Binary Search on Persistent Segment Tree

方法二：在持久化線段樹上二分搜尋

In this approach, instead of storing the maximum room IDs in the array positions, we use a Persistent Segment Tree (PST).
這個方法不是把最大房間編號存在陣列位置上，而是用一個持久化線段樹 (PST)。

Version $i$ of the tree will keep track of the frequencies of all sword levels (room IDs) present in the chain from position 1 up to position $i$.
版本 $i$ 的樹會記錄從鏈上位置 1 一路到位置 $i$ 之間，各個劍等級（房間編號）出現的次數。

When querying the path from pos[S] to pos[T], we want to find the maximum sword level that has an occurrence count of 0 inside this range. By traversing down the Persistent Segment Tree and checking the right child (larger sword levels) first, we can find this answer in $O(\log N)$ time.
當我們查詢 pos[S] 到 pos[T] 這段路徑時，我們想找出在該範圍內出現次數為 0 的最大劍等級。透過在持久化線段樹上往下走，並優先檢查右子樹（較大的劍等級），就能在 $O(\log N)$ 內找到答案。

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

Time Complexity (Persistent Segment Tree):

· Precompute: $O(N \log N)$
    預處理：$O(N \log N)$
· Per query: $O(\log N)$
    每次查詢：$O(\log N)$
· Overall: $O(N \log N + Q \log N)$
    整體：$O(N \log N + Q \log N)$

---

Subtask 3: $M = N-1$ (Graph is a Tree)

The graph is now a standard Tree. The simple path from $S$ to $T$ is still unique, but the graph is no longer a single straight line.
現在圖是一棵普通的樹了。從 $S$ 到 $T$ 的簡單路徑仍然是唯一的，但圖不再只是一條直線。

However, we can still use Approach 2 from Subtask 2!
不過，我們還是可以沿用子任務 2 的方法二！

We will use a technique called Heavy-Light Decomposition (HLD). HLD cleverly cuts any tree into a collection of straight chains. The beautiful property of HLD is that any path between two nodes $u$ and $v$ can be broken down into at most $O(\log N)$ continuous segments across these chains.
我們會用到一個叫重鏈剖分 (HLD) 的技巧。HLD 巧妙地將任意樹切成好幾條直鏈。HLD 的一個優美特性是，任意兩點 $u$ 和 $v$ 之間的路徑，都可以被拆成不超過 $O(\log N)$ 段在這些鏈上的連續區間。

Once we break the query path into $O(\log N)$ segments, we can query our Persistent Segment Tree just like before. Instead of checking a single interval [L, R], we sum up the occurrences of swords across all the $O(\log N)$ intervals simultaneously as we traverse down the PST.
當我們把查詢路徑拆成 $O(\log N)$ 段後，就可以像之前一樣查詢持久化線段樹。不同的是，我們不再只看一個 [L, R] 區間，而是在沿著 PST 往下走的過程中，同時加總所有 $O(\log N)$ 個區間內的劍出現次數。

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

Time Complexity (HLD + Persistent Segment Tree):

· Precompute: $O(N \log N)$
· Per query: $O(\log^2 N)$ (Because HLD provides $O(\log N)$ segments, and iterating through them while traversing the $O(\log N)$ tree depth gives $O(\log^2 N)$).
    每次查詢：$O(\log^2 N)$（因為 HLD 會給出 $O(\log N)$ 段區間，而在走 PST 的 $O(\log N)$ 層時會需要走訪所有這些區間，合起來就是 $O(\log^2 N)$）。
· Overall: $O(N \log N + Q \log^2 N)$
    整體：$O(N \log N + Q \log^2 N)$

---

Practice Questions:

· M2243 Yet Another RMQ

