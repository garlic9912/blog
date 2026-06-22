`abc` 用于定义**抽象基类**。它的核心目的是：**规定子类必须实现哪些方法**，如果子类不实现，就无法实例化。

```python
from abc import ABC, abstractmethod

# 抽象类：描述"动物"应该具备什么能力，但不具体实现
class Animal(ABC):
    @abstractmethod
    def make_sound(self) -> str:
        """子类必须实现：发出声音"""
        pass
    
    @abstractmethod
    def move(self) -> str:
        """子类必须实现：移动方式"""
        pass
    
    def sleep(self) -> str:
        """普通方法：子类可以直接继承使用，不必重写"""
        return "Zzz..."

# 正确的子类：实现了所有抽象方法
class Dog(Animal):
    def make_sound(self) -> str:
        return "汪汪"
    
    def move(self) -> str:
        return "用腿跑"

# 错误的子类：没有实现 move 方法
class Fish(Animal):
    def make_sound(self) -> str:
        return "咕噜咕噜"
    # 缺少 move() 方法
```
- 子类必须实现基类全部的未定义方法
- 子类的实现必须确保参数, 返回值与基类定义完全一致