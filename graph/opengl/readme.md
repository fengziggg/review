#### 纹理：  
数据加载 + loop:设置状态+render  
1. ##### 数据加载:  
两步：加载到GPU相关容器对象+设置格式  
> 顶点数据：加载到VAO/VBO/EBO + 设置VertexAttributePointer格式  
> 纹理数据：加载到texture对象 + 设置TexParameter格式  
> 着色器程序(指令数据)：加载到shader对象/Link到Program对象 + 设置上述两种数据的对齐方式 **(顶点数据的对齐是写在VS里面的layout编码死的，纹理数据的对齐是在FS里面预留Sampler的uniform对象)**  
> > 此外着色器还会加载顶点数据和纹理数据到着色器里面，这里比较特别的是顶点数据自动加载，**纹理数据需要手动加载指定的Texture位置而不是texture数据本身**，而且Texture位置支持动态改变，每个位置匹配的Texture也支持动态Active后Bind，不是很懂为什么需要这么高灵活度(SetUniform -> TEXTURE_X -> Active&Bind(texture)??)

> > 相关容器有：VertexArray/ArrayBuff，Texture/SamplerND，Shader/Program   

2. ##### 设置状态：  
应该是对应渲染管线的drawCall，每一次drawCall都是要**加载**一个渲染对象的所有东西，**设置**最新的动态状态(Uniform，Parameter)，然后执行渲染命令(DrawElemnt)


----

管线各个阶段：坐标变化所在vshader各个阶段，剔除所在阶段  
坐标在vshader和fshader:gl_fragcoord??  

uniform color归一化时机
