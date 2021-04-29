### 构造和初始化
- __new__  创建类并返回类的实例，__init__奖传入的参数来初始化实例
- __init__ 对象初始化时调用，在调用__init__方法时会先调用__new__方法, __init__ `除了返回None不能返回任何值`
-__del__   对象的生命周期结束时，会调用此方法，__del__定义的时当一个对象进行垃圾回收时的行为<br/>

**x.__del__()不是对于del x 的实现，而是执行del x 时会调用 x.__del__()**

对于这句话的解释如下：
```markdown
foo = Foo()
foo.__del__()
print foo
del foo
print foo  # NameError, foo is not defined
```

虽然调用了__del__()方法,但是foo对象并没有被删除。调用del foo后，foo对象被删除

### 属性访问
-__getattribute__  定义了访问一个类的某个属性时的行为，如果该属性不存在，则会调用__getattr__方法
-__getattr__  定义了试图访问一个**不存在**的属性时的行为
-__setattr__ 定义了对属性进行赋值和修改时的行为（无论该属性是否存在，都可以对其进行赋值）。但在实现时需要避免无限递归的问题（即实现此方法时不能使用self.attr =  xxx）
-__delattr__  定义删除属性时的行为，也要避免无限递归的问题，（实现此方法时不能使用del self.attr ）

__setattr__ / __delattr__ 的无限递归
```markdown
def __setattr__(self, name, value):
    self.name = value
# 属性赋值时又会调用自身
```
正确写法
```markdown
def __setattr__(self, name, value):
    self.__dict__[name] = value
```

以上四种魔术方法调用情况

```markdown
class Access(object):

    def __getattr__(self, name):
        print '__getattr__'
        return super(Access, self).__getattr__(name)

    def __setattr__(self, name, value):
        print '__setattr__'
        return super(Access, self).__setattr__(name, value)

    def __delattr__(self, name):
        print '__delattr__'
        return super(Access, self).__delattr__(name)

    def __getattribute__(self, name):
        print '__getattribute__'
        return super(Access, self).__getattribute__(name)

access = Access()
access.attr1 = True  # __setattr__调用
access.attr1  # 属性存在,只有__getattribute__调用
try:
    access.attr2  # 属性不存在, 先调用__getattribute__, 后调用__getattr__
except AttributeError:
    pass
del access.attr1  # __delattr__调用

```
### 描述器对象

属性访问顺序
```python
1. 实例属性
2. 类属性
3. 父类属性
如果找到普通值直接返回，如果是描述器，则调用其__get__方法
```

距离既可以使用单位meter也可以使用foot表示
```markdown
class Meter(object):
    '''Descriptor for a meter.'''
    def __init__(self, value=0.0):
        self.value = float(value)
    def __get__(self, instance, owner):
        return self.value
    def __set__(self, instance, value):
        self.value = float(value)

class Foot(object):
    '''Descriptor for a foot.'''
    def __get__(self, instance, owner):
        return instance.meter * 3.2808
    def __set__(self, instance, value):
        instance.meter = float(value) / 3.2808

class Distance(object):
    meter = Meter()
    foot = Foot()

d = Distance()
print d.meter, d.foot  # 0.0, 0.0
d.meter = 1
print d.meter, d.foot  # 1.0 3.2808
d.meter = 2
print d.meter, d.foot  # 2.0 6.5616
```

虽然d.meter是一个对象，但是因为在Meter()类中重写了__get__方法，所以在使用d.meter是调用了Meter()  的__get__方法，返回了数字
描述器的特点：

```markdown
1. 描述器对象不能单独存在
2. 描述器对象可以访问其拥有者实例的属性，如在Foot()描述器中，__get__方法调用了instance的meter属性
3. 一个类要成为描述器，必须实现__get__, __set__， __deleete__中的至少一个方法
面向对象编程时，如果一个类的属性具有相互依赖的关系，可以使用描述器来编写代码可以巧妙地组织逻辑
```

### 构造自定义容器

不可变类型需要实现__len__和__getitem__，可变类型需要在不可变类型的基础上实现__delitem__和__setitem__，如果要自定义容器可迭代，还需要实现__iter__方法
被调用情况：
x = X()
- __len__ :  len(x) 
- __getitem__: x[A]
- __setitem__: x[B] = C
- __delitem__: del X[D]
- __iter__:  for n in x 或 iter(n) 时调用
- __reversed__: reversed(x) 时调用
- __contains__: n in x 或 item not in x 时调用
- __missing__: 如 a = {"b": "c"}, 调用a[notexist]时会调用a.__missing__['notexist']


