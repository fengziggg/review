一. 对纹理资源本身的设置texture Importer，不等于Scene中物体应用上去后效果的设置，比如pivot不是ui中的pivot，
但是本身的设置会影响应用上去的结果  
1. TextureType:
  > Normal(改变原始tex?), cookie, lightmap, shadowMask, single ??
  > meshtype??
2. Advance: Mipmap相关
3. WrapMode: 纹理重复截断设置

![image](https://github.com/user-attachments/assets/b9c08f6e-46af-439d-8f3c-9380b993ed05)


二. sprite Editor  
Sprite Importer（资产本身的设置，资产本身轮廓相关，归属类型）：  
- sprite type是不同用途的sprite
- sprite mode是纹理的组成模式，多边形，mutiple(图集)
- 不同的sprite mode有不同的editor操作
- ![image](https://github.com/user-attachments/assets/636ca53f-64b6-45a1-81d7-5ce8cc35d78d)

Component（sprite相关组件对sprite资产的引用）：
- spriteRenderer：DrawMode控制拉伸模式模式，Mask的作用内外
- spriteMask：Mask功能，配合Mask内外设置
- SortGroup
