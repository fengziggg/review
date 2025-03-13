#### 后处理：
  
  - 暗角
    计算uv距离中心的距离，平方化(变成曲线)后被1减去来设置上限为0，乘上系数(二次函数A)再Saturate阶段控制亮区范围和指数变换进行扩散，随后采样  
    ![image](https://github.com/user-attachments/assets/b2d609a8-84de-4136-bee3-21507507d14d)
  
  - LUT  
    https://zhuanlan.zhihu.com/p/623210058  
    https://hello-david.github.io/archives/f116047e.html
    https://zhuanlan.zhihu.com/p/604897964  
    > 颜色映射函数，最终输出颜色，给定一个2D化的3D纹理图(1d排列或者2d排列)，可能是256*16(16*16*16)
    > 然后根据片元原色rgb，映射到LuT图得到颜色进行替换，可以可以做一些插值运算比如邻接b通道间插值
    > ![image](https://github.com/user-attachments/assets/d1a5f27a-c1d3-49e7-8255-fa9e18d0c30f)
    > ![image](https://github.com/user-attachments/assets/036017ca-c3ae-458b-a8e7-128445f5ee00)

#### matcap  

#### early-z  

#### HSR，渲染路径DR，TBDR...  

#### LoD，minmaps是纹理的Lod  
  > 整个资产的多方案，顶点，纹理
  > 纹理相关的Lod就是Minmap

#### 场景管理，遮罩剔除，顶点裁剪：  
  > 场景管理(四叉八叉)是用户级别自定义范围的剪枝
  > 遮罩剔除也是用户级别，但范围限定在摄像机视觉空间内(frutum)
  > 顶点裁剪是管线级别的(vs.裁剪空间输出，且被ndc)，干预不了

#### fs的片元和像素
  > fs的偏远是一个颜色单位对象，经过一些列处理(alpht, depth, stencil ...)才会输出为像素,有的可能中途就被丢弃，是候选像素
  > 片元也有提前剪枝，背面剔除，各种测试，early-z

#### 材质相关  

#### lightMap,lightProbe,体积光

#### color:  
https://zhuanlan.zhihu.com/p/129095380  

#### computeShader:
复用gpu的并行算力资源：
写shader：名字，输入输出参数RWStructBuffer/RWTexture2d，编写函数体，创建shader资产  
unity脚本：shader.FindKernel找到目标函数，SetParam，Dispatch分配线程组

#### 各种着色模型：
flat/phong/blingphong/...



---
1. 基础光照与阴影

    动态阴影（Dynamic Shadows）

        应用：通过 Shadow Map 技术实现物体投射动态阴影。

        Unity 实现：

            光源（Directional/Point/Spot Light）启用 Shadows 选项。

            调整 Shadow Distance 和 Shadow Resolution 优化性能。

    全局光照（Global Illumination, GI）

        应用：模拟光线多次反弹的间接光照效果。

        Unity 实现：

            使用 Lightmapping（Baked GI）或 Enlighten/Progressive GPU（Realtime GI）。

            在 HDRP/URP 中通过 Light Probe 和 Reflection Probe 增强动态物体光照。

2. 材质与表面效果

    PBR（基于物理的渲染）

        应用：通过 Metallic-Smoothness 或 Specular-Glossiness 工作流模拟真实材质。

        Unity 实现：

            使用 Standard Shader 或自定义 PBR Shader（如 HDRP 的 Lit Shader）。

    透明与半透明效果

        应用：玻璃、水、雾气等。

        技术：

            Alpha Blending（透明混合）。

            Alpha Testing（镂空效果，如树叶）。

            折射（Refraction）：通过 GrabPass 或屏幕空间扭曲实现。

    法线贴图（Normal Mapping）

        应用：通过法线贴图模拟表面凹凸细节，无需增加几何体。

        实现：在材质中绑定法线贴图，Shader 中使用 UnpackNormal 函数解码。

3. 后处理效果（Post-Processing）

    Bloom（泛光）

        应用：模拟高光溢出效果（如发光物体、灯光）。

        实现：通过提取高亮区域并叠加模糊结果。

    抗锯齿（Anti-Aliasing）

        技术：

            MSAA（硬件多采样，适合几何边缘）。

            FXAA/TAA（后处理抗锯齿，性能更低）。

    环境光遮蔽（Ambient Occlusion, AO）

        应用：模拟物体接触区域的阴影（如墙角、缝隙）。

        技术：

            SSAO（屏幕空间环境光遮蔽）。

            HBAO（更高质量，但性能消耗更高）。

    运动模糊（Motion Blur）

        应用：快速移动时模糊画面，增强速度感。

        实现：基于摄像机或物体运动矢量的模糊。

    景深（Depth of Field）

        应用：模拟相机焦点外的模糊效果。

        实现：基于深度图的高斯模糊或 Bokeh 散景。

4. 屏幕空间效果

    屏幕空间反射（Screen Space Reflection, SSR）

        应用：实时计算光滑表面的反射（如水面、金属）。

        限制：无法反射屏幕外的物体。

    屏幕空间阴影（Screen Space Shadows）

        应用：在屏幕空间计算动态阴影，适合局部细节优化。

5. 体积效果

    体积光（Volumetric Lighting）

        应用：模拟光线穿过空气的效果（如上帝光、手电筒光柱）。

        实现：通过光线步进（Ray Marching）或后处理模糊。

    雾效（Fog）

        应用：模拟大气雾效，增强场景深度感。

        类型：线性雾、指数雾、高度雾。

6. 特殊效果

    粒子系统（Particle System）

        应用：火焰、烟雾、魔法特效等。

        优化：使用 GPU Instancing 或 VFX Graph（HDRP）。

    全息投影（Hologram）

        实现：通过扫描线、噪波贴图和菲涅尔效应模拟。

    卡通渲染（Cel Shading）

        应用：动漫/卡通风格（如《塞尔达传说》）。

        技术：

            硬边缘光照（Ramp 贴图）。

            轮廓线（通过法线或深度边缘检测）。

7. 高级渲染技术

    光线追踪（Ray Tracing）

        应用：实时精确反射、阴影和全局光照（需硬件支持）。

        Unity 支持：通过 HDRP 的 Ray Traced Effects。

    延迟渲染（Deferred Rendering）

        应用：复杂光照场景（多光源性能更优）。

        Unity 实现：在 URP/HDRP 中配置渲染路径。

    动态分辨率（Dynamic Resolution）

        应用：在性能不足时降低分辨率，保持帧率稳定。

8. 风格化渲染

    像素化（Pixelation）

        实现：通过降低屏幕分辨率或后处理像素化滤镜。

    水彩/油画风格

        技术：边缘模糊、颜色扩散、纸张纹理叠加。

总结

    写实项目：优先使用 PBR、全局光照、抗锯齿、SSR 和后处理堆栈。

    风格化项目：结合卡通渲染、像素化或自定义 Shader 实现独特风格。

    性能优化：根据目标平台选择技术（如移动端避免光线追踪）。

学习这些效果时，建议结合 Unity 的 Shader Graph 和 VFX Graph 工具实践，同时参考官方文档和社区资源（如 Unity Learn）。
