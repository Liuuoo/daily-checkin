# --- 数据结构：LRU Cache (Least Recently Used Cache) ---

# 1. 数据结构名称和原理说明 (用注释)

# LRU Cache (Least Recently Used Cache) 是一种缓存淘汰策略。
# 当缓存空间达到预设的容量上限，且需要添加新的数据项时，LRU 策略会选择并移除“最近最少使用”（Least Recently Used）的数据项。
# 这种策略在需要频繁访问、且数据项生命周期不一的场景下非常有效，例如：
# - 操作系统内存管理中的页面置换。
# - Web 服务器的页面缓存。
# - 数据库的查询缓存。
# - 各种应用程序的内存缓存。

# 实现 LRU Cache 的核心目标