Canvas五个基础组件：
- Canvas：
  > 设置渲染摄像机：屏幕空间overlap，屏幕空间camera(这两个会受视口影响导致ui变动)，世界空间
  > overlap类似UI优先，camera可以23D混合但要指定摄像机，世界空间就是吧ui当作obj(但是其z不是gameObj的线性z)
  > UI的视口和Frustum的视口并存

- Scaler：
  > 图片在屏幕上的比率，constantPx，constantPhy，scaleScreen  
  > 设计的逻辑分辨率RR，ReferencePixelPerUnit(逻辑分辨率下的物理单位与像素比例)，PixelPerUnit(图片资源指定)，DPI  
  > constantPx和constantPhy都是屏幕变化的时候Img尺寸是不会再变的(Scale),换算大致是ImgSize * (RPPU/PPU)，初始话的时候与PPU和RPPU，DPI比率有关  
  > scaleToScreen，自适应：动态调整所有UI缩放，变化的是屏幕分辨ScreenResulution，不变的是参考RefrenceResolution，expand(黑边，按最小的SR/RR)，shrink(截断，按最大的SR/RR),还有Match会变动切换的时候

- reactTransform：
  > Anchor锚框是对**父节点**的相对位置，作为子节点本身的相对变换基准，pivot是每个节点的锚点(cocos的anchor)
  > 当某个维度的anchor是一个点的时候变换逻辑就是与锚点的偏移和这个维度的width，当是锚框anchor range的时候会动态切换为left,和right差值

- EventSys：
  > Input模块输入后对信号的分发，需要依赖StandaloneInput

- StandaloneInput：
  > 用户输入的封装，依赖EventSystem来发信号


scalar的效果：  
  > ![image](https://github.com/user-attachments/assets/76291c73-62ff-44f1-b20e-85639fe167e0)
  > ![image](https://github.com/user-attachments/assets/3927f2eb-b826-4ab0-b5ed-5016f9dba011)
  >
  > ![image](https://github.com/user-attachments/assets/01929124-7acc-476f-b8fc-318e859073c6)
  > ![image](https://github.com/user-attachments/assets/c38a3067-02cf-4451-b040-dd8548bec482)
