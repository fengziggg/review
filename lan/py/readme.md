re/sys._getframe().f_locals  

![image](https://github.com/user-attachments/assets/0233aabc-f9e1-4d76-90de-77d276246380)2024/10/17  
https://github.com/jackfrued/Python-Interview-Bible/blob/master/Python%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8-%E5%9F%BA%E7%A1%80%E7%AF%87-2020.md  
CPython,PIPY  
filter,map  

2024/11/19  
![image](https://github.com/user-attachments/assets/5b53dae0-f7cb-49ac-9aa8-56dd1fe41349)  
![image](https://github.com/user-attachments/assets/09697bf3-1ef9-4c95-8720-9772c23b4f95)  
![image](https://github.com/user-attachments/assets/3856051c-32bc-4455-ba25-b7bdb3117e1a)

?? https://peps.python.org/pep-0422/
https://python3-cookbook.readthedocs.io/zh-cn/latest/c09/p19_initializing_class_members_at_definition_time.html

2024//11/20  
![image](https://github.com/user-attachments/assets/521030e5-27f9-4e23-b245-b7822451aab5)

exec  :
https://blog.csdn.net/LutingWang/article/details/124133994
![image](https://github.com/user-attachments/assets/1910e3ef-0561-41f0-9636-12e336bf26ca)
![image](https://github.com/user-attachments/assets/b3238fce-1901-45cb-8f62-ead0686654f8)


2024/11/21  
![image](https://github.com/user-attachments/assets/2e9a3f49-a999-461a-a025-8117c649e7eb)  
string: https://docs.python.org/3/library/string.html
![image](https://github.com/user-attachments/assets/a7b97bda-b77c-4ca1-aeb4-44d22db80f47) 源码剖析中的计算方式  
![image](https://github.com/user-attachments/assets/e9d7cc87-a71b-49f1-878f-c4189eca9e07)  写错❔
https://rhettinger.wordpress.com/2011/05/26/super-considered-super/  c3线性mro❓
![image](https://github.com/user-attachments/assets/61f94ea3-3ac3-4894-8340-643a09908548)
![image](https://github.com/user-attachments/assets/b9cd0419-f675-47d6-8958-8f75ba4a0a8a)
![image](https://github.com/user-attachments/assets/a7b9ba83-a4d8-4ad6-8807-1a25d22eef22)


2024/11/22
![image](https://github.com/user-attachments/assets/31c34a63-94c7-4754-b3b3-50b6abc0e7fe) 
![image](https://github.com/user-attachments/assets/ccf45782-0412-4dfd-a651-0f89e5f2d4cc)
👌:__getattr__只分发.运算符失败的情况，len和[]和=是其他运算符有各自对应的其他钩子函数

2024/11/25
![image](https://github.com/user-attachments/assets/04060b23-077a-4cf7-a377-1b232ff4cb2e)  
![image](https://github.com/user-attachments/assets/6ee4fcf8-5b11-4be6-bd82-c9a3e0fdca57) ❓  
![image](https://github.com/user-attachments/assets/065cf998-2b15-4239-94c5-26da2a6a9c8c)❓  

管道用法：https://www.dabeaz.com/generators/  
async用法嵌套实现：
yield from基于生成器和携程的并发编程：生成器吧语法块变成可迭代对象(语法块列表)，就吧静态逻辑给数据化下来，数据化下来之后就可以实现切换上下文即携程并发
![image](https://github.com/user-attachments/assets/1af958b4-2acd-485b-9f2d-473c15e596c9)

2024/11/26
![image](https://github.com/user-attachments/assets/22e4ded6-e172-4ea2-bdee-91d4c9125ebb)
![image](https://github.com/user-attachments/assets/9cd2511f-0d75-4215-8baa-8825dd21952e)  ret>2❓  
lambda：
![image](https://github.com/user-attachments/assets/5f265854-5e19-490e-9083-5df4f5bd80d8)  正常函数有独立上下文，因为变量通过参数传递内外隔离，lambda可以捕获上下文参数(闭包)所以不是隔离环境而是运行时环境；lambda更像是express
![image](https://github.com/user-attachments/assets/ffed9616-3270-44e8-b774-22b6d35d800c)  这里lambda里面的n在定义lamda的时候并没有被eval，所以是个变量  
而且这种是在运行时动态定义函数，函数要么是硬编码，或是用api动态的增加属性定义，运行时动态定义 是想当然了，里面的n不会被当作定义而是解释为捕获

参数定义和解释的歧义❓
![image](https://github.com/user-attachments/assets/e85241c5-8574-4849-a3af-e297596dd9cc)

需要保存上下文的简单方法：
单一功能函数类，用成员方法或者重载__call__
函数闭包
yield
特点: obj(*args)(*args)

![image](https://github.com/user-attachments/assets/42a800bc-9e76-412c-a938-6ddf5abf8525)  ❓queue会阻塞yield??


2024/11/28  
![image](https://github.com/user-attachments/assets/f42dc004-1176-4ca4-8856-84cd411742fa)  ：import有不能import的类型？直接import/from import
https://peps.python.org/pep-0420/  : import pep  
![image](https://github.com/user-attachments/assets/64636889-4ab5-440f-9394-e0d16b94a927): 模块里面的东西都是新的指向，模块本身的地址没变，所以用模块索引可以，直接用模块里面的指向内容则没法跟随更新  
![image](https://github.com/user-attachments/assets/fc08fa93-2a1a-401e-880a-3c456e4d2a3d)  
自定义loader和importer:  
https://python3-cookbook.readthedocs.io/zh-cn/latest/c10/p11_load_modules_from_remote_machine_by_hooks.html
https://peps.python.org/pep-0302/

  
2024/11/29   
re  
unicode 相关:  https://nedbatchelder.com/text/unipain.html    
https://python3-cookbook.readthedocs.io/zh-cn/latest/c02/p19_writing_recursive_descent_parser.html  

2024/12/2
![image](https://github.com/user-attachments/assets/bb5850af-98a3-4503-aa16-52ed6ee73a4a)
![image](https://github.com/user-attachments/assets/b563393f-1b76-4e2f-a17e-458ff518ee56)  ❓

![image](https://github.com/user-attachments/assets/2b0545d2-95fd-4daf-bc33-af10bc70a3be)  ❓start没有立即执行

![image](https://github.com/user-attachments/assets/b3a5b1ee-e0c9-442f-bcaa-674e79648788)   ❓with + condition(enter/exit do what)

![image](https://github.com/user-attachments/assets/be011162-d3db-43b8-b423-23576b5e038f)  ❓信号量的意义

![image](https://github.com/user-attachments/assets/76d54914-f164-44b0-ad73-85edf7c8c4b8)

![image](https://github.com/user-attachments/assets/9cbe11a5-a9f2-4f04-be87-a5b7fd5c7951)  ❓原语级别

2024/12/03  

socket/IO是自带阻塞的  
线程是多一个独立执行者(执行完整任务：func)，阻塞是阻塞，目前有event，condition，semaphore，queue，两者是独立不相关的   
但两者常配合使用，阻塞是阻塞线程，可以是主线程也可以是其他线程，再配合分配多个线程，就可以开辟多个执行者，在不同执行者身上加阻塞逻辑来实现同步  
协程是用户自定义中断和自定义调度实现线程的基础，线程相当于os帮你调度不可干预，协程为自己调度

thread.join??  
join可能是吧线程归并到主线程上来??  
asyncio: https://www.cnblogs.com/changting/p/13731794.html  🔎  

![image](https://github.com/user-attachments/assets/9ea65f33-daee-478f-94ed-40a1a9fa87be)  一个int的整数部分pow(2, 15)  
![image](https://github.com/user-attachments/assets/82bfca53-ba99-40a2-bd48-4d5cc1b6c6bb)  ❓  
开启多线程/线程池后会阻塞主进程，看起来好像还是等待一件事情完成，但用处是处理的任务*n并行，pool封装了都会等待future的执行完毕再推进main??   
GIL  
gil是对解释器工作枷锁的(py构建者级别的锁，比如对引用计数的加锁，对cpu还是gpu访问类别区分的加锁，Lock是用户级别的枷锁)
![image](https://github.com/user-attachments/assets/098ca093-77bb-4dfe-bbdd-cfaa838a97a1)  🔍
![image](https://github.com/user-attachments/assets/a8f97a80-7d67-42b9-94c8-b15a158d6260)  ❓
![image](https://github.com/user-attachments/assets/171a8f74-cf8b-439d-8e78-9b954d7e7e3e)  

相互加锁依赖却没有死锁：  
result加的锁目的是用于获取返回值，保证驱动了第一个锁的执行即保证打破一次锁定  
这里用的queue，应该queue还有类似帧的延迟间隔(get/put)，所以start/put/get对应的多线程命令没有与主线程时序矛盾(主线程直接都到close和join了)  
![image](https://github.com/user-attachments/assets/a2c96733-ff72-4888-9ac8-2c5dee19c971)  

任务系统demo:  (循环)
> sche 主循环: 执行task，执行io；  
> task: 添加io事件  
> io事件: 执行io操作添加task

2024/12/04  
![image](https://github.com/user-attachments/assets/843cfaed-b1dc-455a-ad99-cfde069c1646)   
encode/decode  
1. unicode和ascii/utfn: unicode是逻辑上的编码分配表(字符映射关系)  utf/ascii/...是一种内存存储方式(客观编码方式)  
2. 编码方式的差别：一个字符X如果unicode是逻辑10000，那存在内存中可以是按定长(ascii/latin-1)也可以是按变长(utfn)，这两种差异在写入文件的时候导致写入的最终byte内容不一致，一个是有很多0然后按固定为写入和解析，一个是没有无用0然后一个c写入和读取的时候的位数(一般是字节为最小单位)是动态的(比如中文的utf可能是3B也可能是4B)  
3. py3：不管以什么格式进行编码解码，最终都是二进制数据的不同解读规则底层都是二进制数据，只是以一种编码方式持久话的二进制数据被加载出来后(bytearray)以另一种方式解码，那就可能解读错意思乱码，但保存的时候都是保存为二进制数据(不过不是都以utf格式)，即持久化的时候都是跟bytearray打交道，bytearray就是unicode的默认最精简表达方式；  py2的区别❓❓
4. ascii和utf转换报错一般是utf转ascii，因为utf转为二进制数据后一个字节可能被填满但ascii的范围只有128一个字节有一半是非法定义的
![image](https://github.com/user-attachments/assets/6f04d6bf-ac65-46bb-9d7f-0da5a9e23421)

pickle  ❓❓

关于闭包的定义❓❓


