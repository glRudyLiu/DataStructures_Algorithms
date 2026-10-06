# 核心資料結構速查手冊 (C++ & C# 對照版)

本手冊涵蓋五大基礎資料結構：**Stack**、**Queue**、**Hash Table**、**Set** 與 **Heap**，整理了常用的操作函式、時間複雜度、適用情境與語言對照。

---

## 一、 Stack（堆疊）

- **運作核心**：後進先出（LIFO, Last-In-First-Out）—— 最後進來的，最先被拿出來。
- **語言對應**：C++ `std::stack<T>` ｜ C# `Stack<T>`
- **引入標頭檔**：`#include <stack>`

### 1. 常用功能與複雜度
- `st.push(x)`：把元素壓入頂端 — $O(1)$
- `st.pop()`：移除頂端元素（**注意：C++ 的 pop() 不會回傳值**）— $O(1)$
- `st.top()`：偷看頂端元素（不移除）— $O(1)$
- `st.empty()`：檢查堆疊是否為空 — $O(1)$
- `st.size()`：取得目前堆疊內的元素數量 — $O(1)$

### 2. 經典使用情境
- **歷史紀錄與復原**：瀏覽器上一頁、編輯器 Ctrl+Z、遞迴系統調用。
- **符號匹配與成對消除**：括號配對（LeetCode #20）、相鄰重複字元消除（LeetCode #1047）。
- **運算式求值**：逆波蘭後序表達式計算（LeetCode #150）。
- **單調堆疊（Monotonic Stack）**：找下一個更大/更小元素，消除無效重複比對，均攤複雜度 $O(N)$（LeetCode #739、#496）。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <stack>

using namespace std;

int main() {
    // 宣告
    stack<int> st;

    // 依序寫入 (push)
    st.push(10);
    st.push(20);
    st.push(30); // 30 在最上方

    // 讀取頂端 (top)
    cout << "當前頂端: " << st.top() << endl; // 輸出: 30

    // 移除頂端 (pop)
    st.pop();
    cout << "pop 一次後的頂端: " << st.top() << endl; // 輸出: 20

    // 檢查大小與狀態
    cout << "目前元素量: " << st.size() << endl; // 輸出: 2
    if (!st.empty()) {
        cout << "Stack 目前不是空的" << endl;
    }

    return 0;
}
```
---

## 二、 Queue（佇列 / 隊列）

- **運作核心**：先進先出（FIFO, First-In-First-Out）—— 先排隊的，先處理離開。
- **語言對應**：C++ `std::queue<T>` ｜ C# `Queue<T>`
- **引入標頭檔**：`#include <queue>`

### 1. 常用功能與複雜度
- `q.push(x)`：新元素從隊尾排進來 — $O(1)$
- `q.pop()`：排第一名的元素從隊頭出隊離開 — $O(1)$
- `q.front()`：讀取隊頭第一名的元素 — $O(1)$
- `q.back()`：讀取剛進隊尾的元素 — $O(1)$
- `q.empty()`, `q.size()`：狀態檢查 — $O(1)$

### 2. 經典使用情境
- **任務排程與緩衝區（Buffer）**：印表機任務列印、訊息佇列（Message Queue）、連線等待隊列。
- **廣度優先搜尋（BFS）**：二元樹的層序遍歷（Level-order Traversal）、無權重圖尋找最短步數。
- **滑動窗口緩衝**：維護特定長度或時間區間內的資料流。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <queue>

using namespace std;

int main() {
    // 宣告
    queue<int> q;

    // 從隊尾排入 (push)
    q.push(100);
    q.push(200);
    q.push(300);

    // 讀取頭尾
    cout << "隊頭 (第一順位): " << q.front() << endl; // 輸出: 100
    cout << "隊尾 (剛排進來): " << q.back() << endl;  // 輸出: 300

    // 隊頭離開 (pop)
    q.pop();
    cout << "pop 一次後的新隊頭: " << q.front() << endl; // 輸出: 200

    // 狀態驗證與清空
    cout << "隊伍長度: " << q.size() << endl; // 輸出: 2
    while (!q.empty()) {
        q.pop();
    }

    return 0;
}
```
---

## 三、 Hash Table（雜湊表 / 哈希表）

- **運作核心**：鍵值對（Key-Value）映射，透過 Hash Function 瞬間直達記憶體抽屜。
- **語言對應**：C++ `std::unordered_map<Key, Value>` ｜ C# `Dictionary<TKey, TValue>`
- **引入標頭檔**：`#include <unordered_map>`

### 1. 常用功能與複雜度
- `mp[key] = val`：存入或更新數值 — 平均 $O(1)$
- `mp[key]`：讀取數值（**注意：若 key 不存在會自動建立並填入預設值 0 或空值**）— 平均 $O(1)$
- `mp.count(key)`：檢查 key 是否存在（回傳 1 或 0）— 平均 $O(1)$
- `mp.find(key)`：查找迭代器，若 `mp.find(key) == mp.end()` 表示找不到 — 平均 $O(1)$
- `mp.erase(key)`：刪除指定的 key 與其數值 — 平均 $O(1)$

### 2. 經典使用情境
- **瞬間查找（以空間換取極速時間）**：將 $O(N)$ 線性搜尋直接壓到 $O(1)$（LeetCode #1 Two Sum）。
- **計數器 / 頻率統計**：統計字母、單字或數字出現的次數（LeetCode #242 Valid Anagram）。
- **關係對應與快取（Cache）**：儲存對應關係、做記憶化搜尋（Memoization）。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <string>
#include <unordered_map>

using namespace std;

int main() {
    // 宣告: Key 為 string, Value 為 int
    unordered_map<string, int> priceTable;

    // 寫入與更新資料
    priceTable["apple"] = 50;
    priceTable["banana"] = 25;
    priceTable["cherry"] = 100;

    // 讀取資料
    cout << "蘋果價格: " << priceTable["apple"] << endl; // 輸出: 50

    // 檢查 Key 是否存在 (避免用 [] 誤增鍵值)
    if (priceTable.count("banana")) {
        cout << "香蕉在庫！" << endl;
    }
    if (priceTable.count("orange") == 0) {
        cout << "柳橙不存在！" << endl;
    }

    // 刪除鍵值
    priceTable.erase("apple");

    // 遍歷所有 Key-Value
    for (const auto& pair : priceTable) {
        cout << pair.first << " -> " << pair.second << endl;
    }

    return 0;
}
```
---

## 四、 Set（集合）

- **運作核心**：只存 Key（沒有 Value），**保證元素唯一、絕不重複**。
- **語言對應**：C++ `std::unordered_set<Key>` ｜ C# `HashSet<T>`
- **引入標頭檔**：`#include <unordered_set>`

### 1. 常用功能與複雜度
- `st.insert(x)`：加入元素（已存在則自動略過）— 平均 $O(1)$
- `st.count(x)`：檢查元素是否在集合內（1 在，0 不在）— 平均 $O(1)$
- `st.erase(x)`：將指定元素從集合中移除 — 平均 $O(1)$
- `st.size()`：回傳目前集合內不重複元素的數量 — $O(1)$

### 2. 經典使用情境
- **資料去重**：過濾輸入資料中重複的值。
- **黑白名單 / 存在性驗證**：快速確認某個數字或字串是否出現過。
- **防止無限迴圈（Visited 標記）**：在 BFS/DFS 搜尋時記錄「已經走過的節點/座標」。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <unordered_set>

using namespace std;

int main() {
    // 宣告
    unordered_set<int> visited;

    // 插入元素 (重複值自動過濾)
    visited.insert(10);
    visited.insert(20);
    visited.insert(10); // 重複，被自動略過

    cout << "不重複數量: " << visited.size() << endl; // 輸出: 2

    // 檢查元素是否存在
    if (visited.count(20)) {
        cout << "20 在集合中" << endl;
    }

    // 刪除元素
    visited.erase(10);
    if (!visited.count(10)) {
        cout << "10 已被成功移除" << endl;
    }

    return 0;
}
```
---

## 五、 Heap / Priority Queue（堆積 / 優先佇列）

- **運作核心**：內部為完全二元樹，動態維持偏序關係，**頂端永遠是當前的極值**。
- **語言對應**：C++ `std::priority_queue<T>` ｜ C# `PriorityQueue<TElement, TPriority>`
- **引入標頭檔**：`#include <queue>`

### 1. 常用功能與複雜度
- **宣告語法**：
  - *最大堆（預設）*：`priority_queue<int> max_heap;`（頂端永遠是最大值）
  - *最小堆*：`priority_queue<int, vector<int>, greater<int>> min_heap;`（頂端永遠是最小值）
- `pq.push(x)`：放入新元素並自動重新調整排位 — $O(\log N)$
- `pq.top()`：查看當前頂端的極值（最大或最小）— $O(1)$
- `pq.pop()`：移除頂端極值，第二順位自動遞補上來 — $O(\log N)$

### 2. 經典使用情境
- **Top K 問題**：動態尋找海量資料中前 K 個最大/最小的數（LeetCode #215）。
- **串流極值即時維護**：在資料不斷進來時，任何時間點都要能以 $O(1)$ 拿出極值。
- **貪婪演算法與最短路徑**：合併 K 個已排序串列（LeetCode #23）、Dijkstra 演算法。

### 3. C++ 範例代碼
```cpp
#include <iostream>
#include <queue>
#include <vector>

using namespace std;

int main() {
    // 1. 最大堆 (Max Heap): 頂端永遠是最大值
    priority_queue<int> max_heap;
    max_heap.push(30);
    max_heap.push(10);
    max_heap.push(50);

    cout << "Max Heap 頂端 (最大): " << max_heap.top() << endl; // 輸出: 50
    max_heap.pop();
    cout << "pop 後的新頂端: " << max_heap.top() << endl;      // 輸出: 30

    // 2. 最小堆 (Min Heap): 頂端永遠是最小值
    priority_queue<int, vector<int>, greater<int>> min_heap;
    min_heap.push(30);
    min_heap.push(10);
    min_heap.push(50);

    cout << "Min Heap 頂端 (最小): " << min_heap.top() << endl; // 輸出: 10
    min_heap.pop();
    cout << "pop 後的新頂端: " << min_heap.top() << endl;      // 輸出: 30

    return 0;
}
```
---

## 六、 橫向對比總整理

| 資料結構 | C++ 容器名稱 | 查看頂端/極值 | 新增元素 | 任意查找 | 核心記憶特色 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Stack** | `stack<T>` | $O(1)$ (`top`) | $O(1)$ (`push`) | 不支援 | 後進先出，歷史回溯 |
| **Queue** | `queue<T>` | $O(1)$ (`front`) | $O(1)$ (`push`) | 不支援 | 先進先出，公平排程 |
| **Hash Table** | `unordered_map<K, V>` | 不適用 | 平均 $O(1)$ | 平均 $O(1)$ (`find` / `[]`) | Key-Value 映射，空間換時間 |
| **Set** | `unordered_set<K>` | 不適用 | 平均 $O(1)$ | 平均 $O(1)$ (`count`) | 純 Key，自動去除重複項 |
| **Heap** | `priority_queue<T>` | $O(1)$ (`top`) | $O(\log N)$ (`push`) | 不支援 | 動態極值，急診室優先機制 |