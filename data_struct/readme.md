#### 静态链表：  

||优点|缺点|
|---|---|---|
|数组|随机访问|内存动态分配(头尾扩容和中间扩容)<br>也不是不可以做，但数据体大从新分配大块内存失败频率高，同时中间切片操作会涉及插入修改操作内存范围大|
|链表|内存动态分配|随机访问|

两者结合就是希望有随机访问能力也要有动态扩容能力，折中的点：数组随机访问的缺点--操作涉及的内存范围大(val大的缘故)，改为操作val的指针(val小了且定长，但依旧有realloc和插入修改)，再多一次寻址跳转机制

#### 红黑树：

#### 跳表(Allco??):