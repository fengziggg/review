元类的的一般用途：控制类对象的生命周期，或者说里面的内容，就是定义类的定义(脚本上的定义还能被中间介入修改)
- __prepare__, __new__, __init__, __signature__, __call__(**这个是影响实例对象的，不是影响类对象**)
- 属性注入，签名注入，方法检查，nametuple(注入属性思路)
- types动态创建新类：nametuple实现(完全新类思路)
- exec思路

装饰器：
- 场景：单例对象池，logged注入，
- 动态属性注入或者nametuple实现其实就是利用系统触发时机动态添加property


```
#! usr/bin/python3 
import time
from functools import wraps, partial
from types import MethodType
from inspect import signature, Signature, Parameter
from weakref import WeakValueDictionary
from collections import OrderedDict
import operator

def splite_out(func):
    def wrap(*args):
        name = func.__name__
        print(name.center(20, "-"))
        func()
        print()
    return wrap

def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("deco")
        ret = func(*args, **kwargs)
        print("finish deco")
        return ret
    return wrapper

def wrapper2(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("wrapper2")
        ret = func(*args, **kwargs)
        print("finish wrapper2")
        return ret
    return wrapper

@timethis
def p1_test(param1, kwparam2):
    print("test p1")

def p1():
    p1_test(1, 2)

@splite_out
def p2():
    print(p1_test.__name__)
    print(p1_test.__wrapped__)
    from inspect import signature
    print(signature(p1_test))

@wrapper2
@timethis
def p3_test():
    print("test p3")

@staticmethod
def p3_test2():
    print("p3 test2")

@splite_out
def p3():
    print(p3_test.__wrapped__)
    try:
        print(p3_test2.__wrapped__)
    except Exception as e:
        print(e)

def p4_wrapper(deco_param):
    print("exec p4 wrapper, param", deco_param)
    def p4_wraper2(func):
        @wraps(func)
        def p4_inner(*args, **kwargs):
            print("p4 inner")
            ret = func(*args, **kwargs)
            print("finish p4")
            return ret
        return p4_inner
    return p4_wraper2

@p4_wrapper("hellow")
def p4_test():
    print("p4 test")

@splite_out
def p4():
    p4_test()

@splite_out
def p5():
    # 装饰器技巧
    # 技巧1：在定义时机完成抓取逻辑，
    # 技巧2: 平替fake/抓取注入行为(不一定只往被装饰对象注入，也可以将被装饰对象用于注入其他人)
    # 技巧3：参数循环定义装饰器
    def deco_inject(obj, target=None):
        # 这个装饰器不是替换类型是注入类型
        if target is None:
            print("inject step1")
            return partial(deco_inject, obj)  # injectee被缓存延迟传参
        else:
            print(f"inject setp2: {obj}<-{target}")
            setattr(obj, target.__name__, target)
    def deco1(param1, param2=1):
        print("deco call: ", param1, param2)
        def deco2(func):
            def wrapper(*args, **kw):
                func(*args, **kw)

            @deco_inject(wrapper)
            def inject_hook1():
                print("this is injected hook1")
            def inject_hook2():
                print("this is inject hook2")
            wrapper.__dict__[inject_hook2.__name__] = inject_hook2

            return wrapper
        return deco2

    @deco1("param1", "param2")
    def test():
        print("test func")
    test()
    test.inject_hook1()
    test.inject_hook2()

@splite_out
def p6():
    def deco1(func=None, param1=None):
        # 循环定义装饰器(兼容无参情况)
        if func is None:
            print("step1: appoint key param")
            return partial(deco1, param1=param1)
        else:
            print("setp2: return functor")
            func.__dict__[str(param1)] = param1
            def wrapper(*args, **kw):
                if param1:
                    print("attach param: ", param1)
                return func(*args, **kw)
            return wrapper
    @deco1(param1="aaa")
    def test():
        print("test func")
    @deco1
    def test2():
        print("test func2")
    test()
    test2()

@splite_out
def p7():
    # 检查前面
    # signature模块使用，bind参数
    from inspect import signature
    def deco1(*type_args, **kw_type):
        def deco2(func):
            sig = signature(func)
            target_argu = sig.bind_partial(*type_args, **kw_type).arguments
            def wrapper(*args, **kw):
                sig_runtime = signature(func)
                real_argu = sig_runtime.bind(*args, **kw).arguments
                for k, v in real_argu.items():
                    if k in target_argu:
                        if not isinstance(v, target_argu[k]):
                            print("type forbidddd", k, v, target_argu[k])
                return func(*args, **kw)
            return wrapper
        return deco2

    @deco1(str, int, z=float)
    def test(x, y, z=0):
        print(x, y, z)
    test(1, 2, 3)
    test(1, z=2.0, y=None)

class A:
    def deco1(self, func):
        print("deco1")
        @wraps(func)
        def deco1_inner(*args, **kw):
            print("deco1_inner")
            ret = func(*args, **kw)
            print("finish deco1_inner")
            return ret
        return deco1_inner
    @classmethod
    def deco2(cls, func):
        print("deco2")
        @wraps(func)
        def deco2_inner(*args, **kw):
            print("doco2_inner")
            ret = func(*args, **kw)
            print("deco2 finish")
            return ret
        return deco2_inner
a = A()
@A.deco2
@a.deco1
def test_p8():
    print("test p8")
class B:
    name = property()
    @name.getter
    def name(self):
        print("get name")
        return self._name
    @name.setter
    def name(self, sName):
        print("set name")
        if not isinstance(sName, str):
            raise TypeError("not str")
        self._name = sName
class C(A):
    @A.deco2
    def test(self):
        pass

@splite_out
def p8():
    # 类装饰器（类对象，实例装饰器）
    test_p8()
    b = B()
    b.name = "haha"
    print(b.name)
    c = C()
    c.test()

@splite_out
def p9():
    # @classmethod/staticmethod/property是创建描述器对象
    # 所以嵌套装饰器有顺序要求不能先变成描述器
    class ClsDeco(object):
        def __init__(self, func):
            self.func = func
        def __call__(self, *args, **kw):
            print("step: deco call", *args)
            self.func(*args, **kw)
        def __get__(self, instance, owner):
            print("step: getter call", self.func, instance, owner)
            if instance is None:
                print("class method getter", self.func)
                return self.func
            print("instance method getter")
            return MethodType(self, self.func)
    @ClsDeco
    def test():
        print("test class deco")
    class A:
        @ClsDeco
        def test2(self):
            print("test class deco22222")
        @classmethod
        @ClsDeco
        def test3(cls):
            print("test class deco33333333")
    test()
    A().test2()
    A.test3()


@splite_out
def p10():
    # 注入参数前面，inspect用
    import inspect
    def test(x, y, z=100):
        print("test class deco")
    sig = signature(test)
    print(sig.parameters)
    param = inspect.Parameter("param", inspect.Parameter.KEYWORD_ONLY, default=False)
    test.__signature__ = sig.replace(parameters=[param])
    print(signature(test))

@splite_out
def p11():
    # __new/init__可以组织类对象
    # __call__是直接组织用户编写的时候的Class()，是直接组织实例对象的
    class Meta(type):
        def __init__(self, *args, **kw):
            self._cache = WeakValueDictionary()
            super(Meta, self).__init__(*args, **kw)

        def __call__(self, *args, **kwargs):
            if args not in self._cache:
                obj = super(Meta, self).__call__()
                self._cache[args] = obj
            return self._cache[args]
    class A(metaclass=Meta):
        pass
    a = A("haha")
    b = A("hehe")
    c = A("haha")
    print(a is b)
    print(a is c)


@splite_out
def p12():
    class MyMeta(type):
        # prepare更靠前，用在类定义的时候返回一个dct
        # 控制属性顺序
        @classmethod
        def __prepare__(mcs, name, bases, *args, order=False):
            print(f"MyMeta:prepare: {name}, {args}, order={order}")
            ret = super(MyMeta, mcs).__prepare__(name, bases, *args)
            if order:
                ret = OrderedDict()
            print(type(ret), ret)
            return ret
        def __new__(cls, name, base, dct, *args, order=False):
            print("MyMeta:new", name)
            for k, v in dct.items():
                print(k, v)
            dct["_order"] = tuple(dct.keys())
            return super(MyMeta, cls).__new__(cls, name, base, dct, *args)
    class A(metaclass=MyMeta, order=False):
        pass
    class OrderA(metaclass=MyMeta, order=True):
        pass
    a = A()
    b = OrderA()
    print(a._order)

@splite_out
def p13():
    # 元编程
    # 动态添加签名
    # 动态加属性
    # 检查方法重载，
    class Meta(type):
        @staticmethod
        def _make_sig(_sign):
            sig = []
            for _f in _sign:
                sig.append(Parameter(_f, Parameter.POSITIONAL_OR_KEYWORD))
            return Signature(sig)
        @classmethod
        def make_sig(cls, dct):
            if "_sign" not in dct:
                dct["_sign"] = []
            _sign = dct["_sign"]
            dct["__signature__"] = cls._make_sig(_sign)

        @classmethod
        def make_field(cls, dct):
            if "_field" not in dct:
                dct["_field"] = []
            _field = dct["_field"]
            for idx, name in enumerate(_field):
                pp = property(operator.itemgetter(idx))
                dct[name] = pp

        @classmethod
        def check_func_sig(cls, ins, dct):
            super_obj = super(ins, ins)
            for k, obj in dct.items():
                if not k.startswith("_") and callable(obj) and hasattr(super_obj, k):
                    super_func = getattr(super_obj, k)
                    super_argus = signature(super_func).parameters
                    argus = signature(obj).parameters
                    if len(argus) != len(super_argus):
                        print("warning: signautre override111: ", ins.__name__, k)
                    if list(filter(lambda pair: pair[0] == pair[1], zip(argus, super_argus))):
                        print("warning: signautre override222: ",  ins.__name__,k)

        def __new__(cls, name, base, dct):
            cls.make_sig(dct)
            cls.make_field(dct)
            ins = super(Meta, cls).__new__(cls, name, base, dct)
            cls.check_func_sig(ins, dct)
            return ins
        def __call__(self, *args, **kwargs):
            for attr in self._field:
                setattr(self, attr, None)
            return super(Meta, self).__call__(*args, **kwargs)

    class A(metaclass=Meta):
        _sign = []
        def __init__(self, *args, **kw):
            rt_argu = self.__signature__.bind(*args).arguments
            for k, v in rt_argu.items():
                setattr(self, k, v)
        def test1(self, x, y):
            pass
    class B(A):
        _sign = ["x", "y", "z"]
        _field = ["attrx", "attry"]
        def __repr__(self):
            return f"B({self.x}, {self.y}, {self.z}"
        def test1(self, x, y, z):
            pass
    class C(A):
        _sign = ["name", "value"]
        _field = ["attr_name", "attr_value"]
        def __repr__(self):
            return f"C({self.name}, {self.value})"
        def test1(self, x, y):
            pass
    b = B(1, 2, 3)
    print(b, b.attrx)
    c = C("haha", "hihi")
    print(c, c.attr_name)

@splite_out
def p14():
    # 动态加属性2
    def Prop(name, ob_type):
        val = None
        _name = "_" + name

        @property
        def field(ins):
            # return val
            return getattr(ins, _name)

        @field.setter
        def field(ins, _val):
            if not isinstance(_val, ob_type):
                print("invalid type setting", type(_val))
                return
            # nonlocal val
            # val = _val
            setattr(ins, _name, _val)

        return field

    class A(object):
        name = Prop("name", str)
        age = Prop("age", int)

        def __init__(self, name, age):
            self.name = name
            self.age = age

    a = A("haha", 100)
    print(a.name)
    b = A(100, "haha")

@splite_out
def p15():
    # 动态创建类1(不用代码事先编写好)
    class Meta(type):
        def __new__(cls, name, base, dct):
            if "_field" not in dct:
                dct["_field"] = []
                print(f"waring: {cls.__name__} no _field")
            cls = super(Meta, cls).__new__(cls, name, base, dct)
            for idx, name in enumerate(dct["_field"]):
                setattr(cls, name, property(operator.itemgetter(idx)))
            return cls

    class NameTuple(tuple, metaclass=Meta):
        # tuple限制了new检查参数数目...
        def __new__(cls, *args):
            if len(args) != len(cls._field):
                raise RuntimeError("signature error")
            return super(NameTuple, cls).__new__(cls, args)

    class NameTuple2(NameTuple):
        _field = ["name", "age", "height"]

    try:
        a = NameTuple("haha", "hehe")
    except:
        b = NameTuple2("hehe", 18, 177)
        print(b[0], b.name, b.age)

@splite_out
def p16():
    import types
    import sys
    # 动态创建新类

    class MyMeta(type):
        def __call__(self, *args, **kwargs):
            print("class calllllll")
            return super(MyMeta, self).__call__(*args, **kwargs)
    def __init__(self, *args):
        print("this is init")
        self.a = 0
        self.b = 1
    def __repr__(self, *args):
        return f"<{self.a}, {self.b}>"
    def mem_fun1(self, *args):
        print("men func", self, args)
    dct = {
        "attr1": "haha",
        "__init__": __init__,
        "__repr__": __repr__,
        "func1": mem_fun1,
    }
    MyCls = types.new_class("DynamicA", (), {"metaclass": MyMeta, }, lambda ns: ns.update(dct))
    MyCls.__module__ = sys._getframe(1).f_globals["__name__"]
    a = MyCls()
    print(a, dir(a))

    # 没有元类
    MyCls2 = type("DynamicB", (), dct)
    b = MyCls2()
    print(b, dir(b))


@splite_out
def p17():
    # exec中的local相当于有get引用了真实local而set是discard的
    a = 1
    loc = locals()
    print(loc)
    exec("a += 1")
    # exec("b = 2")
    print(a, loc)
    locals()
    print(loc)


@splite_out
def p18():
    import dis
    def test1(*args, **kw):
        print("test1111")
    defTest1 = """
def test1(*args, **kw):
    print("test1111")  
    """
    class A(object):
        def __init__(self):
            self.a = 0
        def test(self):
            return self.a
    defA = """
class A(object):
    def __init__(self):
        self.a = 0
    def test(self):
        return self.a    
    """
    dis.dis(A)
    codeObj = compile(defA, "ClsA", mode="exec")
    print(codeObj.co_code)
    dis.dis(test1)
    codeObj2 = compile(defTest1, "Test1", mode="exec")
    print(test1.__code__.co_code)
    print(codeObj2.co_code)
    # test1.__code__.co_code = codeObj2.co_code
    # test1()

@splite_out
def p19():
    pass

@splite_out
def p20():
    from contextlib import contextmanager
    @contextmanager
    def function_auto_close():
        print("function enter")
        yield
        print("function exit")
    with function_auto_close():
        print("test in")


if __name__ == "__main__":
    print("practice p2")
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
    p12()
    p13()
    p14()
    p15()
    p16()
    p17()
    p18()
    p19()
    p20()
```
