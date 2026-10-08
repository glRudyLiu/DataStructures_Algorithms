# 核心資料結構速查手冊 (C++ 版)

本手冊涵蓋面試常用的基礎資料結構：**Vector**、**Stack**、**Queue / Deque**、**Hash Table**、**Set**、**有序容器 (map / set)** 與 **Heap**，整理了常用操作、時間複雜度、適用情境與常見陷阱。

> 本版修訂重點：新增 Vector、Deque、有序容器、嵌入式補充；補上最差複雜度、未定義行為（UB）警告、自訂 key 的雜湊寫法，以及題型對應速查。

---

## 〇、使用前先記住的通則

1. **空容器不要呼叫 `top()` / `front()` / `back()` / `pop()`**：在 `stack`、`queue`、`priority_queue`、`vector`、`deque` 上這樣做是**未定義行為（UB）**，不一定會當機，但結果不可預期。操作前先檢查 `empty()`。
2. **`size()` 回傳無號整數（`size_t`）**：`v.size() - 1` 在空容器時會變成極大的數。需要做減法或和負數比較時，先轉成 `int`：`(int)v.size() - 1`。
3. **「平均 $O(1)$」不等於最差 $O(1)$**：雜湊容器在碰撞嚴重時最差為 $O(N)$。面試時講「平均 $O(1)$，最差 $O(N)$」比講「瞬間查找」安全。
4. **標頭檔別漏**：`greater<int>` 需要 `#include <functional>`，`sort` / `reverse` / `remove` 需要 `#include <algorithm>`。很多環境會間接引入所以能編譯，但養成習慣最保險。
5. **傳參數用 `const &`**：容器按值傳遞會整份複製，刷題時容易因此超時。
6. **整數溢位**：累加和、乘積先考慮是否需要 `long long`。

---

## 一、 Vector（動態陣列）

- **運作核心**：記憶體**連續**的動態陣列，支援隨機存取，尾端新增 / 刪除均攤 $O(1)$。
- **語言對應**：C++ `std::vector<T>`
- **引入標頭檔**：`#include <vector>`

### 1. 常用功能與複雜度
- `v.push_back(x)` / `v.emplace_back(...)`：尾端新增 — 均攤 $O(1)$
- `v.pop_back()`：移除尾端（**不回傳值**）— $O(1)$
- `v[i]`：隨機存取（**不檢查邊界，越界是 UB**）— $O(1)$
- `v.at(i)`：隨機存取（越界會丟 `out_of_range` 例外）— $O(1)$
- `v.front()` / `v.back()`：首 / 尾元素 — $O(1)$
- `v.size()` / `v.empty()` / `v.capacity()`：狀態查詢 — $O(1)$
- `v.insert(it, x)` / `v.erase(it)`：中間插入 / 刪除（後面元素要搬移）— $O(N)$
- `v.clear()`：清空（`size` 變 0，`capacity` 通常不變）— $O(N)$
- `v.resize(n)` / `v.reserve(n)`：調整大小 / 預先配置容量 — $O(N)$
- `v.assign(n, val)`：重設為 n 個 val — $O(N)$
- 搭配 `<algorithm>`：`sort`（$O(N \log N)$）、`reverse`、`find`、`lower_bound`（需先排序）、`accumulate`（`<numeric>`）

### 2. 經典使用情境
- **一般陣列的替代品**：絕大多數題目的輸入輸出、暫存結果。
- **DP 表**：`vector<int> dp(n + 1, 0)`、二維 `vector<vector<int>> dp(m, vector<int>(n, 0))`。
- **鄰接表（圖）**：`vector<vector<int>> adj(n)`，`adj[u].push_back(v)`。
- **回溯的 path / 結果集合**：`path.push_back(x)` 與 `path.pop_back()`（回溯模板的「選擇與撤銷」）。
- **前綴和**：`vector<long long> prefix(n + 1)`。
- **當作 Stack 用**：`push_back` / `back` / `pop_back` 就是堆疊，且可以遍歷。

### 3. 常見陷阱
- **擴容會使迭代器 / 參考 / 指標失效**：`push_back` 觸發重新配置時，之前取得的 `it`、`&v[0]` 都不能再用。已知大小時先 `reserve`。
- **擴容策略**：容量通常以固定倍率成長（常見為 2 倍或 1.5 倍，視實作而定），這是均攤 $O(1)$ 的來源。
- **`size()` 無號問題**：`for (int i = 0; i < v.size() - 1; i++)` 在空 vector 會變成超長迴圈。
- **`vector<bool>` 是特化版本**：用位元壓縮儲存，`v[i]` 回傳的是代理物件，不能取參考，行為與一般容器不同。需要時改用 `vector<char>` 或 `bitset`。
- **刪除指定值**：`erase` 配合 `remove`（erase-remove idiom），C++20 可以直接用 `std::erase(v, x)`。
- **迴圈中邊遍歷邊 `erase` / `insert`** 會讓迭代器失效，要用 `erase` 的回傳值接續，或改用索引倒著刪。
- **二維陣列宣告**：`vector<vector<int>> g(m, vector<int>(n))`；不要寫成 `vector<int> g[m]` 當動態大小用（m 不是編譯期常數時不合法）。

### 4. C++ 範例代碼
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
    vector<int> v;
    v.reserve(10);            // 預留容量，避免多次重新配置

    // 新增
    v.push_back(30);
    v.push_back(10);
    v.push_back(20);

    // 存取
    cout << "首: " << v.front() << ", 尾: " << v.back() << endl; // 30, 20
    cout << "v[1] = " << v[1] << endl;                          // 10

    // 中間插入與刪除 (O(N))
    v.insert(v.begin() + 1, 99);   // 30 99 10 20
    v.erase(v.begin());            // 99 10 20

    // 排序與反轉
    sort(v.begin(), v.end());                 // 10 20 99
    sort(v.begin(), v.end(), greater<int>()); // 99 20 10 (遞減)
    reverse(v.begin(), v.end());              // 10 20 99

    // 刪除指定值 (erase-remove idiom)
    v.erase(remove(v.begin(), v.end(), 20), v.end()); // 10 99

    // 遍歷
    for (int x : v) cout << x << " ";
    cout << endl;

    // 安全的索引遍歷（避免 size_t 減法陷阱）
    for (int i = 0; i < (int)v.size() - 1; i++) {
        cout << v[i] + v[i + 1] << " ";
    }
    cout << endl;

    // 二維陣列
    int m = 3, n = 4;
    vector<vector<int>> grid(m, vector<int>(n, 0));
    grid[1][2] = 5;

    return 0;
}
```

---

## 二、 Stack（堆疊）

- **運作核心**：後進先出（LIFO, Last-In-First-Out）—— 最後進來的，最先被拿出來。
- **語言對應**：C++ `std::stack<T>`
- **引入標頭檔**：`#include <stack>`
- **補充**：`stack` 是**容器轉接器（container adapter）**，底層預設用 `deque`，只開放堆疊操作，**不能遍歷、不能隨機存取**。需要遍歷時可直接用 `vector` 當堆疊。

### 1. 常用功能與複雜度
- `st.push(x)`：把元素壓入頂端 — $O(1)$
- `st.pop()`：移除頂端元素（**注意：C++ 的 pop() 不會回傳值**）— $O(1)$
- `st.top()`：偷看頂端元素（不移除）— $O(1)$
- `st.empty()`：檢查堆疊是否為空 — $O(1)$
- `st.size()`：取得目前堆疊內的元素數量 — $O(1)$
- **務必先 `empty()` 檢查再 `top()` / `pop()`**，否則是 UB。

### 2. 經典使用情境
- **歷史紀錄與復原**：瀏覽器上一頁、編輯器 Ctrl+Z、遞迴系統調用（call stack）。
- **符號匹配與成對消除**：括號配對（LeetCode #20）、相鄰重複字元消除（LeetCode #1047）。
- **運算式求值**：逆波蘭後序表達式計算（LeetCode #150）。
- **單調堆疊（Monotonic Stack）**：找下一個更大 / 更小元素，每個元素最多進出一次，均攤 $O(N)$（LeetCode #739、#496）。
- **用堆疊把遞迴改成迭代**：DFS 的非遞迴寫法、避免遞迴過深。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <stack>

using namespace std;

int main() {
    stack<int> st;

    st.push(10);
    st.push(20);
    st.push(30); // 30 在最上方

    cout << "當前頂端: " << st.top() << endl; // 30

    st.pop();
    cout << "pop 一次後的頂端: " << st.top() << endl; // 20

    cout << "目前元素量: " << st.size() << endl; // 2

    // 取值與移除要分兩步
    while (!st.empty()) {
        int x = st.top();
        st.pop();
        cout << x << " ";
    }
    cout << endl;

    return 0;
}
```

### 4. 單調堆疊範例：Daily Temperatures（#739）
```cpp
// 堆疊存「索引」，保持對應溫度由底到頂遞減；遇到更高溫就結算被壓住的日子
vector<int> dailyTemperatures(vector<int>& t) {
    int n = t.size();
    vector<int> ans(n, 0);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && t[st.top()] < t[i]) {
            int j = st.top();
            st.pop();
            ans[j] = i - j;
        }
        st.push(i);
    }
    return ans;
}
```

---

## 三、 Queue（佇列 / 隊列）

- **運作核心**：先進先出（FIFO, First-In-First-Out）—— 先排隊的，先處理離開。
- **語言對應**：C++ `std::queue<T>`
- **引入標頭檔**：`#include <queue>`
- **補充**：`queue` 同樣是容器轉接器（底層預設 `deque`），**不能遍歷、不能隨機存取**。

### 1. 常用功能與複雜度
- `q.push(x)`：新元素從隊尾排進來 — $O(1)$
- `q.pop()`：隊頭出隊（**不回傳值**）— $O(1)$
- `q.front()`：讀取隊頭元素 — $O(1)$
- `q.back()`：讀取隊尾（最後進入）的元素 — $O(1)$
- `q.empty()`, `q.size()`：狀態檢查 — $O(1)$
- 同樣要先 `empty()` 檢查再 `front()` / `pop()`。

### 2. 經典使用情境
- **任務排程與緩衝區（Buffer）**：印表機任務、訊息佇列（Message Queue）、連線等待隊列。
- **廣度優先搜尋（BFS）**：二元樹層序遍歷（Level-order）、無權重圖尋找最短步數（多源 BFS 也一樣）。
- **資料流處理**：先進先出的事件處理。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <queue>

using namespace std;

int main() {
    queue<int> q;

    q.push(100);
    q.push(200);
    q.push(300);

    cout << "隊頭 (第一順位): " << q.front() << endl; // 100
    cout << "隊尾 (剛排進來): " << q.back() << endl;  // 300

    q.pop();
    cout << "pop 一次後的新隊頭: " << q.front() << endl; // 200

    cout << "隊伍長度: " << q.size() << endl; // 2
    while (!q.empty()) {
        q.pop();
    }

    return 0;
}
```

### 4. BFS 層序遍歷模板
```cpp
// 一次處理一整層：先記下當層大小 sz，再處理 sz 個節點
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level;
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front();
            q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

---

## 四、 Deque（雙端佇列）

- **運作核心**：**兩端**都能 $O(1)$ 新增 / 刪除，同時支援**隨機存取** `[]`。
- **語言對應**：C++ `std::deque<T>`
- **引入標頭檔**：`#include <deque>`
- **底層**：分段的連續記憶體區塊，**整體不連續**（與 `vector` 不同），所以不能把 `&dq[0]` 當連續陣列使用。

### 1. 常用功能與複雜度
- `dq.push_back(x)` / `dq.push_front(x)`：兩端新增 — $O(1)$
- `dq.pop_back()` / `dq.pop_front()`：兩端移除（不回傳值）— $O(1)$
- `dq.front()` / `dq.back()`：讀取兩端 — $O(1)$
- `dq[i]` / `dq.at(i)`：隨機存取 — $O(1)$
- `dq.insert(it, x)` / `dq.erase(it)`：中間操作 — $O(N)$
- `dq.empty()`, `dq.size()`, `dq.clear()`

### 2. 經典使用情境
- **單調佇列（Monotonic Queue）**：滑動視窗最大 / 最小值（LeetCode #239）。
- **需要兩端操作的場景**：0-1 BFS、回文判斷（從兩端各取一個）、LRU 類結構的輔助。
- **作為 `stack` / `queue` 的底層**：知道這點，就能解釋為什麼它們不能遍歷。

### 3. 單調佇列範例：Sliding Window Maximum（#239）
```cpp
// 佇列存「索引」，保持對應數值由頭到尾遞減；頭永遠是當前視窗最大值
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    int n = nums.size();
    deque<int> dq;
    vector<int> res;
    for (int i = 0; i < n; i++) {
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();   // 移除滑出視窗的索引
        while (!dq.empty() && nums[dq.back()] < nums[i]) dq.pop_back(); // 移除不可能成為最大值的元素
        dq.push_back(i);
        if (i >= k - 1) res.push_back(nums[dq.front()]);
    }
    return res;
}
```

---

## 五、 Hash Table（雜湊表 / 哈希表）

- **運作核心**：鍵值對（Key-Value）映射，透過雜湊函式（Hash Function）把 key 算成桶（bucket）的位置，再到該桶中查找。
- **語言對應**：C++ `std::unordered_map<Key, Value>`
- **引入標頭檔**：`#include <unordered_map>`

### 1. 常用功能與複雜度
- `mp[key] = val`：存入或更新數值 — 平均 $O(1)$，最差 $O(N)$
- `mp[key]`：讀取數值（**注意：若 key 不存在會自動建立並填入預設值 0 或空值**）— 平均 $O(1)$
- `mp.at(key)`：讀取數值，key 不存在會丟 `out_of_range`，**不會**自動建立
- `mp.count(key)`：檢查 key 是否存在（回傳 1 或 0）— 平均 $O(1)$；C++20 可用 `mp.contains(key)`
- `mp.find(key)`：查找迭代器，若 `mp.find(key) == mp.end()` 表示找不到 — 平均 $O(1)$
- `mp.emplace(k, v)` / `mp.try_emplace(k, v)`：key 不存在才插入，不覆蓋既有值
- `mp.erase(key)`：刪除指定的 key 與其數值 — 平均 $O(1)$
- `mp.size()`, `mp.empty()`, `mp.clear()`, `mp.reserve(n)`（預留桶數，減少 rehash）

### 2. 雜湊表的原理（面試常追問）
- **雜湊函式**：把 key 轉成整數，再對桶數取餘決定位置。
- **碰撞（Collision）**：不同 key 落在同一個桶。常見處理方式：**鏈結法（chaining）**——每個桶掛一串節點（標準庫實作通常屬於此類）；以及**開放定址法（open addressing）**——往後探測空位。
- **負載因子（load factor）**：元素數 ÷ 桶數。超過門檻會 **rehash**（擴張桶數並重新分配所有元素），所以單次插入可能是 $O(N)$，但均攤仍是 $O(1)$。
- **最差情況**：全部碰撞到同一個桶時退化成 $O(N)$。
- **遍歷順序不保證**：`unordered_*` 的走訪順序與插入順序無關，也可能在 rehash 後改變。需要有序時用 `map`。

### 3. 經典使用情境
- **平均 $O(1)$ 查找（以空間換時間）**：把 $O(N)$ 線性搜尋壓到平均 $O(1)$（LeetCode #1 Two Sum）。
- **計數器 / 頻率統計**：`mp[c]++`，統計字母、單字或數字出現次數（LeetCode #242 Valid Anagram、#49 Group Anagrams）。
- **前綴和加雜湊表**：和為 K 的子陣列（LeetCode #560）。
- **關係對應與快取（Cache）**：儲存對應關係、記憶化搜尋（Memoization）。

### 4. 常見陷阱
- 想「只檢查存在與否」時，**不要用 `mp[key]`**，它會偷偷新增一筆。用 `count` / `find` / `contains`。
- **遍歷時 `erase`**：要用 `it = mp.erase(it)` 接續，直接 `erase(it)` 後再 `++it` 是錯的。
- **自訂型別當 key**：`pair`、`vector`、自訂 struct **沒有預設雜湊函式**，`unordered_map<pair<int,int>, int>` 無法編譯（見下方）。
- 用 `unordered_map` 時 key 若是 `char` 或小範圍整數，可以改用 `vector<int>(26)` 之類的陣列，更快更省。

### 5. 自訂 key（以座標 `pair<int,int>` 為例）
```cpp
// 作法一（推薦，最簡單）：把座標編碼成單一整數
int key = r * cols + c;
unordered_set<int> visited;
visited.insert(key);

// 作法二：自訂雜湊函式
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        return hash<long long>()(((long long)p.first << 32) ^ (unsigned int)p.second);
    }
};
unordered_set<pair<int, int>, PairHash> visited2;
unordered_map<pair<int, int>, int, PairHash> dist;
```
> 若不想寫雜湊，也可以改用有序的 `set<pair<int,int>>` / `map<pair<int,int>, int>`（`pair` 預設支援比較），代價是操作變成 $O(\log N)$。

### 6. C++ 範例代碼
```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>

using namespace std;

int main() {
    unordered_map<string, int> priceTable;

    // 寫入與更新資料
    priceTable["apple"] = 50;
    priceTable["banana"] = 25;
    priceTable["cherry"] = 100;

    // 讀取資料
    cout << "蘋果價格: " << priceTable["apple"] << endl; // 50

    // 檢查 Key 是否存在 (避免用 [] 誤增鍵值)
    if (priceTable.count("banana")) {
        cout << "香蕉在庫！" << endl;
    }
    if (priceTable.find("orange") == priceTable.end()) {
        cout << "柳橙不存在！" << endl;
    }

    // 刪除鍵值
    priceTable.erase("apple");

    // 遍歷所有 Key-Value（順序不保證）
    for (const auto& pair : priceTable) {
        cout << pair.first << " -> " << pair.second << endl;
    }

    return 0;
}
```

### 7. 經典題範例：Two Sum（#1）
```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> idx;           // 值 -> 索引
    for (int i = 0; i < (int)nums.size(); i++) {
        auto it = idx.find(target - nums[i]);
        if (it != idx.end()) return {it->second, i};
        idx[nums[i]] = i;                  // 先查再放，避免用到自己
    }
    return {};
}
```

---

## 六、 Set（集合）

- **運作核心**：只存 Key（沒有 Value），**保證元素唯一、絕不重複**。
- **語言對應**：C++ `std::unordered_set<Key>`
- **引入標頭檔**：`#include <unordered_set>`

### 1. 常用功能與複雜度
- `st.insert(x)`：加入元素，已存在則略過。回傳 `pair<iterator, bool>`，`.second` 為 `true` 代表真的新增了 — 平均 $O(1)$，最差 $O(N)$
- `st.count(x)`：檢查元素是否在集合內（1 在，0 不在）— 平均 $O(1)$；C++20 可用 `st.contains(x)`
- `st.find(x)`：回傳迭代器 — 平均 $O(1)$
- `st.erase(x)`：將指定元素從集合中移除 — 平均 $O(1)$
- `st.size()`, `st.empty()`, `st.clear()`
- **遍歷順序不保證**；需要有序時用 `set`。

### 2. 經典使用情境
- **資料去重**：過濾輸入資料中重複的值（LeetCode #217 Contains Duplicate）。
- **存在性驗證**：快速確認某個數字或字串是否出現過；最長連續序列（LeetCode #128）。
- **防止無限迴圈（Visited 標記）**：BFS / DFS 時記錄走過的節點或座標。座標當 key 的寫法見「五、Hash Table」的自訂 key。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <unordered_set>
#include <vector>

using namespace std;

int main() {
    unordered_set<int> visited;

    visited.insert(10);
    visited.insert(20);
    auto res = visited.insert(10);   // 重複，被自動略過
    cout << "是否真的新增: " << res.second << endl; // 0

    cout << "不重複數量: " << visited.size() << endl; // 2

    if (visited.count(20)) {
        cout << "20 在集合中" << endl;
    }

    visited.erase(10);
    if (!visited.count(10)) {
        cout << "10 已被成功移除" << endl;
    }

    // 用 vector 快速建立去重集合
    vector<int> nums = {1, 2, 2, 3, 3, 3};
    unordered_set<int> uniq(nums.begin(), nums.end());
    cout << "去重後數量: " << uniq.size() << endl; // 3

    return 0;
}
```

---

## 七、 有序容器：`map` / `set` / `multiset`

- **運作核心**：元素**依 key 排序**，底層通常是紅黑樹（自平衡二元搜尋樹）。
- **語言對應**：C++ `std::map` / `std::set` / `std::multiset`
- **引入標頭檔**：`#include <map>`、`#include <set>`

### 1. 與 `unordered_*` 的取捨

| 比較項目 | `unordered_map` / `unordered_set` | `map` / `set` |
| :--- | :--- | :--- |
| 底層 | 雜湊表 | 紅黑樹 |
| 新增 / 查找 / 刪除 | 平均 $O(1)$，最差 $O(N)$ | **穩定 $O(\log N)$** |
| 元素順序 | 無序 | **依 key 排序** |
| 範圍查詢、前驅 / 後繼 | 不支援 | 支援 |
| key 的要求 | 需可雜湊、可比較相等 | 需可比較大小（`pair` 預設可用） |

> 只需要「存在與否 / 計數」用 `unordered_*`；需要排序、最小最大值、範圍查詢時用 `map` / `set`。

### 2. 常用功能
- `begin()`：最小元素；`rbegin()`：最大元素（`m.begin()->first`、`m.rbegin()->first`）。
- `lower_bound(k)`：第一個 **≥ k** 的元素；`upper_bound(k)`：第一個 **> k** 的元素 — $O(\log N)$。
- `multiset`：允許重複元素；刪除單一個要用 `ms.erase(ms.find(x))`，否則 `ms.erase(x)` 會把所有相同值一起刪掉。

### 3. 經典使用情境
- 需要動態維護「有序集合」並隨時取最小 / 最大（例如滑動視窗用 `multiset`）。
- 查找「剛好比某值大 / 小的元素」（前驅、後繼）。
- 行事曆、區間類題目（如 #729 My Calendar I）。

### 4. C++ 範例代碼
```cpp
#include <iostream>
#include <map>
#include <set>

using namespace std;

int main() {
    map<int, string> m = {{10, "a"}, {20, "b"}, {30, "c"}};

    cout << "最小 key: " << m.begin()->first << endl;   // 10
    cout << "最大 key: " << m.rbegin()->first << endl;  // 30

    auto it = m.lower_bound(15);                        // 第一個 >= 15
    if (it != m.end()) cout << it->first << endl;       // 20

    // 找「小於等於 k 的最大 key」(floor)
    int k = 25;
    auto it2 = m.upper_bound(k);                        // 第一個 > 25 -> 30
    if (it2 != m.begin()) {
        --it2;
        cout << "floor(25) = " << it2->first << endl;   // 20
    }

    multiset<int> ms = {5, 3, 5, 1};
    ms.erase(ms.find(5));                                // 只刪一個 5
    cout << "ms 最小值: " << *ms.begin() << endl;        // 1

    return 0;
}
```

---

## 八、 Heap / Priority Queue（堆積 / 優先佇列）

- **運作核心**：內部為完全二元樹（通常以陣列實作），動態維持偏序關係，**頂端永遠是當前的極值**。
- **語言對應**：C++ `std::priority_queue<T>`
- **引入標頭檔**：`#include <queue>`（`greater<>` 需 `#include <functional>`）

### 1. 常用功能與複雜度
- **宣告語法**：
  - *最大堆（預設）*：`priority_queue<int> max_heap;`（頂端永遠是最大值）
  - *最小堆*：`priority_queue<int, vector<int>, greater<int>> min_heap;`（頂端永遠是最小值）
- `pq.push(x)`：放入新元素 — $O(\log N)$
- `pq.top()`：查看頂端極值 — $O(1)$
- `pq.pop()`：移除頂端極值（不回傳值）— $O(\log N)$
- `pq.empty()`, `pq.size()` — $O(1)$
- **限制**：不能遍歷、不能修改或刪除任意元素（沒有 decrease-key）。
- **建堆**：用迭代器範圍建構 `priority_queue<int> pq(v.begin(), v.end());` 為 $O(N)$；逐一 `push` 則是 $O(N \log N)$。

### 2. 比較方向要特別記
`priority_queue` 的比較函式語意與 `sort` **相反**：`sort` 的 comp 回傳 `true` 表示 a 排前面；`priority_queue` 的 comp 回傳 `true` 表示 a 的優先權**較低**（排在 b 後面）。所以「想讓小的先出」要用 `>`（`greater`）。

### 3. 經典使用情境
- **Top K 問題**：前 K 大 / 小（LeetCode #215、#347、#703）。
- **串流極值即時維護**：資料不斷進來時隨時取極值。
- **雙堆求中位數**：一個最大堆加一個最小堆（LeetCode #295）。
- **貪婪與最短路徑**：合併 K 個已排序串列（LeetCode #23）、Dijkstra、任務排程。

### 4. C++ 範例代碼
```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <functional>

using namespace std;

int main() {
    // 1. 最大堆 (Max Heap)
    priority_queue<int> max_heap;
    max_heap.push(30);
    max_heap.push(10);
    max_heap.push(50);

    cout << "Max Heap 頂端 (最大): " << max_heap.top() << endl; // 50
    max_heap.pop();
    cout << "pop 後的新頂端: " << max_heap.top() << endl;      // 30

    // 2. 最小堆 (Min Heap)
    priority_queue<int, vector<int>, greater<int>> min_heap;
    min_heap.push(30);
    min_heap.push(10);
    min_heap.push(50);

    cout << "Min Heap 頂端 (最小): " << min_heap.top() << endl; // 10
    min_heap.pop();
    cout << "pop 後的新頂端: " << min_heap.top() << endl;      // 30

    return 0;
}
```

### 5. 自訂比較（Dijkstra 常用）
```cpp
// 元素為 {節點, 距離}，希望「距離小的先出」
auto cmp = [](const pair<int, int>& a, const pair<int, int>& b) {
    return a.second > b.second;   // second 小的在頂端
};
priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(cmp)> pq(cmp);

// 另一種常見寫法：把距離放 pair 的第一個，搭配 greater
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq2;
pq2.push({dist, node});   // 以 first 為主比較
```
> Dijkstra 沒有 decrease-key，常見做法是**重複 push、取出時若發現過期（距離比已記錄的大）就跳過**（lazy deletion）。

### 6. Top K 標準寫法（找第 K 大）
```cpp
// 維持一個「大小為 K 的最小堆」：堆頂就是目前第 K 大；時間 O(N log K)
priority_queue<int, vector<int>, greater<int>> pq;
for (int x : nums) {
    pq.push(x);
    if ((int)pq.size() > k) pq.pop();
}
int kthLargest = pq.top();
```

---

## 九、 橫向對比總整理

| 資料結構 | C++ 容器 | 新增 | 刪除 | 查找 / 存取 | 最差情況 | 有序 | 可遍歷 | 核心記憶特色 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Vector** | `vector<T>` | 尾端均攤 $O(1)$，中間 $O(N)$ | 尾端 $O(1)$，中間 $O(N)$ | 索引 $O(1)$，找值 $O(N)$ | 擴容時單次 $O(N)$ | 依插入順序 | 是 | 連續記憶體，最常用 |
| **Stack** | `stack<T>` | $O(1)$ | $O(1)$ | 只能看 `top` | $O(1)$ | — | 否 | 後進先出，歷史回溯 |
| **Queue** | `queue<T>` | $O(1)$ | $O(1)$ | 只能看 `front` / `back` | $O(1)$ | — | 否 | 先進先出，BFS |
| **Deque** | `deque<T>` | 兩端 $O(1)$ | 兩端 $O(1)$ | 索引 $O(1)$ | 中間操作 $O(N)$ | 依插入順序 | 是 | 雙端皆快，單調佇列 |
| **Hash Table** | `unordered_map<K,V>` | 平均 $O(1)$ | 平均 $O(1)$ | 平均 $O(1)$ | $O(N)$ | 否 | 是（順序不定） | Key-Value 映射，空間換時間 |
| **Set** | `unordered_set<K>` | 平均 $O(1)$ | 平均 $O(1)$ | 平均 $O(1)$ | $O(N)$ | 否 | 是（順序不定） | 純 Key，自動去重 |
| **有序容器** | `map` / `set` | $O(\log N)$ | $O(\log N)$ | $O(\log N)$ | $O(\log N)$ | **是** | 是（有序） | 紅黑樹，可範圍查詢 |
| **Heap** | `priority_queue<T>` | $O(\log N)$ | 只能刪頂端 $O(\log N)$ | 只能看 `top` $O(1)$ | $O(\log N)$ | 只保證頂端 | 否 | 動態極值，Top K |

---

## 十、 題型對應速查（看到什麼，想到什麼）

| 題目特徵 | 優先想到的結構 / 技巧 |
| :--- | :--- |
| 需要快速判斷「出現過沒」「找補數」 | `unordered_set` / `unordered_map` |
| 統計頻率、分組 | `unordered_map`（計數器） |
| 括號配對、成對消除、倒序處理 | `stack` |
| 下一個更大 / 更小元素 | 單調 `stack` |
| 層序遍歷、最短步數（無權重） | `queue` + BFS |
| 滑動視窗的最大 / 最小值 | 單調 `deque` |
| 前 K 大 / 小、持續取極值 | `priority_queue` |
| 需要排序狀態並查前驅 / 後繼 | `map` / `set` / `multiset` |
| 子陣列和、區間和 | 前綴和（搭配 `unordered_map`） |
| 圖的建立 | `vector<vector<int>>` 鄰接表 |
| DP 表、回溯的 path | `vector` |

---

## 十一、 嵌入式補充（韌體面試）

### 1. STL 在嵌入式環境的考量
- 多數 STL 容器（`vector`、`deque`、`map`、`unordered_*`）會**動態配置記憶體**，在記憶體受限、需要確定性（deterministic）行為或禁用 `new` / `malloc` 的系統上常被避免。
- 常見替代：**固定容量**的陣列加索引，自行實作 ring buffer、memory pool；編譯期固定大小可用 `std::array`。
- 面試可以主動說明：知道 STL 的便利，也知道在受限環境下為何改用固定容量結構。

### 2. 環形緩衝區（Ring Buffer）：嵌入式最常考的佇列實作
```cpp
#include <cstddef>

// 固定容量、不動態配置的環形佇列（單執行緒版本）
template <typename T, size_t N>
class RingBuffer {
    T buf_[N];
    size_t head_ = 0;   // 下一個要讀取的位置
    size_t tail_ = 0;   // 下一個要寫入的位置
    size_t count_ = 0;

public:
    bool push(const T& v) {
        if (count_ == N) return false;   // 已滿：回報失敗（也可選擇覆寫最舊資料）
        buf_[tail_] = v;
        tail_ = (tail_ + 1) % N;
        ++count_;
        return true;
    }
    bool pop(T& out) {
        if (count_ == 0) return false;   // 已空
        out = buf_[head_];
        head_ = (head_ + 1) % N;
        --count_;
        return true;
    }
    bool empty() const { return count_ == 0; }
    bool full()  const { return count_ == N; }
    size_t size() const { return count_; }
};
```
**面試追問重點**：
- 如何區分「滿」與「空」（用 `count`，或空出一格，或用單調遞增的索引）。
- 容量取 2 的冪時，可以用位元遮罩 `& (N - 1)` 取代 `%`。
- **在 ISR 與主程式之間共用時**，上面這個版本並不安全。單一生產者、單一消費者（SPSC）的情境，可以讓 `head` 只由消費者修改、`tail` 只由生產者修改，並搭配 `volatile` 或 `std::atomic` 與適當的記憶體順序，避免競爭條件。
- 滿了怎麼辦：丟棄新資料、覆寫最舊資料、或回報錯誤，取決於需求。
