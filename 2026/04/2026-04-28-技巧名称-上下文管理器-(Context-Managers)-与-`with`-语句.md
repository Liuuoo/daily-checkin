```python
# 技巧名称: 上下文管理器 (Context Managers) 与 `with` 语句

# 简介:
# Python 的 `with` 语句提供了一种简洁且安全的方式来管理资源，
# 确保资源（如文件、锁、网络连接）在使用后能被正确地获取和释放，
# 即使在 `with` 块内部发生异常时也能如此。
# 上下文管理器是实现了 `__enter__` 和 `__exit__` 方法的对象。
# 它们是实现“资源获取即初始化 (Resource Acquisition Is Initialization, RAII)”
# 模式的 Pythonic 方式。

# 使用场景说明:
# 许多编程任务都涉及到资源的获取、使用和释放。例如：
# 1.  **文件操作:** 打开文件、读取/写入内容、关闭文件。如果文件未关闭，可能导致数据丢失或文件句柄泄漏。
# 2.  **网络连接/数据库连接:** 连接到服务器/数据库、执行操作、断开连接。连接不关闭可能耗尽连接池或导致资源泄漏。
# 3.  **线程/进程锁:** 获取锁、执行临界区代码、释放锁。锁不释放可能导致死锁。
# 4.  **临时修改系统状态:** 改变当前工作目录、修改环境变量等，在操作完成后需要恢复原始状态。
#
# 传统上，这些场景通常需要使用 `try...finally` 结构来确保资源无论如何都能被释放，
# 但这会导致代码冗长且可读性差，尤其是在嵌套多个资源管理时。
#
# 上下文管理器和 `with` 语句的优势在于：
# -   **自动化资源管理:** 开发者无需显式调用关闭/释放方法。
# -   **异常安全:** 即使 `with` 块中发生异常，`__exit__` 方法也会被调用，确保资源被正确清理。
# -   **代码简洁:** 将资源管理逻辑封装起来，使业务逻辑更清晰。

# 完整可运行的示例代码:

# --- 示例1: 使用内置的 `open()` 函数作为上下文管理器 ---
# `open()` 函数返回的文件对象就是一个内置的上下文管理器。
print("--- 示例1: 使用内置的 `open()` 函数 ---")
file_path = "my_log.txt"
try:
    with open(file_path, "w") as f:
        print(f"文件 '{file_path}' 已打开。")
        f.write("这是通过上下文管理器写入的第一行。\n")
        f.write("这是通过上下文管理器写入的第二行。\n")
        # 模拟一个错误，看文件是否仍能正确关闭
        # raise ValueError("写入过程中发生了一个错误!")
    print(f"文件 '{file_path}' 已成功写入并关闭。")
except ValueError as e:
    print(f"捕获到异常: {e}，但文件 '{file_path}' 仍会在此之前被关闭。")

# 验证文件是否已关闭并读取其内容
with open(file_path, "r") as f:
    print(f"文件 '{file_path}' 的内容:\n{f.read().strip()}")
print("-" * 40)


# --- 示例2: 自定义一个上下文管理器 (模拟数据库连接) ---
# 通过定义一个类并实现 `__enter__` 和 `__exit__` 方法来创建自定义上下文管理器。
class DatabaseConnection:
    def __init__(self, db_name):
        self.db_name = db_name
        self.connection = None
        print(f"初始化数据库连接对象: {self.db_name}")

    def __enter__(self):
        """
        进入 with 块时执行。
        负责获取资源（建立连接）。
        返回值会绑定到 `as` 关键字后面的变量上。
        """
        print(f"  __enter__: 正在连接数据库 '{self.db_name}'...")
        # 模拟连接操作
        self.connection = f"Connected to {self.db_name}"
        print(f"  __enter__: 数据库 '{self.db_name}' 连接成功。")
        return self.connection  # 返回连接对象

    def __exit__(self, exc_type, exc_val, exc_tb):
        """
        离开 with 块时执行（无论是正常退出还是发生异常）。
        负责释放资源（关闭连接）。
        参数:
        - exc_type: 异常类型 (如 ValueError, None if no exception)
        - exc_val: 异常值 (异常实例, None if no exception)
        - exc_tb: 异常的跟踪信息 (traceback, None if no exception)
        如果此方法返回 True，则表示它已处理了异常，异常不会继续向上抛出。
        如果返回 False (或不返回任何值)，则异常会继续传播。
        """
        print(f"  __exit__: 正在关闭数据库 '{self.db_name}' 连接...")
        # 模拟关闭操作
        self.connection = None
        print(f"  __exit__: 数据库 '{self.db_name}' 连接已关闭。")

        if exc_type:
            print(f"  __exit__: 在 with 块中捕获到异常: {exc_type.__name__}: {exc_val}")
            # 这里选择不处理异常，让它继续传播，所以返回 False 或不返回
            return False
        return True # 通常情况下，返回 True 或 None 都可以，但为了明确，我们返回 True

print("\n--- 示例2: 自定义上下文管理器 (正常执行) ---")
with DatabaseConnection("my_app_db") as db:
    print(f"在 with 块内部，当前数据库连接状态: {db}")
    # 模拟一些数据库操作
    print("  执行一些数据库查询...")
print("With 块结束，数据库连接已释放。")

print("\n--- 示例2: 自定义上下文管理器 (带异常处理) ---")
try:
    with DatabaseConnection("another_db") as db:
        print(f"在 with 块内部，当前数据库连接状态: {db}")
        # 模拟一个数据库操作失败的异常
        raise ConnectionError("数据库操作失败，需要回滚或重试！")
    print("此行不会被执行，因为上面抛出了异常。")
except ConnectionError as e:
    print(f"在 with 块外部捕获到异常: {e}")
print("With 块结束，即使有异常，数据库连接也已释放。")
print("-" * 40)


# --- 示例3: 使用 `contextlib.contextmanager` 装饰器 ---
# 对于简单的上下文管理器，特别是那些只涉及少量设置和清理代码的，
# 使用 `contextlib.contextmanager` 装饰器可以避免编写完整的类，
# 而是使用生成器函数来定义上下文管理器，代码更简洁。
import time
from contextlib import contextmanager

@contextmanager
def timer(name):
    """
    一个简单的计时器上下文管理器。
    `yield` 之前的代码相当于 `__enter__` 方法，
    `yield` 之后的代码（包括 `finally` 块）相当于 `__exit__` 方法。
    `yield` 语句的值会绑定到 `as` 关键字后面的变量上。
    """
    start_time = time.time()
    print(f"计时器 '{name}' 启动...")
    try:
        yield  # 控制权交给 with 块，with 块执行完毕后，控制权返回到这里
    finally:
        end_time = time.time()
        duration = end_time - start_time
        print(f"计时器 '{name}' 结束，耗时: {duration:.4f} 秒。")

print("\n--- 示例3: 使用 `contextlib.contextmanager` 装饰器 (正常执行) ---")
with timer("复杂计算"):
    print("  正在进行一些耗时操作...")
    sum_val = 0
    for i in range(1_000_000):
        sum_val += i
    print(f"  计算结果: {sum_val}")
    time.sleep(0.1) # 模拟其他操作
print("With 块结束。")

print("\n--- 示例3: 使用 `contextlib.contextmanager` 装饰器 (带异常处理) ---")
try:
    with timer("可能出错的操作"):
        print("  尝试执行可能出错的操作...")
        time.sleep(0.05)
        raise RuntimeError("操作中途失败!")
except RuntimeError as e:
    print(f"在 with 块外部捕获到异常: {e}")
print("With 块结束，即使有异常，计时器也已停止。")
print("-" * 40)

# 最佳实践建议:
# 1.  **优先使用 `with` 语句:** 当处理任何需要获取和释放资源的场景时，首先考虑使用 `with` 语句。它比手动 `try...finally` 更简洁、更安全，因为它确保了资源无论如何都会被释放。
#
# 2.  **正确实现 `__enter__` 和 `__exit__`:**
#     *   `__enter__` 方法应该返回 `with ... as var:` 语句中 `var` 所绑定的对象（通常是资源本身）。如果不需要绑定任何对象，可以返回 `self` 或 `None`。
#     *   `__exit__` 方法的三个参数 (`exc_type`, `exc_val`, `exc_tb`) 用于接收 `with` 块中发生的异常信息。如果 `__exit__` 返回 `True`，表示它已经处理了异常，异常不会再次向上抛出。返回 `False` (或不返回任何值) 则会让异常继续传播。通常情况下，除非上下文管理器旨在封装异常处理逻辑，否则我们倾向于让异常传播。
#
# 3.  **善用 `contextlib` 模块:** 对于那些只需几行代码即可完成设置和清理的简单上下文管理器，`@contextmanager` 装饰器提供了一种更简洁、更 Pythonic 的方式来创建，避免了编写完整的类结构。它基于生成器实现，利用 `yield` 语句分隔 `__enter__` 和 `__exit__` 的逻辑。
#
# 4.  **嵌套 `with` 语句:** 当需要管理多个相关资源时，可以嵌套 `with` 语句。从 Python 3.1 开始，也可以使用单个 `with` 语句以逗号分隔多个上下文表达式，使代码更紧凑：
#     ```python
#     with open("file1.txt", "r") as f1, open("file2.txt", "w") as f2:
#         # 操作 f1 和 f2
#         pass
#     ```
#
# 5.  **异常处理的考量:** 在 `__exit__` 方法或 `@contextmanager` 装饰器的 `finally` 块中，确保资源释放逻辑本身是健壮的，即使在 `with` 块内部发生异常时也能正确执行。避免在释放资源时引入新的错误。
#
# 6.  **幂等性与可重入性:** 设计自定义上下文管理器时，考虑其操作是否幂等（多次调用 `__enter__` 或 `__exit__` 是否有副作用）以及是否可重入（同一个实例是否可以被多次用于 `with` 语句）。通常，资源管理对象不应设计为可重入，即一个连接对象在一个 `with` 块中被使用后，不应再被用于另一个 `with` 块。
```