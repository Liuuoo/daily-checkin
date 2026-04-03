# 设计模式实战：装饰器模式 (Decorator Pattern)

## 1. 模式名称与适用场景
```python
"""
模式名称：装饰器模式 (Decorator Pattern) - 结构型设计模式

核心思想：
允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是采用继承的方式来实现在运行时动态地组合不同的行为。

适用场景：
1. 需要动态地为对象添加职责（而非在编译时静态确定）。
2. 替代多层继承产生的“类爆炸”问题（例如：Button -> RedButton -> RoundRedButton -> ClickableRedButton...）。
3. 当需要扩展功能但无法或不宜修改原有类的源代码时（符合开闭原则）。
4. Python 原生装饰器语法 (@) 的底层原理即基于此模式的变体。

关键特性：
- 组合优于继承 (Composition over Inheritance)。
- 透明性：客户端无法区分是操作原始对象还是装饰后的对象。
"""
from abc import ABC, abstractmethod
from typing import List, Callable


# 2. 完整 Python 代码实现

class Coffee(ABC):
    """
    抽象组件 (Component)
    定义所有具体组件和装饰器的公共接口。
    """
    @abstractmethod
    def cost(self) -> float:
        pass

    @abstractmethod
    def description(self) -> str:
        pass


class SimpleCoffee(Coffee):
    """
    具体组件 (Concrete Component)
    代表最基本的实体。
    """
    def cost(self) -> float:
        return 10.0

    def description(self) -> str:
        return "基础咖啡"


class CoffeeDecorator(Coffee):
    """
    抽象装饰器 (Decorator)
    持有一个指向组件的引用，并委托大部分请求给该组件。
    """
    def __init__(self, coffee: Coffee):
        self._coffee = coffee

    def cost(self) -> float:
        return self._coffee.cost()

    def description(self) -> str:
        return self._coffee.description()


class MilkDecorator(CoffeeDecorator):
    """
    具体装饰器 A (Concrete Decorator A)
    添加牛奶的功能。
    """
    def cost(self) -> float:
        # 在父类基础上增加成本
        return super().cost() + 5.0

    def description(self) -> str:
        # 在描述中增加特征
        return f"{super().description()}, 加奶"


class SugarDecorator(CoffeeDecorator):
    """
    具体装饰器 B (Concrete Decorator B)
    添加糖的功能。
    """
    def cost(self) -> float:
        return super().cost() + 2.0

    def description(self) -> str:
        return f"{super().description()}, 加糖"


class CreamDecorator(CoffeeDecorator):
    """
    具体装饰器 C
    添加奶油的功能。
    """
    def cost(self) -> float:
        return super().cost() + 8.0

    def description(self) -> str:
        return f"{super().description()}, 加奶油"


# 3. 使用示例

def main():
    print("--- 订单系统演示 ---")
    
    # 基础对象
    my_coffee: Coffee = SimpleCoffee()
    print(f"{my_coffee.description()} - ¥{my_coffee.cost():.2f}")

    # 动态叠加装饰
    # 逻辑链：基础 -> 加糖 -> 加奶 -> 加奶油
    
    decorated_coffee: Coffee = my_coffee
    decorated_coffee = SugarDecorator(decorated_coffee)
    decorated_coffee = MilkDecorator(decorated_coffee)
    decorated_coffee = CreamDecorator(decorated_coffee)

    print(f"{decorated_coffee.description()} - ¥{decorated_coffee.cost():.2f}")
    
    # 另一种组合
    print("\n--- 另一杯咖啡 ---")
    another_coffee = MilkDecorator(SimpleCoffee())
    another_coffee = SugarDecorator(another_coffee)
    print(f"{another_coffee.description()} - ¥{another_coffee.cost():.2f}")


if __name__ == "__main__":
    main()
```

## 4. 优缺点深度分析

### ✅ 优点 (Pros)

1.  **符合开闭原则 (Open/Closed Principle)**:
    *   对扩展开发开放，对修改封闭。新增一种口味（如“加盐”）只需新建一个 `SaltDecorator` 类，无需修改原有的 `SimpleCoffee` 或其他装饰器代码。
2.  **避免类爆炸 (Avoids Class Explosion)**:
    *   如果使用多重继承来实现不同属性组合（如颜色、大小、样式），类数量会呈指数级增长。装饰器模式通过组合将功能拆分到独立的类中，大大减少了类的数量。
3.  **运行时灵活性**:
    *   可以在程序运行期间动态组装行为。你可以像搭积木一样，根据用户的选择实时构建复杂的对象结构。
4.  **职责单一 (Single Responsibility)**:
    *   每个装饰器只负责一项具体的增强功能，降低了单个类的复杂度。

### ❌ 缺点 (Cons)

1.  **对象复杂性增加**:
    *   系统中会产生大量的小对象（小对象很多且互相嵌套）。对于初学者来说，理解对象图（Object Graph）和调用栈可能比直接看继承树要困难。
2.  **调试困难**:
    *   由于层层包裹，异常追踪（Stack Trace）可能会变得冗长。如果某层装饰器抛出了异常，很难第一时间定位是哪一层逻辑出了问题。
3.  **配置繁琐**:
    *   相比于直接实例化子类，这里需要手动编写大量的包装代码（虽然可以使用工厂模式来优化这部分初始化过程）。
4.  **性能轻微损耗**:
    *   每一层调用都需要经过一次方法转发（Dispatch），在极高性能要求的场景下，深层次的装饰链可能会引入微小的 CPU 开销。但在一般业务逻辑中可忽略不计。

### 💡 工程师建议 (Engineering Note)
在实际生产环境中，如果你发现 `if-else` 或 `switch-case` 代码块变得越来越臃肿，或者为了适应微小变化就要复制粘贴大量相似类，这就是考虑使用装饰器模式的最佳信号。此外，Python 原生的 `@functools.wraps` 语法糖本质上就是函数版的装饰器模式，理解本结构模式有助于你更深刻地掌握 Python 的高级特性。