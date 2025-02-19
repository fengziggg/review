### Chart1  
#### 纹理：  
数据加载 + loop:设置状态+render  
1. ##### 数据加载:  
两步：加载到GPU相关容器对象+设置格式  
> 顶点数据：加载到VAO/VBO/EBO + 设置VertexAttributePointer格式  
> 纹理数据：加载到texture对象 + 设置TexParameter格式  
> 着色器程序(指令数据)：加载到shader对象/Link到Program对象 + 设置上述两种数据的对齐方式  
> > - **(顶点数据的对齐是写在VS里面的layout编码死的，纹理数据的对齐是在FS里面预留Sampler的uniform对象)**  
> > - 此外着色器还会加载顶点数据和纹理数据到着色器里面，这里比较特别的是顶点数据自动加载，**纹理数据需要手动加载指定的Texture位置而不是texture数据本身**，而且Texture位置支持动态改变，每个位置匹配的Texture也支持动态Active后Bind，不是很懂为什么需要这么高灵活度(SetUniform -> TEXTURE_X -> Active&Bind(texture)??)

相关容器有：VertexArray/ArrayBuff，Texture/SamplerND，Shader/Program   

2. ##### 设置状态：  
应该是对应渲染管线的drawCall，每一次drawCall都是要**加载**一个渲染对象的所有东西，**设置**最新的动态状态(Uniform，Parameter)，然后执行渲染命令(DrawElemnt)
环绕状态 :  
```  
  glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT)
  glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT)
```  
缩放状态/多级渐远mipmap(复合状态，放大或者分辨率低适合linear，缩小或者分辨率超了适合neareast)：  
```
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

![image](https://github.com/user-attachments/assets/d74b794e-42b1-470f-89ea-cd210e87b62f)


----
#### 变换：  
向量矩乘M(行矩阵)*N(列矩阵)：接口对齐d(行向量) == d(列向量)，然后就是将列矩阵投影到行向量坐标系构成的空间，新空间的维度是行向量的数目  
旋转矩阵，缩放矩阵，变换矩阵...  
齐次坐标系：x, y, z, w，提供一个冗余w供xyz进行加和的功能(常数项)，齐次项如果是0就是方向，非0就是一个点？？  
![image](https://github.com/user-attachments/assets/7ab52bc4-e3b4-4577-96f9-56f7e5838f90)  
stbi的特殊编译导致没法在多个cpp复用，设置cpp文件排除编译了...  
❓万向锁和四元素  


----
#### 空间坐标：  
- 局部坐标(空间) * Model(模型变化) ==> 世界坐标(空间) * view(**视角**变换) ==> 视角坐标(空间) * proj(投影矩阵) ==> 裁剪坐标(裁剪空间,到此转化为**NDC**) [gl自动进行**视口**变换 ==> 屏幕空间]  
> localPos -> MODEL -> worldPos -> VIEW -> viewPos -> PROJ -> clipPos(NDC, 抛弃频幕外) [-> VIEWPORT -> screen]  
> 这是在vs的流程中，不涉及fs流程，转化ndc的时候会抛弃屏幕外顶点即修改了渲染的三角图元的顶点(只有屏幕范围内包含在渲染范围)   
> 抛弃顶点不等于双面渲染，面剔除那些  

- 投影矩阵：  
```
glm::mat4 proj = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
```
> 正交投影运算是直接用边界判断：l，r，t，b，n，f  
> 投影矩阵的运算逻辑：fov，w/h，n，f(fov和n就可以确定近平面的Hn然后得到Wn)    

- 右手坐标系：  
 >![image](https://github.com/user-attachments/assets/e93ac565-4a35-4490-8a9e-9435e4ce3cce)

![image](https://github.com/user-attachments/assets/ca1ce044-71af-4f43-8242-6319ae9c5039)

----
#### 摄像机：  
主要改变view矩阵，proj的fov  
model是 Trans*Scale*Localpos  ==> WorldPos  
view是  -(Scale)*(-Trans)*WorldPos ==> ViewPos，这个过程是先位移在旋转，旋转是摄像机的本地旋转  
所以摄像机需要两个参数pos和dir，也习惯用pos和eye，lookAt矩阵就是[(Scale) * (-Trans)]  
摄像机坐标系: Cross(Front, WroldUp)得到Right，Cross(Front, Rgith)得到CamUp，施密特正交  
施密特正交：  
>> 空间内一组向量可以表示一个子空间，那么可以有这组向量得到该子空间的正交基，两个不共线向量就可以根据投影得到基于其中一个向量的另一个垂向量，第三个向量又可以和前两个**正交基**投影得到第三个正交基，以此类推  
>> 投影只是得到合适的做差长度，真正分量的产生是在做向量差上得到  
>> 正交基不唯一，根据第一个向量不同而不同，摄像机坐标系就是根据Front得到的正交基  
>> ![image](https://github.com/user-attachments/assets/11961049-2871-461a-8cd6-988388020d93)  
>> 正交基yn, α假设为y1，绿色向量β为α的正交基y2向量，黄色为与α，β分别投影后得到的y3  
>> up对于front来说是一个非共线向量而已，所以pitch不能是+-90°  

----
---- 

### Chart2  
#### Color  
看到的颜色是 light*matReflect(或者说1-matAbsorb)  

管线各个阶段：坐标变化所在vshader各个阶段，剔除所在阶段  
坐标在vshader和fshader:gl_fragcoord??  

uniform color归一化时机

预乘  
