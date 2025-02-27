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
万向锁问题：  
question：欧拉角不是增量每一个都是状态量??  
一个特性：一个轴旋转90°，剩下两个轴会交换(可能是另一个轴的反向)  
两种问题：是给定旋转量(旋转矩阵，四元素，欧拉角等)求一个V变换后的值，还是给两个V求其旋转量(旋转量进行差值同理)  
三种旋转：欧拉旋转角，轴角旋转，四元素旋转  
- 欧拉旋转角：
1. 欧拉旋转是分轴来叠加旋转的，轴要区分世界坐标轴还是局部坐标轴，且旋转轴有次序(各家都有各家的方案U3d为ZXY，UE4为ZYX)  
2. 按次序对轴进行旋转运算叠加，如果是世界坐标系则是**静态欧拉角**(没有万向锁问题)，如果是局部坐标则因为修改是按轴来的，每次修改会影响后面的轴，这种是**动态欧拉角**(万向锁就是在第一个特性的基础加上动态影响后面轴的原因产生的)
3. 相互垂直的纬线圈是静态欧拉角，经纬度圈是动态欧拉角(可以先Pitch再Yaw？？)  
  ![1740060704320](https://github.com/user-attachments/assets/e574d02a-4ac1-4804-b2e1-03f0b02308cd)  
4. 叠加旋转计算，用矩阵自定义顺序累乘:  
   ![image](https://github.com/user-attachments/assets/9d6978cd-c227-42c4-88cb-ca7ce13fddca)  
5. 万向锁定义：
   - - Unity中在第二个轴打满90°的时候会出现Y调整旋转与Z同效果的情况，是因为Unity的Y是惯性坐标系(显示在物体上的世界坐标Y)
   - - 数学上的定义是不管什么次序的欧拉旋转，绕第二个轴旋转时候如果打满，第三个轴的效果会作用在第一个轴上，原来第三个的旋转效果丢失(这里的轴都是指局部坐标系)
   - - https://www.cnblogs.com/HDDDDDD/p/15067619.html
   - - 很多资料很差劲，世界坐标局部坐标都没分清，什么是万向锁的定义也没讲清...

- 轴角旋转——[R1, R2, R3, theta]：  
1. 运算就用现成的那个复杂矩阵(glm一般用的那个)，差值不可以因为theta和前面三个不是统一维度空间的    

- 四元数是轴角旋转的扩展：
1. 四元数是多维复数空间，复数空间是向量空间所以有几何特性(可以用三角展开相关的)，还有自己的特殊运算(乘，叉乘，求逆，共轭等)以及i^2 == -1带来的圆特性  
2. 四元数可以直接跟向量运算，是个工具 qvq(-1)
3. 可直接求差值, 可与欧拉旋转角，轴角矩阵相互转换
4. https://blog.csdn.net/silangquan/article/details/39008903


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
![image](https://github.com/user-attachments/assets/a5835806-d39f-48f6-9840-c1e78d8eb199)  

---
#### light  
vs空间与fs空间  
1. 在vs和fs中传递的in out，要留意fs的都是vs的有限个顶点数据(任何顶点数据，所有通过out传递的，包括中间的自定义辅助数据都是与该顶点绑定)的插值后的光栅数据  
  > 所以fs依赖到顶点的数据要考虑：拿差值后的数据进行处理**func(Lerp(data))** 与 在顶点空间处理完后的数据自动插值过来**Lerp(func(data))** 是否等价  
2. 法线在model变换后需要修正，model非等比例变化后，法线也跟着变的情况下数据也不对，法线不变的情况下(原法线)也不对，得用**原法线**进行另一个model_norm（tranpos(inverse(M))）变换  

phong模型：  
都是反射光乘以比例再叠加来贡献最终光，然后再light * color  
1. ambient 固定比例  
2. diff 与光源位置和每个顶点位置和法线有关，比例系数就是夹角： dirLight * norm   
3. spec 在diff的基础上再与视角有关，比例系数是反射向量与视线向量的夹角，再用指数进行扩大：pow(reflect(dirLight, nrom) * eyeDir)  
4. fragColor = (fact_ambient + fact_diff + fact_spec) * dirColor * objColor  
![image](https://github.com/user-attachments/assets/ca2888a2-ebf1-4c3f-98c8-467e21b444c1)

---
#### mat  
进一步拓展，最终渲染颜色：三种成分反射光的比例，三种成分光的角度投影系数，三种光成分固定系数(材质)*物体反色颜色  
light[3 vec3] * mat[3 vec3] * rat[3 float]   
![image](https://github.com/user-attachments/assets/0f3925af-3e93-41b0-929b-37092368d7e2)  

---
#### diffusee mat  
物体反射颜色和三种光模式常量系数统一纳入一个材质表征：即材质是objColor*strengthness  
![image](https://github.com/user-attachments/assets/97b58e21-d94b-486d-b17b-e5af192b03bf)

--- 
#### light cast  
光颜色 * 光源类型的衰减系数 * 物体颜色 * 光照模型三组分的夹角系数  
光源衰减系数：  
> 光源类型：平行光，点光源(1/dist^2 反平方比公式)，聚光(片元光线与聚光中心方向夹角系数)  
> 与光源类型，**光源距离**，**光源中心朝向相关**  
![image](https://github.com/user-attachments/assets/c5ae5cc5-456e-4d52-aa75-0ac662781acb)

---
#### assimp
模型加载，主要就是用这个库读取数据然后自行拆解数据出来用  
资源/场景一般会尽量包含多的数据：节点，网格，顶点，索引，纹理，贴图内容，甚至摄像机  
assimp的逻辑是场景存node，mesh，mat，然后node指向mesh，mesh指向mat  
assimp库最新的已经更教程不一致了(25/2/28)，编译出来后lib和dll不再同一个地方(都要放exe目录)，然后include不全src和编译工程各一些，都要放一起  
dll：动态运行，不编入exe里面； lib：如果是dll的则lib只是充当指引dll位置和里面函数位置的指引文件，如果是单纯静态库则是数据和代码会被编入exe；.h是都要的  
![image](https://github.com/user-attachments/assets/d9528094-0587-47a4-9fc7-3a51077b88d1)
![image](https://github.com/user-attachments/assets/af1229d5-a68a-4bb4-a01a-ec6372b35c37)  



#### 阴影 
阴影就是对两个视角的深度图的时差片元加一些阴暗表现  
总体思路就是正常渲染的时候在片元渲染之时，检查一下该片元是否处于阴影中(是否在光源视角被z丢弃)  
> 该片元可能是要被深度过滤丢弃的，但也不影响他的阴影检测
> 光源视角深度图哩的深度信息，包含了一个obj片元对另一个obj片元的遮盖(如果渲染会被丢弃)，也包含了一个obj的一些前面片元对后面片元的遮盖(会被丢弃)
> 所以阴影深度图就是光源视角所有被丢弃z丢弃的片元，但记录z维度的最值只需要只需要一张fb就可以搞定

平行光投影的光源是个箱体(范围有限只有摄像机视角空间与之重合的部分能有投影，是不是可以用阴影贴图解决❓❓)
点光源用立方体贴图覆盖全空间(也有范围但是是farplan限制，但平行光的更小且不能用立方体贴图平替)
CSM ❓❓


管线各个阶段：坐标变化所在vshader各个阶段，剔除所在阶段  
坐标在vshader和fshader:gl_fragcoord??  

uniform color归一化时机

预乘  
