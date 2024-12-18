#### 静态链表：  

||优点|缺点|
|---|---|---|
|数组|随机访问|内存动态分配(头尾扩容和中间扩容)<br>也不是不可以做，但数据体大从新分配大块内存失败频率高，同时中间切片操作会涉及插入修改操作内存范围大|
|链表|内存动态分配|随机访问|

两者结合就是希望有随机访问能力也要有动态扩容能力，折中的点：  
1. 将数组随机访问的缺点--操作涉及的内存范围大(val大的缘故)，改为操作val的指针(val小了且定长
2. 但依旧有realloc和插入修改
3. 再多一次寻址跳转机制

#### 红黑树：

#### 跳表(Allco??):

#### 图：  
https://www.cnblogs.com/BigJunOba/p/9247682.html  
https://github.com/LtLei/articles/blob/master/data_structure_and_algorithm/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B9%8B%E5%9B%BE%E7%9A%84%E6%A6%82%E5%BF%B5%E5%92%8C%E5%AD%98%E5%82%A8.md  
> 邻接矩阵
> 邻接表  
> 十字链表
> ![image](https://github.com/user-attachments/assets/c7c12a8e-d485-413c-8971-ae9126da1077)


#### 堆，二叉堆，fab堆：

#### 优先队列：

