- early-z
- 后处理：
  
  - 暗角
    > 
  
  - LUT
    https://zhuanlan.zhihu.com/p/623210058  
    https://hello-david.github.io/archives/f116047e.html
    https://zhuanlan.zhihu.com/p/604897964  
    > 颜色映射函数，最终输出颜色，给定一个2D化的3D纹理图(1d排列或者2d排列)，可能是256*16(16*16*16)
    > 然后根据片元原色rgb，映射到LuT图得到颜色进行替换，可以可以做一些插值运算比如邻接b通道间插值
    > ![image](https://github.com/user-attachments/assets/d1a5f27a-c1d3-49e7-8255-fa9e18d0c30f)
    > ![image](https://github.com/user-attachments/assets/036017ca-c3ae-458b-a8e7-128445f5ee00)



- HSR，渲染路径DR，TBDR...
- LoD，minmaps是纹理的Lod
  > 整个资产的多方案，顶点，纹理
  > 纹理相关的Lod就是Minmap
- 场景管理，遮罩剔除，顶点裁剪：
  > 场景管理(四叉八叉)是用户级别自定义范围的剪枝
  > 遮罩剔除也是用户级别，但范围限定在摄像机视觉空间内(frutum)
  > 顶点裁剪是管线级别的(vs.裁剪空间输出，且被ndc)，干预不了
- fs的片元和像素
  > fs的偏远是一个颜色单位对象，经过一些列处理(alpht, depth, stencil ...)才会输出为像素,有的可能中途就被丢弃，是候选像素
  > 片元也有提前剪枝，背面剔除，各种测试，early-z
- 材质相关  

color:  
https://zhuanlan.zhihu.com/p/129095380  
