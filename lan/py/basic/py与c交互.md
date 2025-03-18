一. 前言：
py本质是c写的，所以py脚本的所有操作，c层面都有一个比脚本能支持的更强大的平台或者说库  
py的动态性也是c提供的，c本身不是动态的，因为py的一切都是c运算出来的，比如py的动态定义其实是创建类对象，在虚拟机运行的时候会执行**定义运算**  
py的所有动态特性，都可以用CAPI编写一个C脚本来实现，比如在脚本里动态创建类对象就类似于py启动时候虚拟机对所有类对象的编译和创建  
吧一些常见的py行为封装后用指令映射，就可以比写C脚本更灵活的组织逻辑，这就是py的字节码的思路  

二. 相互调用的本质
PY脚本环境调用CAPI提供的逻辑，要在CAPI层面包装出一个PY的调用行为，然后在CAPI中用这个包装CPAI去调用目标API
CAPI要调用PY的逻辑，直接调用PY对应的CAPI，如果是自定义的就调用自定义的API

三. 方案：
1. ctypes：
- 编译出dll或者so放在之形目录, 或者用findLibrary搜索
- mod = ctypes.cdll.loadLibrary(path)
- 对于func，则还要指定func的argtype和restype(理解为对Py动态性的一个保护性限制，防止随便传不同对象参数后被乱解析传进去运算返回一个结果不报错)
- 对于func指针，可以ctypes.FUNCTYPE(ctypes.int, ctypes.float...)(func_ptr)转化为可执行对象
- 对于array，获取底层数据，ctypes.cast(buffer, ctypes.pointer)强转为指针模式
- 对于struct直接用类继承ctypes.struct

2.上述的自己写C脚本方案，提供了一个结构体接口往里面填充后这个结构体就会被解析
- PyObect * func(PyObject *self, PyObject *args)
- return Py_BuildValue("i", &var)
- PyArgs_ParseTuple("ifO", &var, &fVar, &obj)
- 提供PyMethodDef[]引用上述结构定义的函数
- PyModuleDef结构体里面引用method数组
- PyInit_FUNC宏定义和return PyModule_Create(mo)

- byte字符串对象PyArgs_ParseTuple(arg, "y", char *s)限制以字节形式转化s, unicode的utf8字符串传递用PyArgs_ParseTulpe(args, "s" char* s)
- 对象字符串得到一个PyObj之后，char* s = PyByte_AsString(obj)，或者byte* b = PyUnicode_Asutf8String(obj)先转为字节流, 再char* s = PyByte_AsString(s)
- 字符串处理的时候，这两者都不会处理中间的null，会当成截断

- 对于Obj可以用PyArg_ParseTuple("O", &obj)
- 如果需要buff可以用PyObject_GetBuffer(obj, buff)得到Py_buffer对象，(不会用00截断？？)

- 对于迭代器，得到Obj之后用PyObject *iter = PyObject_GetIter(obj), PyObject *item = PyIter_Next(iter)获取，记得都要Decref
  
- 对于对象指针，可以用PyCapsule_New来包装实例对象为PyObj，PyCapsule_GetPointer来从pyObj里面获取对象指针，这样不用暴露struct细节给脚本
  
- **（一个常用的套路，可以统一Obj然后Obj再各种转换比如得到buffer）**

- 如果要暴露模块的API给其他模块，可以用结构体包装函数指针，然后创建这个提供API的模块的时候PyModule_AddObj(m)添加结构体对象
- 然后在创建调用着模块的时候，执行一次PyCapsuleImport

- 全局GIL的使用：PYGILState_ENSURE()对C代码加锁，PYGILSTATE_RELEASE解锁
- PY_BEGIN_ALLOW_THREADS**释放**后文的锁允许自己多线程，PY_END_ALLOW_THERAD恢复GIL

- 上述定义的c文件，然后以某种方式加入到C的运行环境，比如用dll/so，或者以前引擎怎么集成到引擎的runtime的(猜测也是编译为dll)
- 可以用distutil.core的setup和Extension构建dll/so后集成到py的比如site_package里面，然后开始import

3. swig方案和cpython方案，都是用他们的语法写函数的定义，然后同样用distutil.core发布，比起原生c拓展是多了一些加工手段比如占位符

