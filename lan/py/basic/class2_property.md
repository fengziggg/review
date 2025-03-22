- 属性property的@用法和property(get, set, del)，property的继承注意
- describe，用装饰器(拦截class/function)，元类进行混入
- 懒加载：用装饰器可以实现，用property本身也可以，还可以两者结合用装饰器存函数，用property触发运算
- __getattr__/__setattr__使用注意，类内就开始生效，setter也会对getter生效

```
#! usr/bin/python3 
def splite_out(func):
    def wrap(*args):
        name = func.__name__
        print(name.center(20, "-"))
        func()
        print()
    return wrap

class A(object):
    def __init__(self):
        self._name = ""
    # property的set/del依赖get
    # 但不是describe有这样的规定，descirbe不依赖
    @property
    def name(self):
        print("get name")
        return self._name
    @name.setter
    def name(self, val):
        print("set name")
        self._name = val
    @name.deleter
    def name(self):
        print("forbid del")
        # raise RuntimeError("no del")

    def get_other(self):
        print("get other")
        return self._name
    def set_other(self, val):
        print("set other")
        self._name = val
    def del_other(self):
        print("forbid del other")
        # raise RuntimeError("no del")
    other = property(get_other, set_other, del_other)

@splite_out
def p1():
    a = A()
    print(a.name)
    a.name = "haha"
    del a.name

    print(a.other)
    a.other = "hehe"
    del a.other

@splite_out
def p2():
    # super(mro起点, 绑定self/未绑定方法).xxx
    # propter的继承比较麻烦要么全覆盖，要么用父类名字单独写setter/delter

    class B(A):
        @property
        def name(self):
            print("get name1")
            return super(B, self).name

    class C(A):
        @property
        def name(self):
            print("get name2")
            return super(C, self).name
        @name.setter
        def name(self, v):
            print("set name2")
            print(super(), super(C, self), super(C, C))  # <C object>> <super: <class 'C'>, <C object>>
            # super(C, self).name.__set__(v)  # 用self去调用name经过了set返回对象是str报错
            # super().name.__set__(self, v)
            super(C, C).name.__set__(self, v)
    class D(A):
        def name(self):
            print("get name3")
            return super(D, self).name
        @A.name.setter
        def name(self, v):
            print("set name 3")
            self._name = v

    try:
        b = B()
        b.name = 1
    except Exception as e:
        print(e)
        c = C()
        c.name = 2
        d = D()
        d.name = 3

@splite_out
def p3():
    # getattr()会触发__get__!!
    # describer的约定是作为静态成员变量才有效
    # 正常是每个field都有三个方法，所有就是field*3的成员函数对象
    # 这样就吧三个操作集成到一个对象里面在放到类里面做成员
    class Descriptor(object):
        def __init__(self, name):
            self._name = name
        def __get__(self, instance, owner):
            print("desc.get: ", instance, owner)
            # return getattr(instance, self._name)
            return instance.__dict__[self._name]
        def __set__(self, instance, value):
            print("desc set: ", instance, value)
            instance.__dict__[self._name] = value
        def __delete__(self, instance, *args):
            print("desc del: ", instance)
            # del instance.__dict__[self._name]
    def DescWrapper(args):
        def Wrapper(cls):
            for k in args:
                desc = Descriptor(k)
                setattr(cls, k, desc)
            return cls
        return Wrapper

    @DescWrapper(["mixField", "mixField2"])
    class A(object):
        field1 = Descriptor("field1")
        field2 = Descriptor("field2")
        def __init__(self):
            self.field1 = 1
            self.field2 = 2
    a = A()
    a.field1 = 1
    print(a.field2)
    a.mixField = 3
    # print(a.mixField2)

@splite_out
def p4():
    # 装饰器是AOP，这里一个装饰器的describe
    # 装饰器懒加载(初始化装饰器的时候就替换好了)， describe懒加载(访问的时候替换)
    class Descriptor(object):
        def __init__(self, func):
            self.func = func
        def __get__(self, instance, owner):
            if instance is None:
                return self
            print("desc delay get and rpl:: ")
            # return getattr(instance, self._name)
            ret = self.func(instance)
            instance.__dict__[self.func.__name__] = ret
            return ret

    class A(object):
        def __init__(self):
            self.field1 = 1
            self.field2 = 2
        @Descriptor
        def calcu(self):
            return 100
    a = A()
    print(a.calcu)

@splite_out
def p5():
    def Type1(cls):
        def __get__(self, instance, owner):
            print("type11111 checkk")
            if instance is None:
                return self
            return instance.__dict__[self._name]
        def __set__(self, instance, value):
            print("desc set111111111: ", instance, value)
            instance.__dict__[self._name] = value
        cls.__get__ = __get__
        cls.__set__ = __set__
        return cls
    def Type2(cls):
        def __get__(self, instance, owner):
            print("type2222 checkk")
            if instance is None:
                return self
            return instance.__dict__[self._name]
        cls.__get__ = __get__
        return cls
    class Descriptor(object):
        def __init__(self, name):
            self._name = name
    @Type1
    class Desc1(Descriptor):
        pass
    @Type2
    class Desc2(Descriptor):
        pass
    class A(object):
        field1 = Desc1("field1")
        field2 = Desc2("field2")
        # init初始化的时候如果没有set会被这个赋值操作覆盖掉Desc
        def __init__(self):
            print(A.field1)
            self.field1 = 1
            self.field2 = 2
    a = A()
    b = A()
    b.field1 = 11
    print(a.field1)
    print(b.field1)

@splite_out
def p6():
    # __getattr__/__setattr__难用
    # 从类定义初始化init里面的类内引用都会被触发!!!
    # 在setattr里面也会触发getattr，要setattr区别对待防止递归
    class A(object):
        def __init__(self):
            self.x = 0
            self.y = 1
        def test(self):
            print("tttttttttttt")
        def __private(self):
            print("pppppppppp")
    class B(object):
        def __init__(self, proxy):
            self._proxy = proxy
            self._x = -100
        @property
        def x(self):
            return self._x
        def testB(self):
            print("test bbbbbbbbbb")
        def __getattr__(self, name):
            print("B.getattr: ", name)
            return getattr(self._proxy, name)
        def __setattr__(self, key, value):
            print("B.setattr: ", key, value)
            if not key.startswith("_"):
                return self._proxy.__setattr__(key, value)
            else:
                super(B, self).__setattr__(key, value)

    b = B(A())
    b.x
    b.y
    b.testB()
    b.test()
    try:
        b.__private()
    except:
        pass


if __name__ == "__main__":
    print("class2...")
    p1()
    p2()

    p3()
    p4()
    p5()
    p6()
```
