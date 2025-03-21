#### py底层结构
py是C写的，没有对象C+的对象，py的对象全是结构体实例
定义类(这些是**定义**，不会跑起来的)：
```
#define 
struct PyObject{
  ob_type
  ob_refcnt
} PYOBJECT_HEAD

#define 
struct PyVarObject{
  ob_type
  ob_refcnt
  ob_size
} PY_VAROBJECT_HEAD

#define
struct PyTypeObject{
  PY_VAROBJECT_HEAD
  tp_name
  tp_base
  tp_basesize  # 内存分配的时候分配多少内存的依据
  tp_itemsize  # 内存分配给元素分配内存的依据
  tp_dealloc
  tp_print
  tp_getter/setter

  tp_asNumber
  tp_asSeq
  tp_asAssociate
  ...
}
````
**实例**，(会运行起来的)：  
- 类型实例(涉及操作集方法的添加)：PyFloat_Type/PyType_Type/PyBaseObj_type
- 对象实例(设计到具体的数据存储字段的引用)：PY_FLOATOBJ/...  
![image](https://github.com/user-attachments/assets/3b7e97eb-12fd-4896-8d89-96ace15463af)
![1742497089651](https://github.com/user-attachments/assets/6c1ee405-1a92-45e9-b68d-ea31c3953e9d)

---
#### 对象生命周期：
创建：
  - 从C函数接口创建比如PyFloat_FromDouble(3.14)
    
  - 用类型对象自身的构造函数，底层走type_type类型对象的c接口逻辑
  - (PyXX_TypeIns).ob_type.tp_call() ==> (PyType_TypeIns).tp_call(PyXX_TypeIns) ==> type_call(PyXX_TypeIns)，会进行内存分配(怎么分配❓)
  - tp_call里面是：obj = (PyXX_TypeIns).tp_new(PyXX_TypeIns) ==> (PyXXIns).tp_init(obj, *args, **kw)
  - new是类方法参数还是TypeIns，init是实例方法参数是obj

多态：
  - 每个对象都有一个Type对象，方法都是Type对象的
  - 所有实例对象都可以用PyObject引用，只有两个字段ob_cnt, ob_type，然后就是自定义数据字段
  - 公共基类的前提有了，虚表的机制靠Type引用ob_type，所以可以多态

行为：
  - 操作集，封装了某种类型的一些列函数结构体
  - 数值操作类型tp_asNum, 序列操作类型tp_asSeq, 关联操作类型tp_asAssosiate
