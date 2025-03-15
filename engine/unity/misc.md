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

