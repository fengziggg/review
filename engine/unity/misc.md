#### 坐标tranasform
2. 法线向量转换：UnityObjectToWorldNormal
    // 转换法线到世界空间
    float3 worldNormal = UnityObjectToWorldNormal(v.normal);
    // 计算漫反射光照
    float diffuse = max(0, dot(worldNormal, _WorldSpaceLightPos0.xyz));

3. 视线方向计算：WorldSpaceViewDir
    // 计算视线方向
    float3 viewDir = normalize(WorldSpaceViewDir(worldPos));
    // 计算菲涅尔强度
    float fresnel = pow(1.0 - saturate(dot(worldNormal, viewDir)), _FresnelPower);

4. 世界空间→裁剪空间：UnityWorldToClipPos
    // 顶点着色器输出
    o.pos = UnityWorldToClipPos(worldPos);

5. 切线空间转换：Unity_WorldToTangent
    // 构造切线空间矩阵
    float3x3 worldToTangent = float3x3(
        v.tangent.xyz,
        v.bitangent.xyz,
        v.normal
    );
    // 将世界空间向量转换到切线空间
    float3 tangentViewDir = mul(worldToTangent, viewDir);

6. 屏幕空间坐标：ComputeScreenPos
    // 计算屏幕UV
    float4 screenPos = ComputeScreenPos(o.pos);
    float2 screenUV = screenPos.xy / screenPos.w;
    // 采样屏幕纹理
    float3 screenColor = tex2D(_ScreenTexture, screenUV).rgb;

7. 视差偏移计算：ParallaxOffset
    // 切线空间视线方向
    float3 tangentViewDir = normalize(viewDirTS);
    // 计算UV偏移
    float2 parallaxOffset = ParallaxOffset(height, _ParallaxScale, tangentViewDir);
    uv += parallaxOffset;

8. 空间转换宏（URP/HDRP）
    TransformObjectToWorld
    模型空间→世界空间。
    TransformWorldToView
    世界空间→摄像机空间。
    TransformWViewToHClip
    摄像机空间→裁剪空间。

---
#### mat4
1. Matrix4x4 的结构
一个 Matrix4x4 包含16个浮点数，排列如下：
| m00 m01 m02 m03 |
| m10 m11 m12 m13 |
| m20 m21 m22 m23 |
| m30 m31 m32 m33 |
    列主序：数据按列存储，即：
        第1列：m00, m10, m20, m30
        第2列：m01, m11, m21, m31
        第3列：m02, m12, m22, m32
        第4列：m03, m13, m23, m33
    行主序：如果按行存储，顺序会不同，但Unity使用的是列主序。
