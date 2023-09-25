---
title: Python 实现自动工厂模式
date: 2017-01-16 00:00:00+0000
image: img/python.jpg
categories:
    - Python
tags:
    - Python
---



**情景演示**：
```python
class Someone(object):
    def __init__(self):
        self.speak = 'haha'

    def __getattr__(self, name):
        if not name[0].isupper():
            return object.__getattribute__(self, name)

        def __getattr(cls, name):
            return object.__getattribute__(self, name)
        try:
            cls = eval(name)
        except:
            print('No %s' % name)
            raise NotImplementedError
        cls.__getattr__ = __getattr
        instance = cls()
        setattr(self, name, instance)
        return instance

class Tom(object):
    def run(self):
        return 'running Tom'

class Jerry(object):
    def run(self):
        return 'running Jerry'

men = Someone()
print(men.speak)
print(men.Tom.run())
print(men.Jerry.run())

# haha
# running Tom
# running Jerry
```

子类 TOM 和 Jerry 并没有显式继承 Speaker，而且实例化的是父类 Speaker，调用子类名称空间的时候自动实例化。

实际应用时，上面的类可能不在同一个文件中，这时候可以通过**动态加载类或包**来实现

**someone.py**
```python
# -*- coding: utf-8 -*-
import importlib

class Action(object):
    _pkg_base = 'some/path/'

    def __getattr__(self, name):
        pkg = _pkg_base + name
        try:
            module = importlib.import_module(pkg)
            Action = module.Action
        except ImportError:
            Action = object

        def __getattr(cls, name):
            return object.__getattribute__(cls._myself, name)

        Action.__getattr__ = __getattr
        instance = Action()
        instance._myself = self
        setattr(self, name, instance)
        return instance
```

**tom.py**
```python
class Action(object):
    def run(self):
        return 'running Tom'
```

**jerry.py**
```python
class Action(object):
    def run(self):
        return 'running Jerry'
```