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

--- 
### Encode/Decode:  
![image](https://github.com/user-attachments/assets/295c7f96-31a5-4b00-81d1-3503c541aadf)
- unicode是一份映射规则，utf8是一种存储格式(单位编码长度/定长变长)
- GBK也是一份映射规则，(但算不算一种存储格式就不好说了??)
- GBK与Unicode是不兼容的，量个规则要通过查表来映射起来，比如"我"在unicode是1000，在gbk是2020
- 存储格式可以用byte来检查，反正是一段字节，只是一个是1000用一种编码格式村为[0， 0， 1, 0, 0, 0]还是另一种[1, 0， 0， 0]的区别
- 映射规则是信息，编码方式(内存中字节用于表示编码的排布方式)，或者说其他载体比如试图，声音是数据，是载体，不同载体可以表现同一信息
- **py3是以unicode为中心(映射规则之间，不同载体之间都可以相互转换，但得以一个为中心坐标)，且支持了与另一套映射规则gbk的换算**
- ![image](https://github.com/user-attachments/assets/a7b2f586-5261-4459-8c5b-2577793f25e3)

encode/decode:
- 关注流向，抽象的码值+编码方式=>载体存储形式(Byte里面的内容)，同一个码值不同编码方式Bytes不同，不同码值一样的编码方式Bytes不同，两者都不同反而可能Bytes一样但代表的码值肯定不一样
- 如果是载体 =映射回=>码值，则一份Bytes可以按不同的编码方式解析出不同的码值，所以**反向解码的时候什么样的载体(编码方式)就得用什么方式解码回去(decode)**

与py2的差异：
- py2 类型是str(等同于Bytes)和Unicode， 两者会隐式转换， 文件读写默认是str(Bytes)，unicode只有encode接口，str只有decode接口跟py3相反。。
- Py3 类型是str(unicode) 和 Bytes， 两者无法直接操作(str+Bytes抛异常)， 文件默认读写是str(Unicode)
