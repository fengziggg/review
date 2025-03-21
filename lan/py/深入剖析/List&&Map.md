#### List:
源码补充：
- 切片
- PyMen_Reallocate
- Py_Decref
  
动态扩容顺序表
结构体：
关键字段：
- ob_item数组表指针
- ob_allocator目前划定的容量上限
- ob_size已经分配的元素数量(PyVarObj_Head)

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


---
#### Map：
底层：
- hashMap
- 可Hash对象：1. 不可变对象，2. hash值可以比较运算

结构体：  
![image](https://github.com/user-attachments/assets/86d3e095-cd85-4d99-aff2-e9409155fdad)  

![1742548386190](https://github.com/user-attachments/assets/90707b97-6d93-457a-903c-0239487a29b6)
PyDictObject(只是个PyObject的主体结构引用住真正的KeyObject)：ma_keys（存储key表），ma_value（旧版字段可以忽略）  
KeyObject(具体维持整个map的数据的表)：  
- dk_indices(索引表，**不是被KeyObj引用而是直接分配在KeyObj上**)
- dk_entries(entry结构体的引用，**直接顺序存在这个个对象的内存块尾部了**)
- 一些容量描述字段：**dk_size**(分配的这个KeyObj的容量，也是本次的理论上限)，**dk_nentries**(应该是entry表的实际数目，包含了被占用的)，**dk_usable**(indices中的可用个数，用来判断是否触发边界)
EntryObj：me_key, me_val, me_hash  

**动态扩容**：
- 与list不同，不是到达分配上限才会扩容，hash的分配位置不可预测，除了满了还有减少冲突，所以需要有一定的空余位置来减少空缺
- **边界**: 所以resize边界是逻辑size的: size*1/3 < staticSize < size*2/3
- **真假resie**: resize有真resize(indeces达到边界，重建索引mod会变和分配entrys内存快)，假resize(entry没变，重新分配了entrys内存快长度不变剔除废弃的)
- **增长率**: 真resize按大小为2^n齐次幂分配newSize，且要让触发resize的**真实indice个数**(2/3*oldSize)在newSize的1/3之上防止死循环resize,一般是三倍(GROWTH_RATE)
- 即初始8(2-5) -> 16(5-10) -> 32(10-20)...

**删除条目**：
- index槽位数据清空标记为dummy，但不能把槽位空出来，因为用线性探测的hash冲突决定的，但有后来人映射到这个key可以被他覆盖(只是不要留空中断探测)
- entrys不进行回收，直接留着等待resize重新分配内存后，整理有效的ele然后整体复制过去
- 所以这里entrys和index的数目不是完全同步的，entrys不会真的分配size只会size*2/3，index是要真的分配size长度
- 所以如果entrys满了，会触发resize，但indexs是还有可复用空间的(dummy可以转active)，这种是假resize只会重新分配entrys
- 如果是indices达到了分界，则会真resize

其他特点：
- 不同于空list，空dict也分配了8个单位，初始字节: PYDICT_MINSIZE
- 索引的单位长度根据数据规模来，默认1Byte即map存储在128个数据内的情况
- **分索引和entry的原因是因为冗余空间存在，拿索引去冗余的代价比较小，entry表是顺序表**
- py3.7之后的顺序结构就是因为引入了entry的顺序存储

