blender/bpy/  
UE PCG  
随机地图  
https://www.bilibili.com/video/BV12i421r7NT?spm_id_from=333.788.videopod.sections&vd_source=81e670e693eb7561bf9efa7f8f86bb48  
https://www.bilibili.com/video/BV1Lr4y1X7JR/?spm_id_from=333.337.search-card.all.click&vd_source=81e670e693eb7561bf9efa7f8f86bb48  
https://www.youtube.com/c/SebastianLague  

---
#### 噪声知识：
https://github.com/Auburn/FastNoiseLite
https://bio-spring.top/post/2025/01/04/perlin-noise/
https://huailiang.github.io/blog/2021/noise/

---
#### H4随机地图细节：  
1. UML：Zone，Hex，Tile，Config (图)  
2. 高频算法：寻路，元胞生成，坐标转换  
3. 重构过程：数据依赖梳理，调试，随机种子复现逻辑，util，业务扩展性，delegate，导表配置模板复用，可视化编辑器  

- 1. Zone是数据容器，里面关联了对应的Hex，Zone最主要的数据是ID，用于顶层玩法的规划(与角色阵营相关)
  - Zone可配置数据有ID，控制Zone尺寸的HexSize，锚定Zone的中心HexPos, Zone内Hex联通环路的概率CircleRate  
      
- 2. Hex是地图形状的基础构成单元，里面关联了Tile单元，Hex集合构成了Zone的形状，进而构成了整个地图的轮廓大体，此外还有空间换算辅助(候补放置点)，道具组寻路辅助(道具组的全连通)等作用    
  - Hex的主要数据段为Pos，可配置数据段为控制Hex尺寸的TileSize(多少个Tile)  
  - 将Hex视为图的节点，则Zone是一个子图，整个地图是由Zone子图构成的  
  - 地图是个无向图，Zone子图是连通图(任意两点之间可互达)  
  - Zone子图之间不一定是连通的，按配置信息Config设置联通  
  - 为了保证Zone内全连通，Zone之间按需求联通，实现做法是：  
  > 1. 先按配置将Zone的中心HexPos进行联通 [作为后续步骤的基础数据，拓展玩法如多层地图则在此数据下对下述步骤进行不同层次数据的执行]  
  > 2. 可以对联通路径加噪声进行扰动，然后由各自的起点沿着连线用元胞生成的方式，在Zone的HexSize限制下交替的生成Hex的集合
  > 3. HexSize沿着已有Zone的集合外围随机扩充一个Hex生成，算法结束则Zone是个无环联通图，两个Zone相互接壤  
  > 4. 根据配置按对应Zone的CircleRate对区域内的无环联通图进行加环  

- 3. Tile图生成  
  - Tile的主要属性有Pos，归属Hex，归属Zone(Hex可能归属多个Zone混合情况)，状态(状态测试表[norm, blk, prot/road， visit, unit])  
  - Tile阶段主要任务两个：初始化Tile的基础基础轮廓和State[norm, blk, prot/road]， 根据Hex信息和Tile实际轮廓统计出一些放置锚点(Hex顶点，边缘中点，中心点等的范围搜索后的有效点)
        
  - 初始化轮廓和基础State的实现：  
  > 1. 按配置的TileSize初始TileMap，并根据生成的Hex图初始化有效的Tile集合(陆地)  
  > 2. 统计Tile图的外轮廓，随机增加或腐蚀或增加，产生不规则表现  
  > 3. 根据Hex联通关系，寻路出Tile通路标记prot
  > 4. Tile统计Hex边界内轮廓，随机腐蚀增厚，并标记blk(状态测试表规定避开prot)  
  > 5. 基础的norm和blk在这个阶段初始化完后面不再出现且逻辑简单直接blk覆盖norm，剩下的则可能不定次序出现即一个Tile进行State测试时旧State可能是其余任意状态
  > 6. 测试的时候可能是按单个tile进行，也可能是按某个Tile集合(unit覆盖多个Tile)并按all条件进行测试
  > 7. 测试通过后会存储到TileMap，可以直接覆盖，但如果按位并存则可以保存更多数据用于拓展规则数据依赖的支持
  > 
  >   ||norm|blk|prot/road|visit|unit|
  >   |---|---|---|---|---|---|
  >   |norm||||||
  >   |blk||||||
  >   |prot/road|🆗||🆗|🆗||
  >   |visit|🆗||🆗|||
  >   |unit|🆗|🆗|||🆗|
      
  - 放置锚点计算：  
  > 1. 用Hex的空间转化公式(见Hex基础换算相关)转为世界坐标再转为Tile坐标，Tile按范围搜索区域内随机可用点作为候补锚点集  
  > 2. 搜索是因为Hex换算后的点往往状态被设置为不可放置的状态  

- 4. 功能单元放置  
  > - 上述流程之后会得到带有基础state(norm, blk, prot)的Tile，Hex一些放置候补锚点  
  > - 按照配置权重和单元刷出公式(配置刷单元批次，对应批次参考权重，区域大小HexSize，TileSize决定单元组数和各单元组权重，再根据地形限制，单元候选列表)，迭代刷出任意个道具作为一个组  
  > - 按一定规则决定是否道具组能够放置，道具组覆盖范围的Tile测试是否全都通过，道具组是否与Hex中心能够联通(放置的过程会改变地图的State可能出现unit堵死旧道具组的情况，以及新的组被旧组影响刷在死胡同，为了避免则让每个道具组到保证在Hex内全连通)  
  > - 此外可能还有一些其他业务规则的拓展限制，全部通过则更新道具组的State和保证联通的通路prot状态到TileMap，测试并覆盖完所有单元组  
  > - 一些特殊业务比如主城，地洞，传送门，船坞，会优先放置，然后在prot中寻路得到road标记上去，再进行通用单元组放置逻辑[Mine，Edge，Empty]  
  > - 刷单元过程由于是数值估算和公式配置的，可能存在数值导致的失败情况，采用尝试上限设定进行抛弃或者转化来保底  
  > - 道具组放置的Hex中心联通检测保证了Zone内道具组都可达，但又缩小了寻路规模限制在Hex内，再由Hex的联通拓展到Zone，Hex的辅助分区作用用于性能优化思路  
 
- 5. 最后一个是根据blk范围和阻挡资源：
  - 按地形类型进行阻挡资源的匹配，使得生成的阻挡过渡自然样式丰富(地貌过度问题)  
      
- 6. Config全局配置内容：  
  > 轮廓腐蚀度
    > Zone联通关系
    > 刷宝策略
    > 多层配置  
      
- 实现方案尚存在问题：
  > Zone联通关系成对生成，选中的Zone对的时序导致的一些不合理，先发育的会先挤占空间，后发育的会先发育好的Zone驱赶导致可能出现大环路  
  > 复杂的Zone连通关系(一个区域有多个联通)或者Zone的HexSize配置可能导致生成失败，比如某个联通Zone较后被选中，且因为被先选中的邻接Zone先发育挤占了自己的发育空间导致发育失败(极端就是没发育就被包围了)
  > 上述两个问题可能的改进是1.用维诺图而不是规整六边形(带来空间换算压力)，2.每个Zone与其余Zone根据size自适应的控制各自的分布位置的算法
  > 地貌阻挡过度问题
  > 超大规模地图生成问题...
