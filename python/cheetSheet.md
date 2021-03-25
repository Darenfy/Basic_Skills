### 确定系统版本
```Python
import sys
if sys.version_info.major < 3:
    print("Version 3.X)
else sys.version_info.major > 3:
    print("Future")
else:
    print("Version 3.X")
```

### 添加文档
```python
def f():
    """return 'Hello World'"""
    return "Hello World!"

print(f.__doc__)
```

### 任意参数
```py
def f(*args, **kargs):
    print("args ", args)
    print("kargs ", kargs)
    print("FP: {} & Scripts: {}".format(kargs.get("fp"), "/".join(args)))

f("Python", "Javascript", ms = "C++", fp = "Haskell")
```

### 修饰器
```py
def log(f)
    def wrapper():
        print("Hey log~")
        f()
        print("Bye logs")
    return wrapper

@log
def fa():
    print("This is fa!")

def fb():
    print("This is fb!")
fb = log(fb)
```

### 类模型
```py
class Animal:
    """This is an Animal"""
    def __init__(self):
        print("Calling __init__ when instantiantion!")
    
    def fly(self):
        return True

class Dog(Animal):
    """This is a Dog"""
    def bark(self):
        print("Woof!")

a = Animal()
print(isinstance(a, Animal))
```
