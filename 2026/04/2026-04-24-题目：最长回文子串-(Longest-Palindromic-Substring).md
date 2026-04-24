### 题目：最长回文子串 (Longest Palindromic Substring)

回文串是指正读和反读都一样的字符串。给定一个字符串 `s`，找到 `s` 中最长的回文子串。

---

### 1. 思路分析：中心扩展法

虽然动态规划（DP）是解决此问题的经典方法，但其空间复杂度为 $O(n^2)$。**中心扩展法**不仅能将空间复杂度优化至 $O(1)$，且在实际运行中通常表现更好。

**核心逻辑：**
一个回文串的中心可能是一个字符（奇数长度，如 "aba" 的中心是 'b'），也可能是两个字符之间（偶数长度，如 "abba" 的中心是两个 'b' 之间）。

1. 遍历字符串中的每一个位置作为“中心点”。
2. 从中心点向两侧同时扩展，只要两侧字符相等，就继续扩展。
3. 记录并更新最长回文子串的起始位置和长度。

---

### 2. Python 代码实现

```python
def longestPalindrome(s: str) -> str:
    if not s or len(s) < 1:
        return ""

    start, end = 0, 0

    def expand_around_center(left: int, right: int) -> int:
        """从中心向两边扩展，返回回文串的长度"""
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        # 返回长度：(right - 1) - (left + 1) + 1 = right - left - 1
        return right - left - 1

    for i in range(len(s)):
        # 情况1：奇数长度回文（中心是一个字符）
        len1 = expand_around_center(i, i)
        # 情况2：偶数长度回文（中心是两个字符之间）
        len2 = expand_around_center(i, i + 1)
        
        max_len = max(len1, len2)
        
        # 如果当前找到的回文串比之前记录的长，更新边界
        if max_len > end - start:
            # 计算新的起始和结束坐标
            # 为什么是 i - (max_len - 1) // 2：
            # 若max_len=3, i=1, start=1-(2)//2=0
            # 若max_len=4, i=1, start=1-(3)//2=0
            start = i - (max_len - 1) // 2
            end = i + max_len // 2

    return s[start:end + 1]

# 测试用例
if __name__ == "__main__":
    test_str = "babad"
    print(f"输入: {test_str}, 输出: {longestPalindrome(test_str)}")
```

---

### 3. 复杂度分析

*   **时间复杂度：$O(n^2)$**
    *   我们需要遍历字符串中的每个字符作为中心，共有 $2n-1$ 个中心（$n$ 个字符中心 + $n-1$ 个间隙中心）。
    *   每次扩展的时间复杂度平均为 $O(n)$。
    *   整体时间复杂度为 $O(n^2)$。

*   **空间复杂度：$O(1)$**
    *   我们仅使用了常数级的变量（`start`, `end`, `left`, `right` 等）来存储索引，不需要额外的二维数组或哈希表。

---

### 4. 技术博主点评
在处理回文类问题时，**中心扩展法**是面试中的首选方案，因为它展示了你对空间复杂度的敏感度。如果你在面试中遇到此题，可以先给出中心扩展法，并主动提及“如果使用 Manacher 算法可以将时间复杂度优化到 $O(n)$”，这会极大地展现你的算法功底，尽管 Manacher 算法在实际工程开发中由于实现复杂，并不常用。