```python
# 数据结构：并查集（Union-Find）
# 原理说明：并查集是一种用于处理不相交集合合并与查询的数据结构。它支持两种主要操作：
# 1. 查找（find）：确定一个元素所属的集合的代表元素（通常是根节点）。
# 2. 合并（union）：将两个不同的集合合并成一个。
# 使用路径压缩和按秩合并（union by rank）优化查找与合并操作的时间复杂度，使其接近常数时间。

class UnionFind:
    def __init__(self, size):
        # 初始化父数组，每个元素的父节点初始指向自己
        self.parent = list(range(size))
        # 初始化秩数组（用于按秩合并），初始值为0
        self.rank = [0] * size

    def find(self, x):
        # 路径压缩：递归查找父节点，并将路径上的所有节点直接指向根节点
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        # 查找x和y的根节点
        root_x = self.find(x)
        root_y = self.find(y)

        # 如果根节点相同，说明已经在同一集合
        if root_x == root_y:
            return

        # 按秩合并：将秩较小的集合合并到秩较大的集合下
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            # 秩相同的情况下，合并后秩加1
            self.parent[root_y] = root_x
            self.rank[root_x] += 1

# 使用示例
if __name__ == "__main__":
    # 初始化包含10个元素的并查集
    uf = UnionFind(10)

    # 合并一些元素
    uf.union(0, 1)
    uf.union(2, 3)
    uf.union(4, 5)
    uf.union(6, 7)
    uf.union(8, 9)

    # 再合并两个集合
    uf.union(0, 2)
    uf.union(4, 6)
    uf.union(1, 8)

    # 测试查找操作
    print("Find(0):", uf.find(0))  # 应该与find(1), find(2), find(8)相同
    print("Find(1):", uf.find(1))
    print("Find(2):", uf.find(2))
    print("Find(8):", uf.find(8))

    print("Find(3):", uf.find(3))  # 应该与find(0)相同
    print("Find(4):", uf.find(4))  # 应该与find(6), find(7), find(9)相同
    print("Find(6):", uf.find(6))
    print("Find(7):", uf.find(7))
    print("Find(9):", uf.find(9))

    # 检测集合关系
    print("Find(0) == Find(3)?", uf.find(0) == uf.find(3))
    print("Find(4) == Find(7)?", uf.find(4) == uf.find(7))
    print("Find(0) == Find(4)?", uf.find(0) == uf.find(4))

# 复杂度分析：
# - find(x)：使用路径压缩后，近似为 O(α(n))，其中 α(n) 是阿克曼函数的反函数，对于实际应用来说几乎是常数时间。
# - union(x, y)：使用按秩合并后，同样为 O(α(n))。
# - α(n) 是一个非常缓慢增长的函数，对于n ≤ 2^200，其值不超过5。
# 并查集在处理动态连通性问题时非常高效，比如网络连接、图像处理、Kruskal算法等。
```