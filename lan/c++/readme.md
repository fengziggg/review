1. alloc：
   Union链表？？

2. traits:
  GP有点类似PY的实现思路，不管传入类型是什么，只管用里面的方法，没有就编译失败，相当于传入是类对象的时候
  template  
  typename  
  用类内嵌类型捕获T作为value_Type(❓指针类型还是原类型？？)  
  再用一个struct或者类模板定义偏特化捕获上一个捕获类(装饰器，嵌套？)  
  
