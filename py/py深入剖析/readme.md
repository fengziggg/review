#####1. C structure  
1. 结构体define(摸版):    
  - 类对象结构体定义: PyObject, PyVarObject, PyTypeObject  
  - 实例对象结构体定义: PyFloatObject, PyIntObject  

2. 类对象实例(被高级语言的关键词干扰，其实都是内存对象来着):  
PyTypeObject实例化出:  
  - 上层基础类型对象: PyObject_Type(name: object), PyType_Type(name: type)  
  - 内置基础类型对象: PyFloat_Type(name: float), PyInt_Type(name:int)...  
  - 自定义类型: class XXX(object), 类的定义是硬编码的，估计运行时执行时也是将硬编码的类走与内建类型类似的流程  
    
3. 实例对象：  
  - 调用类对象比如float的构造函数float()，然后执行一段内存分配逻辑创建的 由PyFloatObject定义的实例比如3.1415926  

4. 引用关系:
  - 每一个实例对象都要引用一个类型对象：一个float实例对象3.14159的c结构体中type指针指向类对象 PyFloat_Type(name: float)
  - 类对象也是个实例对象，同样指向类型对象即更上层的PyType_Type(name: type)，同时type的类型也是指向type本身
  - 类型指向是用于存放操作方法的，即实例调用的方法都要去对应的类型实例里面找，类型实例都是单例比如float等(每种类型的定义对象只需要一个)(自定义类对象实例应该也是如此)

  - 继承关系存在类型对象里面的ob_base引用里面,即每个实例上是没单独村一份继承关系的(没必要)
  - 整体的追溯逻辑是先从实例type找到类型对象引用，然后类型对象的继承关系里面索引继承链
  
----
