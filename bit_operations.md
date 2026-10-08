# Bit Operation 速查手冊 (C++ 版)

本手冊整理位元運算的基本運算子、常用技巧、位元遮罩（bitmask）、常見題型模式，以及韌體 / 嵌入式面試常考的暫存器操作與對齊技巧。

---

## 〇、使用前先記住的通則

1. **位元運算盡量用無號型別**：`uint32_t`、`uint64_t`（`#include <cstdint>`）。有號整數的移位在邊界情況下容易踩到 UB 或實作定義行為。
2. **移位量必須小於型別位寬**：對 32 位元整數做 `x << 32` 或 `x >> 32` 是 **UB**。
3. **`1 << 31` 在 `int` 上有風險**：會移進符號位。請寫成 `1u << 31`，需要 64 位元就寫 `1ULL << n`。
4. **有號數右移**：負數右移在 C++20 之前是實作定義（實務上多為算術右移，補符號位）；要「邏輯右移」請先轉成無號型別。
5. **運算子優先順序陷阱**：`==`、`<` 等比較運算子的優先順序**高於** `&`、`^`、`|`。`x & 1 == 0` 會被解析成 `x & (1 == 0)`，必須寫成 `(x & 1) == 0`。另外 `+`、`-` 的優先順序高於移位：`1 << n + 1` 等於 `1 << (n + 1)`。**不確定就加括號。**
6. **整數提升（integer promotion）**：`uint8_t` / `uint16_t` 參與運算會先提升成 `int`，例如 `~x`（`x` 是 `uint8_t`）的結果是 `int`，高位全是 1，需要時轉型：`(uint8_t)~x`。
7. **相關標頭檔**：`<cstdint>`（固定寬度型別）、`<bitset>`、C++20 的 `<bit>`（`std::popcount`、`std::countr_zero`、`std::bit_width`、`std::rotl` / `std::rotr`）。

---

## 一、基本運算子

| 運算子 | 名稱 | 規則 | 範例（4 位元） |
| :---: | :--- | :--- | :--- |
| `&` | AND | 兩邊都是 1 才是 1 | `1100 & 1010 = 1000` |
| `\|` | OR | 有一邊是 1 就是 1 | `1100 \| 1010 = 1110` |
| `^` | XOR | 兩邊不同才是 1 | `1100 ^ 1010 = 0110` |
| `~` | NOT | 每個位元反轉 | `~1100 = 0011` |
| `<<` | 左移 | 往高位移，低位補 0；無溢位時等於乘 $2^k$ | `0011 << 2 = 1100` |
| `>>` | 右移 | 往低位移；無號數高位補 0，等於除 $2^k$（向下取整）| `1100 >> 2 = 0011` |

**二補數（two's complement）**：`-x == ~x + 1`，`~x == -x - 1`。這是很多技巧（例如 `x & -x`）的基礎。

**XOR 的性質**（極重要）：
- `a ^ a = 0`
- `a ^ 0 = a`
- 交換律與結合律：`a ^ b ^ a = b`
- 所以「成對出現的數字會互相抵消」，常用來找落單的數。

---

## 二、單一位元操作（第 i 位，從 0 開始）

```cpp
x |=  (1u << i);      // 設為 1  (set)
x &= ~(1u << i);      // 清為 0  (clear)
x ^=  (1u << i);      // 翻轉    (toggle)
bool b = (x >> i) & 1u;   // 讀取第 i 位 (test)
```

| 目的 | 寫法 | 說明 |
| :--- | :--- | :--- |
| 判斷奇偶 | `x & 1` | 1 為奇數，0 為偶數 |
| 乘 2 / 除 2 | `x << 1` / `x >> 1` | 僅適用於非負數 |
| 對 $2^k$ 取餘 | `x & ((1u << k) - 1)` | 取低 k 位 |
| 低 n 位全為 1 的遮罩 | `(1u << n) - 1` | `n == 32` 時會 UB，需特別處理：`n == 32 ? ~0u : (1u << n) - 1` |
| 兩數是否異號 | `(a ^ b) < 0` | 用於有號整數 |

---

## 三、經典位元技巧

### 1. 清除最低位的 1：`x & (x - 1)`
把 `x` 最右邊的那個 1 變成 0。
```
x     = 1011000
x - 1 = 1010111
x&(x-1) = 1010000
```
應用：
- **判斷是否為 2 的冪**：`x > 0 && (x & (x - 1)) == 0`
- **計算 1 的個數**（Brian Kernighan 演算法）：不斷清除最低位的 1，直到變 0，複雜度 $O(\text{1 的個數})$。

### 2. 取出最低位的 1：`x & -x`
只保留最右邊的那個 1。
```
x  = 1011000
-x = 0101000
x&-x = 0001000
```
應用：樹狀陣列（Fenwick Tree）、找最低位的 1 的位置。建議在**無號型別**上使用（有號的 `INT_MIN` 取負會溢位）。

### 3. 計算 1 的個數（popcount）
```cpp
// 方法一：Brian Kernighan，O(1 的個數)
int popcount(uint32_t x) {
    int c = 0;
    while (x) {
        x &= x - 1;   // 清除最低位的 1
        ++c;
    }
    return c;
}

// 方法二：內建函式
__builtin_popcount(x);        // GCC / Clang，32 位元
__builtin_popcountll(x);      // 64 位元
// C++20: std::popcount(x)    // <bit>
```

### 4. 最低 / 最高位的 1 的位置
```cpp
__builtin_ctz(x);             // count trailing zeros：最低位 1 的索引（x 不可為 0）
31 - __builtin_clz(x);        // count leading zeros：最高位 1 的索引（x 不可為 0）
// C++20: std::countr_zero(x)、std::bit_width(x) - 1
```
> `__builtin_ctz` / `__builtin_clz` 對 0 的行為未定義，使用前先確認 `x != 0`。

### 5. 用 XOR 找落單的數
```cpp
int singleNumber(vector<int>& nums) {
    int r = 0;
    for (int x : nums) r ^= x;   // 成對的會抵消，只剩落單的
    return r;
}
```

### 6. XOR 交換兩數（了解即可，不建議實務使用）
```cpp
a ^= b;
b ^= a;
a ^= b;
```
> 若 `a` 和 `b` 是**同一個變數**（同一位址），結果會變成 0。面試可以提，但實務直接用 `std::swap`。

### 7. 翻轉位元（reverse bits）
```cpp
uint32_t reverseBits(uint32_t n) {
    uint32_t r = 0;
    for (int i = 0; i < 32; i++) {
        r = (r << 1) | (n & 1);   // 把 n 的最低位接到 r 的尾端
        n >>= 1;
    }
    return r;
}
```

### 8. 不用加減乘除做加法
加法 = **不進位的和**（XOR）加上**進位**（AND 後左移一位），重複直到沒有進位。
```cpp
int getSum(int a, int b) {
    while (b != 0) {
        unsigned carry = (unsigned)(a & b) << 1;  // 進位
        a ^= b;                                    // 不進位的和
        b = carry;
    }
    return a;
}
```

### 9. 計算 0 到 n 每個數的 1 的個數（DP）
```cpp
vector<int> countBits(int n) {
    vector<int> ans(n + 1, 0);
    for (int i = 1; i <= n; i++) {
        ans[i] = ans[i >> 1] + (i & 1);   // 去掉最低位後的結果，加上最低位
        // 等價寫法：ans[i] = ans[i & (i - 1)] + 1;
    }
    return ans;
}
```

---

## 四、位元遮罩（Bitmask）與狀態壓縮

用一個整數的每個位元表示「某個元素選或不選」，n 個元素共有 $2^n$ 種狀態（通常 n ≤ 20 左右）。

### 1. 枚舉所有子集
```cpp
int n = nums.size();
for (int mask = 0; mask < (1 << n); mask++) {
    vector<int> subset;
    for (int i = 0; i < n; i++) {
        if ((mask >> i) & 1) subset.push_back(nums[i]);
    }
    // 處理 subset
}
```
複雜度 $O(n \cdot 2^n)$。

### 2. 枚舉某個 mask 的所有非空子集
```cpp
for (int s = mask; s > 0; s = (s - 1) & mask) {
    // s 是 mask 的一個非空子集
}
// 空集合要另外處理
```

### 3. 用位元表示字元集合
```cpp
int mask = 0;
for (char c : word) mask |= 1 << (c - 'a');   // 26 個字母，各佔一位
// 兩個字串沒有共同字母：(maskA & maskB) == 0
```
應用：判斷字串是否有重複字元、字串之間的字元集合是否重疊。

### 4. 位元遮罩 DP（進階）
`dp[mask]` 表示「已選取集合為 mask」的最佳值，適合小規模的排列 / 指派類問題。

---

## 五、常見題型模式（看到什麼，想到什麼）

| 題目特徵 | 想到的技巧 |
| :--- | :--- |
| 成對出現、只有一個落單 | 全部 XOR |
| 缺一個數、多一個字元 | 與索引或另一個集合一起 XOR |
| 兩個落單的數 | 全部 XOR 後，用 `x & -x` 找出一個區分位，將數字分成兩組再各自 XOR |
| 其他數出現 3 次，一個出現 1 次 | 逐位元統計（每一位的 1 的個數 mod 3）|
| 判斷 2 的冪、4 的冪 | `x & (x - 1)`，4 的冪額外檢查位置（例如 `x & 0x55555555`）|
| 計算 1 的個數 / 漢明距離 | `x & (x - 1)`、`a ^ b` 後 popcount |
| 補數、翻轉 | 先找出全 1 的遮罩，再 XOR |
| 不能用 `+` `-` | XOR 加上進位 |
| 子集 / 狀態壓縮 / 字元集合 | bitmask |
| 範圍 AND | 找兩端的最長共同前綴 |
| 兩數最大 XOR | 逐位元貪心，搭配前綴集合或 Trie |

---

## 六、韌體 / 嵌入式專區

### 1. 暫存器操作的標準巨集
```cpp
#define BIT(n)            (1u << (n))
#define SET_BIT(reg, n)   ((reg) |=  BIT(n))
#define CLEAR_BIT(reg, n) ((reg) &= ~BIT(n))
#define TOGGLE_BIT(reg,n) ((reg) ^=  BIT(n))
#define READ_BIT(reg, n)  (((reg) >> (n)) & 1u)
```

### 2. 讀取 / 寫入多位元欄位（field）
假設欄位寬度對應的**未移位遮罩**為 `MASK`（例如 3 位元就是 `0x7`），起始位置為 `SHIFT`：
```cpp
// 讀取欄位
uint32_t field = (reg >> SHIFT) & MASK;

// 寫入欄位（先清除該欄位，再填入新值）
reg = (reg & ~(MASK << SHIFT)) | ((val & MASK) << SHIFT);
```
重點：**先清除、再寫入**，而且要先用 `& MASK` 限制 `val`，避免超出欄位寬度污染相鄰位元。

### 3. 暫存器存取的注意事項
- 硬體暫存器要用 `volatile` 修飾，避免編譯器把讀寫最佳化掉：`volatile uint32_t* const REG = (volatile uint32_t*)0x40000000;`
- `reg |= BIT(n)` 其實是 **讀取、修改、寫回（read-modify-write）**，**不是原子操作**。若 ISR 與主程式都會修改同一個暫存器或變數，可能發生競爭條件。常見解法：暫時關閉中斷、使用硬體提供的 set / clear 專用暫存器（例如 SET / CLR / TOGGLE 暫存器）、或使用原子操作。
- 有些暫存器的位元是 **write-1-to-clear**（寫 1 清除旗標），此時用 `reg |= BIT(n)` 可能會誤清其他旗標，要依資料手冊確認寫法。

### 4. Bit-field（位元欄位）與可攜性
```cpp
struct Flags {
    uint8_t enable : 1;
    uint8_t mode   : 3;
    uint8_t reserved : 4;
};
```
bit-field 的**排列順序、對齊、跨型別邊界的行為都是實作定義**，不同編譯器與位元組順序（endianness）可能不同，所以**需要與硬體暫存器或通訊協定對應時，通常改用遮罩加移位**，可攜性較好。面試被問到時，這是很好的加分回答。

### 5. 位元組順序（Endianness）與位元組交換
```cpp
// 判斷小端（little-endian）
bool isLittleEndian() {
    uint16_t x = 1;
    return *reinterpret_cast<unsigned char*>(&x) == 1;
}

// 32 位元位元組順序交換
uint32_t swap32(uint32_t x) {
    return ((x & 0x000000FFu) << 24) |
           ((x & 0x0000FF00u) <<  8) |
           ((x >>  8) & 0x0000FF00u) |
           ( x >> 24);
}

// 位元組內交換高低 4 位元（nibble）
uint8_t swapNibble(uint8_t x) {
    return (uint8_t)(((x & 0x0F) << 4) | ((x & 0xF0) >> 4));
}
```

### 6. 循環移位（rotate）
```cpp
uint32_t rotl(uint32_t x, unsigned n) {
    n &= 31;                              // 避免移位量 >= 32 的 UB
    return n ? (x << n) | (x >> (32 - n)) : x;
}
// C++20: std::rotl(x, n)、std::rotr(x, n)
```

### 7. 位址對齊（alignment）
對齊值 `a` 必須是 2 的冪：
```cpp
bool isAligned(uintptr_t addr, uintptr_t a) { return (addr & (a - 1)) == 0; }
uintptr_t alignUp(uintptr_t addr, uintptr_t a)   { return (addr + a - 1) & ~(a - 1); }
uintptr_t alignDown(uintptr_t addr, uintptr_t a) { return addr & ~(a - 1); }
```
應用：記憶體池、DMA 緩衝區、結構體 padding 的說明。

### 8. 韌體面試常見的位元題
- 設定、清除、翻轉某一位；一次設定 / 清除多個位元。
- 計算 1 的個數、判斷 2 的冪、反轉位元、交換奇偶位元。
- 判斷 endianness、位元組交換、將多個位元組組合成整數（例如 `(b3 << 24) | (b2 << 16) | (b1 << 8) | b0`）。
- 位址對齊計算、環形緩衝區中用位元遮罩取代 `%`（容量為 2 的冪時：`idx & (N - 1)`）。
- 說明 `volatile` 的用途、read-modify-write 為什麼不是原子操作。

---

## 七、常見陷阱總整理

1. **優先順序**：`x & 1 == 0` 是錯的，要寫 `(x & 1) == 0`。
2. **移位量過大**：`x << 32`（32 位元型別）是 UB；變數移位量先檢查範圍。
3. **`1 << 31` 與 `1 << 63`**：用 `1u`、`1ULL`。
4. **有號數右移與左移**：負數的行為容易出問題，位元運算改用無號型別。
5. **`~` 與整數提升**：小型別取反後要轉型回來。
6. **`__builtin_ctz` / `__builtin_clz` 傳入 0**：行為未定義。
7. **`x & -x` 用在有號 `INT_MIN`**：取負溢位，改用無號型別。
8. **XOR 交換同一變數**：會變成 0。
9. **用 `int` 存 64 位元遮罩**：會截斷，要用 `long long` / `uint64_t`。
10. **暫存器 read-modify-write**：不是原子操作，與中斷共用時要保護。

---

## 八、複習順序建議

1. 先熟記：基本運算子、單一位元的 set / clear / toggle / test、二補數。
2. 掌握三個核心技巧：`x & (x - 1)`、`x & -x`、XOR 的抵消性質。
3. 練習：popcount、2 的冪、落單的數、翻轉位元、不用加減做加法。
4. 進階：bitmask 子集枚舉、字元集合壓縮。
5. 韌體專區：暫存器欄位讀寫、對齊、endianness、rotate，這部分可以自己手寫一遍，確保不看筆記也寫得出來。
