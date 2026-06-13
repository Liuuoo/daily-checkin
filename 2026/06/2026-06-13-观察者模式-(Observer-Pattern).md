# 观察者模式 (Observer Pattern)

## 适用场景 (Use Cases)
*   当一个对象（称为“主体”或“被观察者”）的状态发生改变时，所有依赖于该对象（称为“观察者”）的对象都会收到通知并自动更新。
*   适用于对象之间存在一对多依赖关系，并且主体对象的状态变化需要通知所有观察者的情况。
*   常用于实现事件处理机制、发布/订阅模型（Publish/Subscribe）、数据同步、UI更新等。
*   当你不希望将具体通知者类与具体被通知者类耦合时。

---

## Python 代码实现

```python
from abc import ABC, abstractmethod
from typing import List, Any

# --- 核心组件 ---

class Subject(ABC):
    """
    被观察者（主体）的接口。
    它维护一个观察者列表，并提供注册、注销和通知观察者的方法。
    """
    def __init__(self):
        # 存储所有观察者对象的列表
        self._observers: List['Observer'] = []
        print(f"Subject: Initialized.")

    def attach(self, observer: 'Observer') -> None:
        """
        注册一个观察者。
        如果观察者已存在，则不重复添加。
        """
        if observer not in self._observers:
            self._observers.append(observer)
            print(f"Subject: Observer {observer.__class__.__name__} attached.")
        else:
            print(f"Subject: Observer {