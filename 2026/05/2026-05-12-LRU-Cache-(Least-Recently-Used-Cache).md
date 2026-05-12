# LRU Cache (Least Recently Used Cache)

## 原理说明 (Principle Explanation)

LRU Cache（Least Recently Used Cache）是一种缓存淘汰策略。当缓存已满时，如果需要添加新的数据项，LRU策略会移除“最近最少使用”的数据项，为新数据腾出空间。这是一种非常常见的缓存管理机制，广泛应用于操作系统（页面置换）、数据库、Web服务器和内存数据库等场景，以优化性能和资源利用率。

要实现LRU Cache，我们需要高效地完成以下操作：
1.  **查找键 (Lookup Key):** 快速判断一个键是否存在于缓存中。
2.  **访问键 (Access Key):** 当一个键被访问时（无论是读取还是写入