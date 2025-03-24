```
transform
for nodi in left_children:
  updatetrnasform()

  draw()

draw_self()
for nodi in right_children:
  updatetrnasform()
  draw()
```
![image](https://github.com/user-attachments/assets/9a65b008-25cf-4d5a-94f0-e93c0422347a)


1. 用途：
   - 决定绘制顺序，GlobalZ/localz(都是先处理<0，再0, 再>0)
   - 决定坐标变换关系：根据锚点(不同于u3d锚框锚定父节点相对位置)
   - 决定事件分发
   - 中序遍历
  
2. 都是autoRelease()
