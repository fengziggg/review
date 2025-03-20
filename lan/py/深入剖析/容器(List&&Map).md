#### List:
切片  ？？
PyMen_Reallocate  ？？
Py_Decref ??

动态扩容顺序表
结构体：
关键字段：ob_item数组表指针, ob_allocator目前划定的容量上限, ob_size已经分配的元素数量(PyVarObj_Head)
![image](https://github.com/user-attachments/assets/c9ec89cf-3d14-4575-989d-c45bb60b6bb4)


扩容逻辑：
- 每次操作肯定都是+1或者-1：append,pop,remove，所以list_resize真正要扩容的时候reqSize都是allocated的基础上+-1
- 触发边界(reqSize超出这个区域就会触发list_resize)： allocated/2 < reqSize < allocated
- list_resize公式：(reqSize/8)+(3?6)+reqSize, 0->4->8->16->25...
- 平均下来每次append时间复杂度是o(1)，不是每次append都要list_resize
- list_resize: 计算公式后核心PyMen_Reallocate大致是分配新内存，迁移旧数据，释放旧内存

尾插逻辑append，O(1)：
- 获取reqSize
- 检查溢出
- list_resize尝试重分配
- Py_Incref

头插O(n)：
- 获取reqSize
- 检查溢出
- 尝试list_resize
- 迁移后续元素
- Py_Incref
- 插入

切片list_ass_slience:有两个不同的语义，批替换或者删除

pop:
- 修正idx(可能为负)
- 一些检查(修正后是否依旧负或大于size)
- 如果是最后一个，则直接list_resize，不用Py_Decref？？
- 如果是中间，则先Py_Incred，再用list_ass_slience切片(内部会释放元素)，Py_Decref

remove:
  - for循环对比找到元素
  - 然后list_ass_slience删除
