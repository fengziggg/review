---
PyObject_Mallco
PyObject_Dellocate

#### float:
结构：  
PyFloatObj: PY_OBJECT_HEAD, ob_fVa  

创建相关：
1. New路线（最后也是调用的PyFloat_FromDouble来分配内存）
2. PyFloat_FromDouble(3.14)
   - 内存池：链表，直接用PyFloatObj的ob_type来做next链条
   - 检查内存池freelist，有就去除，freenum--
   - 没有就PyObject_Mallco
  
   - 销毁：检查freelist是否达到PyFLOAT_MAXFREE
   - 没有就归还freelist，PyFloatObj->type => freelist, num++
   - 达到上限就PyObject_Dellocate -> PyObject_Free
  
   - ![image](https://github.com/user-attachments/assets/bf09f5d3-2c94-474e-b504-64b7bc30dd23)  

特点：
数据对象是需要频繁创建的，比如运算的临时对象，用对象池缓存  
![image](https://github.com/user-attachments/assets/b8c39f02-4d01-419a-a3d1-7dd2194cebff)  

---
#### int：  
结构：  
- PyLongObj: ob_type, ob_cnt, ob_size, ob_digital  
- ob_size是ob_digital的长度 + 整数的正负
- ob_digital是高位在高地址(小端)
- ob_digital使用30位: 31位防止溢出丢失进位(直接掩码得到进位的)，30是为了和shot15位的情况兼容

创建：
- 小整数池：NSMALLPOSINT/NSMALLNEGINT, small_ints，是个表结构但是int的值可以做下表随机访问

操作：
sign单独计算，a b两数绝对值结算且固定a > b， 然后转换为绝对值加减法 x_add/x_sub(borrow = a[i] - b[i] - borrow; borrow & mask; borrow >> longshit; borrow &= 1)
