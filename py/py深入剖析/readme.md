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
  -  内置的类型对象实例的继承关系都是指向 PyObject_Type(name: object)即都是继承object，自定义类的顶层基类也是，然后派生类再在派生类实例对象中保存继承关系的引用指向父类型对象实例
  -  object的父类没有继续指向自己，说是怕递归
  
----
##### List：  
1. 内存结构体: PyVarObj + allocated + *ob_item (头部固定3+2字)  
2. 常用操作：增append,insert, concat, +, *, 删remove, pop，改=, 查[], index  
3. ob_item指针数组动态扩容：  
  > list_reisize: 增长模型 new_size + new_size//8 + (new_size<9)3:6，❓❓（0, 4, 8, 16...怎么来），PyMem_alloc  
  > list_ass_slice: 对内存上区间数据进行修改，删除即前插(O(n))，或者直接该值，会释放对象

4. append: 检查，resize，引用，setItem
5. insert: 检查，resize，引用，前插迁移新内存上数据
6. pop: 检查，调用resize或者ass_slice，resize占用list的引用/ass_slice要预先增加一次引用
7. remove: 遍历比对后ass_slice 
