- 类的静态方法(纯工具)，类成员方法(构造)
- MixIn：多继承，装饰器(直接拿到cls后想替换整个cls或者注入cls的成员都行)，元类(也是类似装饰器的一种，生命周期在模块被加载的时候，只是一般用来控制的是类对象的创建的一些逻辑)
- 单例实现：new思路，元类思路，装饰器思路(混入)，工厂方法这些太low
- 对象池：元类控制类对象创建的new，new里面是类对象的成员变量可以替换掉为用对象池的get/recyc，也可以加一些注入比如注册cls
- __del__：**自定义后GC会不管**

```
#! usr/bin/python3 
from enum import Enum, auto
from weakref import WeakValueDictionary, WeakSet
from functools import partial, total_ordering

def splite_out(func):
    def wrap(*args):
        name = func.__name__
        print(name.center(20, "-"))
        func()
        print()
    return wrap

class KEY(Enum):
    KEY1 = auto()
    KEY2 = auto()
    KEY3 = auto()

class A(object):
    def __init__(self, key=KEY.KEY1):
        self.key = key

    def __repr__(self):
        if self.key == KEY.KEY1:
            return "A:ins"
        elif self.key == KEY.KEY2:
            return f"this is a {self}"
        elif self.key == KEY.KEY3:
            return "A(KEY.KEY3)"

    def __str__(self):
        return f"self str: {id(self)}"

    def __format__(self, code):
        return "(fmt prefix; param:" + code + ")"

class LazzyConn(object):
    def __enter__(self):
        print("with enter...")

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("with exit: ", exc_type, exc_val, exc_tb)

@splite_out
def p1():
    a = A()
    print(a)
    print("{!r}".format(a))
    print("{!r}".format(A(KEY.KEY2)))

    # b = None  # 这是locals，如果没有这个会更新globals读取的也是globals
    # 提前定义了的话这里读取的就是locals而且是快照，更新的会进globals
    exec("b = {!r}".format(A(KEY.KEY3)), globals())
    print("{!r}".format(b))

    print(format(A(), "paramTest"))

@splite_out
def p2():
    b = LazzyConn()
    with b:
        print(b)

@splite_out
def p3():
    class A(object):
        def __init__(self):
            self.__private = 0

        def __private_method(self):
            print("aaaaaaaaaaaaa")

    class B(A):
        def __init__(self):
            self.__private = 1

        def __private_method(self):
            print("bbbbbbbbbbbb")

    a = A()
    b = B()
    try:
        print(a.__private, b.__private)
    except Exception as e:
        print("privateeeeee")
        b._B__private = 2
        print(a._A__private, b._B__private)

@splite_out
def p4():
    # 单例，new
    class A(object):
        def __init__(self):
            raise Exception("forbid init")

        def __new__(cls, *args, **kwargs):
            ins = super().__new__(cls, *args, **kwargs)
            ins.b = "bb"
            return ins

        @classmethod
        def create(cls):
            ins = cls.__new__(cls)
            ins.c = "cc"
            return ins

        def __repr__(self):
            return f"A(b: {self.b}, c: {self.c})"

    try:
        a = A()
    except Exception as e:
        print(e)
        a = A.create()
        print(a)

@splite_out
def p5():
    # 混入：继承，装饰器
    class MixOne:
        __slots__ = []
        def getAbility1(self):
            print("gain ability 1")

    class InitOnce:
        __slots__ = []
        _singleTon = None
        def __new__(cls, *args, **kwargs):
            if cls._singleTon is None:
                ins = super().__new__(cls, *args, **kwargs)
                cls._singleTon = ins
            return cls._singleTon

    class DuplicateAttr:
        __slots__ = []
        def __setattr__(self, key, value):
            if hasattr(self, key):
                print("duplicate init attr: ", key)
                return
            super(DuplicateAttr, self).__setattr__(key, value)

    def MixWrapper(cls):
        class Wrapper(DuplicateAttr, cls):
            pass
        return Wrapper

    @MixWrapper
    class A(MixOne, InitOnce, dict):
        pass

    a = A()
    b = A()
    print(id(a), id(b))
    print(a, b)
    print(a.getAbility1())
    a.x = 1
    a.x = 2
    print(a.x)

@splite_out
def p6():
    class A(object):
        def test(self):
            print("this is a test func")
    import operator
    a = A()
    operator.methodcaller("test")(a)
    exec("a.test()")

@splite_out
def p7():
    # 对象池
    class Pool(object):
        _Cache = {}
        @classmethod
        def try_regist(cls, cls_name):
            if cls_name not in cls._Cache:
                print("registtttttttttt", cls_name)
            obj_list = cls._Cache.setdefault(cls_name, [])
            return obj_list
        @classmethod
        def recycle(cls, obj):
            print("recycleeeeeeee", obj.__class__.__name__)
            obj_list = cls.try_regist(obj.__class__.__name__)
            obj_list.append(obj)
        @classmethod
        def get(cls, target_cls, create_func=None):
            print("gettttttttttttt", target_cls)
            obj_list = cls.try_regist(target_cls.__name__)
            if obj_list:
                obj = obj_list.pop()
            else:
                obj = create_func() if create_func else None
            return obj
        @classmethod
        def reset(cls):
            cls._Cache.clear()

    class PoolEntry(type):

        @staticmethod
        def _force_new(cls, *args, **kwargs):
            print("Meta Force Rpl:new: ")
            print("param: ", *args)
            print("kw: ", **kwargs)
            ins = Pool.get(cls, partial(cls.__ori_new__, cls, *args, **kwargs))
            return ins
        @staticmethod
        def _force_del(self, *args, **kwargs):
            print("Meta Force Rpl:del: ")
            Pool.recycle(self)

        def __new__(cls, cls_name, base, dct):
            ### 元类的运行时机是创建类对象的时候
            ### 所以这里都不需要实例化A()就会触发这里了
            ### 类似于装饰器
            ### 用元类限定类对象行为，即使继承重写也无法改变的逻辑
            print("Meta:PoolEntry:new: ")
            print(cls_name, base)
            print(dct)
            Pool.try_regist(cls_name)
            print("")
            dct["__ori_del__"] = dct["__del__"]
            dct["__del__"] = cls._force_del
            dct["__ori_new__"] = dct["__new__"]
            dct["__new__"] = cls._force_new
            return super(PoolEntry, cls).__new__(cls, cls_name, base, dct)

    class A(metaclass=PoolEntry):
        def __init__(self, *args):
            print("A:init:\nparam: ", args)
            self.a = ""
        def __new__(cls, *args, **kwargs):
            print("A:new: ")
            print(*args)
            print(**kwargs)
            return super(A, cls).__new__(cls)
        def __del__(self, *args):
            ### 定义了这个不参与GC！！！
            print("A:del: ", *args)

        def test_fun(self):
            pass

    a = A("xxxxxxx")
    # a = None
    l = []
    for _ in range(5):
        l.append(A(_))
    print("\n", Pool._Cache)
    l = None
    print("\n", Pool._Cache)
    Pool.reset()
    print("\n", Pool._Cache)

@splite_out
def p8():
    @total_ordering
    class A(object):
        def __init__(self, val):
            self.val = val

        def __eq__(self, other):
            return self.val == other.val

        def __gt__(self, other):
            return self.val > other.val

    a = A(100)
    b = A(200)
    print(a <= b)

@splite_out
def p9():
    ### mro: 拓扑排序+定义顺序
    # 在mro基础上，有super的就能继承下去，没有就断开
    class A:
        def test(self):
            print("A.test")
    class B:
        def test(self):
            super(B, self).test()
            print("B.test")
    class C(A, B):
        def test(self):
            super(C, self).test()
            print("C.test")
    class D(B, A):
        def test(self):
            super(D, self).test()
            print("D.test")
    C().test()
    print(C.__mro__)

    D().test()
    print(D.__mro__)

@splite_out
def p10():
    # 抽象类==接口
    from abc import ABCMeta, abstractmethod
    class A(metaclass=ABCMeta):
        @property
        @abstractmethod
        def func1(self):
            pass
        @property
        @abstractmethod
        def func2(self):
            print("----abs func2------")
            pass
    class B(A):
        pass
    class C(A):
        def func2(self):
            print("over abstract")
    class D(A):
        def func1(self):
            pass
        # @property ???
        def func2(self):
            print("-----getter------")
            pass
        # @A.func2.setter
        # def func2(self, val):
        #     print("setter")
        #     self.v = val
    try:
        a = A()
    except Exception as e:
        try:
            print(e)
            b = B()
        except Exception as e:
            try:
                print(e)
                c = C()
                print(c)
            except Exception as e:
                print(e)
                d = D()
                print(d.func2)
                d.func2 = "haha"

@splite_out
def p11():
    from collections import Iterable, Sequence
    import numbers
    import operator
    class A(Iterable):
        def __iter__(self):
            print("my iter")
    class B(Sequence):
        def __getitem__(self, idx):
            print("get item: ", idx)
            return 0
        def __len__(self):
            print("get len")
            return 0
    a = A()
    b = B()
    print(len(b), b[10])
    print(isinstance(a, Iterable))
    print(isinstance(a, Sequence))
    print(isinstance(a, Iterable))


if __name__ == "__main__":
    print("class...")
    p1()
    p2()
    p3()
    p4()
    p5()
    p6()
    p7()
    p8()
    p9()
    p10()
    p11()
```
