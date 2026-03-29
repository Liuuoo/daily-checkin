# 【算法详解】接雨水 (Trapping Rain Water) —— 双指针法的最优解

## 1. 题目描述

**LeetCode 42. Trapping Rain Water**

给定 $n$ 个非负整数表示每个宽度为 $1$ 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

**示例 1：**
```
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。
```

**示例 2：**
```
输入：height = [4,2,0,3,2,5]
输出：9
```

**约束条件：**
- $n == height.length$
- $1 \le n \le 2 \times 10^4$
- $0 \le height[i] \le 10^5$

---

## 2. 思路分析

### 核心原理
对于任意一个位置 $i$，它能接住雨水的量取决于其**左侧最高柱子的高度**和**右侧最高柱子的高度**中的较小值。如果当前位置的高度低于这个较小值，那么差值即为该位置能存储的雨水量。

公式表达为：
$$ Water_i = \max(0, \min(\text{LeftMax}_i, \text{RightMax}_i) - \text{height}[i]) $$

其中：
- $\text{LeftMax}_i$：索引 $i$ 左侧（包含 $i$）的最大高度。
- $\text{RightMax}_i$：索引 $i$ 右侧（包含 $i$）的最大高度。

### 方案演进

1.  **暴力法 (Brute Force)**
    对每个位置遍历左边和右边寻找最大值。
    - 时间复杂度：$O(n^2)$
    - 空间复杂度：$O(1)$
    - *评价：数据量大时超时，不可取。*

2.  **动态规划 (Dynamic Programming)**
    预处理两个数组 `left_max` 和 `right_max`，分别记录每个位置左右两侧的历史最大值。
    - 时间复杂度：$O(n)$ （三次遍历）
    - 空间复杂度：$O(n)$ （两个辅助数组）
    - *评价：效率提升明显，但空间消耗较高。*

3.  **单调栈 (Monotonic Stack)**
    维护一个递减栈，当遇到比栈顶高的柱子时，说明形成了一个“凹槽”，可以计算积水。
    - 时间复杂度：$O(n)$
    - 空间复杂度：$O(n)$
    - *评价：适合处理一维数组中的区间问题，逻辑稍复杂。*

4.  **双指针 (Two Pointers) —— 最优解**
    我们不需要预先知道整个数组的左右最大值。我们可以维护两个指针 `left` 和 `right`，以及当前遇到的左边界最大值 `left_max` 和右边界最大值 `right_max`。
    
    **关键推导：**
    - 若 `height[left] < height[right]`：说明左侧一定有一个比当前 `height[left]` 大的限制（即 `left_max`），而右侧肯定存在一个比 `height[right]` 更大的或相等的限制（因为 `height[right]` 本身就在右侧）。更重要的是，由于 `height[left] < height[right]`，此时 `left_max` 必然小于等于 `right_max`（否则循环会先移动 right 指针或者保持不动）。因此，**决定当前 `left` 位置积水量的瓶颈在于 `left_max`**。
    - 反之，若 `height[left] >= height[right]`，则瓶颈在于 `right_max`。
    - 每次移动较小高度的那一侧指针，并更新对应的最大值。

    这种方法将空间复杂度降到了 $O(1)$，是工程面试中最推荐的解法。

---

## 3. Python 代码实现

```python
from typing import List

class Solution:
    def trap(self, height: List[int]) -> int:
        """
        计算能够接住的雨水总量
        
        算法：双指针法
        核心思想：对于每个柱子，积水量由 min(左侧最大值，右侧最大值) 决定。
        通过比较左右指针指向的高度，只移动高度较小的一侧，从而保证另一侧有足够高的屏障。
        
        Args:
            height: 柱子高度列表
            
        Returns:
            可接住的雨水总量
        """
        # 边界情况处理：如果柱子数量少于 3 根，无法形成凹槽积水
        if not height or len(height) < 3:
            return 0
        
        left, right = 0, len(height) - 1
        left_max, right_max = 0, 0
        total_water = 0
        
        # 当左右指针未相遇时继续循环
        while left <= right:
            # 核心逻辑：比较左右指针的高度
            # 如果左侧高度小于右侧，说明左侧是短板，积水高度由 left_max 决定
            if height[left] < height[right]:
                # 如果当前柱子高度大于等于历史左侧最大高度，更新 left_max
                # 否则，该位置的积水量 = left_max - 当前高度
                if height[left] >= left_max:
                    left_max = height[left]
                else:
                    total_water += left_max - height[left]
                
                # 移动左指针向右
                left += 1
            else:
                # 对称逻辑：右侧高度小于等于左侧
                # 如果当前柱子高度大于等于历史右侧最大高度，更新 right_max
                # 否则，该位置的积水量 = right_max - 当前高度
                if height[right] >= right_max:
                    right_max = height[right]
                else:
                    total_water += right_max - height[right]
                
                # 移动右指针向左
                right -= 1
                
        return total_water

# 测试用例验证
if __name__ == "__main__":
    solver = Solution()
    
    # 案例 1：标准凹槽结构
    h1 = [0,1,0,2,1,0,1,3,2,1,2,1]
    print(f"输入：{h1}")
    print(f"输出：{solver.trap(h1)}")  # 期望输出：6
    
    # 案例 2：无中间低洼
    h2 = [4,2,0,3,2,5]
    print(f"输入：{h2}")
    print(f"输出：{solver.trap(h2)}")  # 期望输出：9
    
    # 案例 3：单调递增/递减
    h3 = [1, 2, 3]
    print(f"输入：{h3}")
    print(f"输出：{solver.trap(h3)}")  # 期望输出：0
    
    # 案例 4：单峰结构
    h4 = [3, 0, 0, 2, 0, 4]
    print(f"输入：{h4}")
    print(f"输出：{solver.trap(h4)}")  # 期望输出：10
```

---

## 4. 时间和空间复杂度分析

### 时间复杂度：$O(n)$
- **原因**：代码中只有一个 `while` 循环，指针 `left` 从 $0$ 移动到 $n-1$，指针 `right` 从 $n-1$ 移动到 $0$。在整个过程中，每个元素最多被访问一次。因此，总操作次数与输入规模 $n$ 成线性关系。

### 空间复杂度：$O(1)$
- **原因**：除了用于计数的变量（`left`, `right`, `left_max`, `right_max`, `total_water`）外，没有使用任何随输入规模 $n$ 增长而增长的额外数据结构（如数组、哈希表或栈）。无论输入多大，占用的内存都是恒定的。

### 总结对比
| 方法 | 时间复杂度 | 空间复杂度 | 适用场景 |
| :--- | :--- | :--- | :--- |
| 暴力法 | $O(n^2)$ | $O(1)$ | 仅适用于极小规模数据教学演示 |
| 动态规划 | $O(n)$ | $O(n)$ | 需要频繁查询某点极大值的场景 |
| 单调栈 | $O(n)$ | $O(n)$ | 解决“下一个更大元素”类问题 |
| **双指针** | **$O(n)$** | **$O(1)$** | **面试首选，最优性能解** |