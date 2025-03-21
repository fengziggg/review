---
### Bytes:
类似Long  
- 结构：ob_refcnt, ob_type, ob_size, **ob_sval**, ob_hash
- 字符缓存池，初始化创建过的单个Byte会被缓存起来，用Byte的值来进行索引，前1Bytes的256个

操作--caocat：
- 提取两个对象的sval数组，创建一个新对象，合并到新对象的数组区间
- 合并陷阱：a+b+c+d 中间涉及到新的临时对象的创建，每次都会重复创建临时对象且拷贝数据过去也是重复操作
- ![image](https://github.com/user-attachments/assets/1ed0efe9-9135-4c49-93e5-d380dc96de23)

- 可以用b''.join()，不会重复拷贝
- ![image](https://github.com/user-attachments/assets/02d376e4-4a3d-4ceb-9a34-9b5945161062)

---
### str: 
所谓的py3str都是unicode: 
- py的str都是Unicode编码(底层的数值映射规则按unicode来)
  
结构：  
会统计str对象中的MaxChar的区间，根据MaxChar的值决定这个str对象是:  
- PyASCIIObject: state字段的ascii为1,type为3
- PyCompatUnicodeObject： state字段的ascii为1,type为3; 多了三个字段utf8/utf8length/wstr_lenth
- PyUnicode_1Byte_KIND：state字段的ascii为0，type为1； 是PyCompatUnicodeObj
- PyUnicode_2Byte_KIND：state字段的ascii为0，type为2；是PyCompatUnicodeObj
- PyUnicode_4Byte_KIND：state字段的ascii为0，type为4；是PyCompatUnicodeObj
- 字符数据直接接在每种不同结构头的后头，不同结构头用于存储数据表的单位长度不同：UCS1/2/4对应不同单位size

![image](https://github.com/user-attachments/assets/3c187f78-c9fc-4678-99f2-9f7f267bd985)

interned机制：
- 缓存str对象
