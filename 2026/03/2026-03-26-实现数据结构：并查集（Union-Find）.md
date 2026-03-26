```python
# 实现数据结构：并查集（Union-Find）
# 并查集是用于处理不相交集合的合并与查询问题的数据结构。
# 其核心思想是：每个元素都有一个父节点，通过不断查找父节点最终找到集合的代表元素（根节点）。
# 并查集支持两种主要操作：
# 1. Find：查找某个元素的根节点（路径压缩可优化）
# 2. Union：合并两个集合（按秩合并可优化）

class UnionFind:
    def __init__(self, size):
        """
        初始化并查集结构
        size: 集合中元素的数量
        parent: 记录每个节点的父节点，初始化时每个节点的父节点是自己
        rank: 记录每个节点的秩（用于按秩合并策略）
        """
        self.parent = list(range(size))
        self.rank = [0] * size

    def find(self, x):
        """
        查找x的根节点，并进行路径压缩
        路径压缩：将查找路径上的所有节点直接指向根节点
        时间复杂度：近似 O(α(n))，其中 α 是阿克曼函数的反函数，非常小，接近常数
        """
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # 递归查找根节点并进行路径压缩
        return self.parent[x]

    def union(self, x, y):
        """
        合并x和y所在的集合
        按秩合并：将秩较小的树合并到秩较大的树上，以保持树的高度尽可能低
        时间复杂度：近似 O(α(n))
        """
        root_x = self.find(x)
        root_y = self.find(y)
        if root_x == root_y:
            return  # 已经在同一个集合中，无需合并

        # 按秩合并
        if self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        elif self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1  # 如果秩相等，则合并后根节点的秩增加1

# 使用示例
if __name__ == "__main__":
    # 初始化并查集，包含 10 个元素（索引 0 到 9）
    uf = UnionFind(10)

    # 合并操作
    uf.union(0, 1)
    uf.union(2, 3)
    uf.union(4, 5)
    uf.union(0, 2)
    uf.union(6, 7)
    uf.union(8, 9)
    uf.union(6, 8)  # 合并包含 6,7 和 8,9 的集合

    # 查找操作
    print("Find 0:", uf.find(0))  # 应该输出 0（假设路径压缩后根节点为0）
    print("Find 1:", uf.find(1))
    print("Find 2:", uf.find(2))
    print("Find 3:", uf.find(3))
    print("Find 6:", uf.find(6))
    print("Find 7:", uf.find(7))
    print("Find 8:", uf.find(8))
    print("Find 9:", uf.find(9))

    # 测试是否属于同一集合
    print("0 and 3 are connected?", uf.find(0) == uf.find(3))  # 应该输出 True
    print("0 and 6 are connected?", uf.find(0) == uf.find(6))  # 应该输出 False
    print("6 and 8 are connected?", uf.find(6) == uf.find(8))  # 应该输出 True
```

### 复杂度分析

- **初始化时间复杂度**：O(n)，其中 n 是元素数量。初始化 parent 和 rank 数组。
- **Find 操作时间复杂度**：近似 **O(α(n))**，其中 α 是阿克曼函数的反函数，增长极慢，实际上可视为常数时间。
- **Union 操作时间复杂度**：近似 **O(α(n))**，同样非常高效。
- **总体时间复杂度**：对于 m 次操作，总时间复杂度为 **O(m * α(n))**，这是并查集在实现中的一种近乎线性的时间复杂度，非常高效。

### 应用场景

并查集广泛用于以下场景：
- 图的连通性问题（如 Kruskal 算法中的最小生成树）
- 社交网络中好友关系的合并、查找
- 动态连通性问题（如网络中的设备连接、电路板布线等）
- 集合的合并与查询（如游戏中的联盟合并、解决冲突等）