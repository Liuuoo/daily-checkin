# 策略模式 (Strategy Pattern)

## 1. 模式名称和适用场景

```python
"""
模式名称：策略模式 (Strategy Pattern)
分类：行为型设计模式
适用场景：
1. 一个类定义了多种行为，这些行为在该类的运行时动态变化。
2. 希望避免大量的 if-else 或 switch-case 语句来判断具体执行哪个算法。
3. 多个类之间存在相似的行为逻辑，但实现细节不同（如不同的支付网关、不同的排序算法）。
4. 系统需要在运行时动态切换算法，而不影响客户端代码。
核心思想：定义一系列可互换的算法，并将每个算法封装在独立的类中，使它们可以相互替换。
"""
```

## 2. 完整的 Python 代码实现

本实现展示了如何利用 `abc.ABC` 构建抽象基类，并通过依赖注入将具体策略传递给上下文（Context）。为了体现 Python 的动态特性，我还演示了如何直接使用函数作为策略。

```python
from abc import ABC, abstractmethod
from typing import Protocol, Callable, TypeVar, Generic

T = TypeVar('T')

# --- 1. 抽象策略接口 (The Interface) ---
class PaymentStrategy(Protocol):
    """
    定义所有支付策略必须遵循的契约。
    使用 Protocol 体现了 Python 的鸭子类型，不强制继承也能兼容。
    """
    def process_payment(self, amount: float) -> bool:
        pass

# --- 2. 具体策略实现 (Concrete Strategies) ---

class CreditCardPayment(PaymentStrategy):
    """信用卡支付策略"""
    
    def __init__(self, card_number: str):
        self.card_number = card_number
    
    def process_payment(self, amount: float) -> bool:
        print(f"正在处理信用卡支付... (卡号尾号：{self.card_number[-4:]})")
        return True  # 模拟成功

class PayPalPayment(PaymentStrategy):
    """PayPal 支付策略"""
    
    def __init__(self, email: str):
        self.email = email
    
    def process_payment(self, amount: float) -> bool:
        print(f"正在重定向到 PayPal 支付... (账户：{self.email})")
        return True  # 模拟成功

class CryptoPayment(PaymentStrategy):
    """加密货币支付策略 (演示多参数需求)"""
    
    def __init__(self, wallet_address: str):
        self.wallet_address = wallet_address
    
    def process_payment(self, amount: float) -> bool:
        print(f"生成 QR 码用于 {self.wallet_address} 支付 {amount} USDT")
        # 模拟区块链确认延迟
        return True

# --- 3. 上下文环境 (The Context) ---

class OrderContext:
    """
    订单上下文，持有对当前策略的引用。
    注意：这里没有硬编码具体的支付逻辑，而是委托给 Strategy。
    """
    
    def __init__(self, strategy: PaymentStrategy):
        # 依赖注入：通过构造函数传入策略
        self._strategy = strategy
    
    def set_strategy(self, strategy: PaymentStrategy):
        """允许在运行时动态切换策略"""
        self._strategy = strategy
    
    def checkout(self, amount: float) -> None:
        """客户端无需关心具体支付实现"""
        print(f"\n--- 开始结账金额：${amount} ---")
        success = self._strategy.process_payment(amount)
        if success:
            print("支付完成！")
        else:
            print("支付失败。")

# --- 4. 进阶技巧：函数式策略 (Function as Strategy) ---
# Python 的特性使得函数也可以直接作为策略，减少样板代码

def instant_checkout_callback(amount: float) -> bool:
    """简单的回调函数作为策略"""
    print("调用即时到账接口...")
    return True

if __name__ == "__main__":
    # --- 5. 使用示例 ---
    
    # 场景 1：默认创建对象并使用策略
    order = OrderContext(CreditCardPayment("1234-5678-9012"))
    order.checkout(100.00)
    
    # 场景 2：运行时切换策略
    print("\n\n用户决定更换支付方式为 PayPal...")
    order.set_strategy(PayPalPayment("user@example.com"))
    order.checkout(500.00)
    
    # 场景 3：混合使用函数作为策略（利用 Python 灵活性）
    # 在实际业务中，这通常对应简单的第三方 API 回调
    print("\n\n用户选择极速通道（基于函数）...")
    order.set_strategy(instant_checkout_callback) 
    order.checkout(10.00)
```

## 3. 优缺点分析

### ✅ 优点 (Pros)

1.  **符合开闭原则 (Open/Closed Principle)**：
    *   当需要新增一种支付方式时，只需增加一个新的类实现 `PaymentStrategy` 接口，无需修改现有的 `OrderContext` 或其他策略类的代码。这极大地降低了回归测试的风险。
2.  **消除复杂的条件判断**：
    *   避免了在上下文中编写冗长的 `if type == A: ... elif type == B: ...` 结构。代码逻辑更加清晰，关注点分离（Separation of Concerns）。
3.  **运行时动态性**：
    *   结合 `set_strategy` 方法，程序可以在运行过程中根据用户选择或配置动态切换算法行为，提供了极高的灵活性。
4.  **复用算法代码**：
    *   共享的策略实现可以被多个上下文使用，减少了代码重复。

### ❌ 缺点 (Cons)

1.  **客户端知晓策略数量**：
    *   客户端必须知道有哪些具体的策略可用，并且负责实例化它们并传递给上下文。如果策略非常多，管理起来可能变得繁琐。
2.  **上下文与策略的耦合**：
    *   虽然解耦了算法本身，但上下文类仍然依赖于策略接口。如果接口频繁变动，所有实现类和上下文都需要更新。
3.  **性能开销**：
    *   相比于直接在类内部实现逻辑，引入额外的对象实例化和间接调用（Indirection）会带来微小的性能损耗（通常在现代语言中可忽略不计，但在高频循环计算场景需考量）。

### 🛠️ 最佳实践建议

*   **何时使用**：当同一个功能有多种实现，且这些实现之间差异较大，或者这些实现经常变更时。例如：排序算法（快排/归并）、加密方式（AES/RSA）、通知渠道（邮件/SMS/WhatsApp）。
*   **何时避免**：如果只有 2-3 种固定情况且几乎不会变，直接使用 `if-else` 可能更简单直接。过度设计会增加系统的复杂度。
*   **Python 特性结合**：在 Python 中，不需要像 Java 那样严格定义接口，利用 `Protocol` 或简单的鸭子类型即可。甚至可以传递 lambda 表达式或普通函数，使策略的使用更加轻量级。