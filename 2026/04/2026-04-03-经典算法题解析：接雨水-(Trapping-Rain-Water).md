# 经典算法题解析：接雨水 (Trapping Rain Water)

## 1. 题目描述

**力扣 (LeetCode) 第 42 题**

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

**输入示例：**
```text
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。
```

**约束条件：**
- $n == \text{height.length}$
- $1 \le n \le 2 \times 10^4$
- $0 \le \text{height}[i] \le 10^5$

---

## 2. 思路分析与推导

这道题的核心在于理解“水位”是如何形成的。对于任意一个位置 `i`，它能接住的雨水量取决于它**左边最高的柱子**和**右边最高的柱子**。

### 2.1 数学模型
设 `height[i]` 为当前位置柱子的高度，`left_max[i]` 为位置 `i` 左侧（包括 `i`）的最高柱子高度，`right_max[i]` 为位置 `i` 右侧（包括 `i`）的最高柱子高度。
则位置 `i` 处的积水高度为：
$$ \text{water}[i] = \max(0, \min(\text{left\_max}[i], \text{right\_max}[i]) - \text{height}[i]) $$
总积水量即为所有位置 `water[i]` 的累加。

直观上理解，水往低处流，最终水位会被两侧的矮墙限制，即“木桶效应”。当前位置的水位高度等于左右两侧最高柱子的较小值减去当前柱子高度。

### 2.2 优化路径

#### 方案一：暴力枚举 (不推荐)
遍历每个位置，向左找最大值，向右找最大值。
- 时间复杂度：$O(N^2)$
- 空间复杂度：$O(1)$
当 $N=2 \times 10^4$ 时，运算量约为 $4 \times 10^8$，会导致超时。

#### 方案二：动态规划 / 预计算 (标准解法)
利用两个辅助数组 `left_max` 和 `right_max` 分别存储每个位置的左右最大高度。
- 预处理左数组：`left_max[i] = max(left_max[i-1], height[i])`
- 预处理右数组：`right_max[i] = max(right_max[i+1], height[i])`
- 最后遍历一次计算总和。
- 时间复杂度：$O(N)$
- 空间复杂度：$O(N)$
这是最直观的线性解法，但占用了额外的内存。

#### 方案三：双指针 (最优解法)
我们能否在 $O(1)$ 空间内完成？
关键在于观察上述公式中的 $\min(\text{left\_max}, \text{right\_max})$。
假设我们在数组的两端维护两个指针 `left` 和 `right`，以及两个变量 `left_max` 和 `right_max` 记录已遍历过的左右最大高度。

**核心逻辑：**
如果 `left_max < right_max`，那么对于当前的 `left` 位置来说，无论右边的具体数值如何变化（只要 `right_max` 是全局已知且大于 `left_max`），该位置的水位一定由 `left_max` 决定。因为即使右边有一个更高的柱子，`min` 取值的瓶颈依然在左边的 `left_max`；而如果右边有比当前 `right_max` 更低的柱子，那也不会影响当前 `left` 位置的水位，因为我们已经知道 `left_max` 更小。

反之亦然。因此，我们只需要移动那个对应最大值较小的指针，即可保证计算的准确性，从而无需预先存储整个数组。

---

## 3. 完整的 Python 代码实现

```python
from typing import List

class Solution:
    def trap(self, height: List[int]) -> int:
        """
        使用双指针法解决接雨水问题
        
        :param height: 非负整数列表，表示柱子高度
        :return: 能接到的雨水总量
        """
        # 边界检查：如果柱子数量少于 3，无法形成凹槽接水
        if not height or len(height) < 3:
            return 0
        
        # 初始化左右指针
        left = 0
        right = len(height) - 1
        
        # 初始化左右两侧的最大高度
        # 初始化为 0 或小于任意 height[i] 的值均可，这里设为 0
        left_max = 0
        right_max = 0
        
        total_water = 0
        
        # 当左右指针未相遇时循环
        while left < right:
            # 更新左侧最大高度和右侧最大高度
            # 注意：这里先比较的是之前的状态，再根据状态移动指针并更新当前状态
            if height[left] <= height[right]:
                # 如果左侧高度较小，说明左侧是瓶颈
                # 此时，当前位置 left 的水位由 left_max 决定
                
                # 如果当前柱子低于历史最大值，则存在积水
                if height[left] >= left_max:
                    left_max = height[left]  # 更新左侧最大高度
                else:
                    # 计算积水：min(left_max, right_max) - height[left]
                    # 由于进入该分支隐含 height[left] <= height[right] 
                    # 且我们已经保证 left_max <= right_max (通过全局视角)，
                    # 实际上积水高度就是 left_max - height[left]
                    total_water += left_max - height[left]
                
                # 移动左指针向右
                left += 1
            else:
                # 如果右侧高度较小，说明右侧是瓶颈
                # 同理，当前位置 right 的水位由 right_max 决定
                
                if height[right] >= right_max:
                    right_max = height[right]  # 更新右侧最大高度
                else:
                    total_water += right_max - height[right]
                
                # 移动右指针向左
                right -= 1
                
        return total_water

# 测试用例
if __name__ == "__main__":
    sol = Solution()
    heights1 = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
    print(f"Input: {heights1}")
    print(f"Output: {sol.trap(heights1)}")  # Expected: 6
    
    heights2 = [4, 2, 0, 3, 2, 5]
    print(f"\nInput: {heights2}")
    print(f"Output: {sol.trap(heights2)}")  # Expected: 9
```

---

## 4. 时间和空间复杂度分析

### 时间复杂度：$O(N)$
- 算法仅对数组进行了一次遍历。`left` 指针从左向右移动，`right` 指针从右向左移动，两者总共移动的次数不超过 $N$ 次。
- 每次循环内的操作都是常数级别的比较和加法。
- 因此，总体时间复杂度为线性级别。

### 空间复杂度：$O(1)$
- 我们只使用了 `left`, `right`, `left_max`, `right_max`, `total_water` 这几个固定数量的变量。
- 没有创建任何与输入规模 $N$ 相关的辅助数组（区别于动态规划解法）。
- 因此，额外空间占用为常量级别。

---

## 5. 拓展思考与面试点评

1.  **为什么 `left_max` 和 `right_max` 不需要同时维护？**
    很多初学者会疑惑为什么不先算出两边的最大值。关键在于**局部最优性**。当我们确信 `left_max < right_max` 时，对于 `left` 指针指向的位置，其右侧必然存在一个高度至少为 `right_max` 的屏障（因为是从右往左扫描过来的，且 `right_max` 记录了历史最大值），而 `left_max` 是左侧已知的最大高度。既然 `left_max` 更小，它就是限制水位的瓶颈，右侧是否还有更高的柱子不影响结果。

2.  **单调栈解法 (Monotonic Stack)**
    除了双指针，这道题还可以用单调栈解决。维护一个高度递减的栈，当遇到比栈顶高的柱子时，说明形成了一个“凹槽”，可以弹出栈顶元素计算这一层的积水。这种方法更适合处理“横向切片”的积水计算，但在本题中双指针的空间效率更高。

3.  **工程化建议**
    在实际系统设计中，如果遇到类似的数据流处理问题（数据量极大，无法一次性加载到内存），双指针法是首选，因为它只需要流式读取数据，不需要随机访问存储的历史数据。这体现了算法选择对系统架构的影响。