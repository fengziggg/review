### 类型：
#### 前缀和：
- 离散间距填空：统计当前的总和，对每个总和值的位置打个标记，同时到达这个总和的时候往回找一个确定间隔-k是否被打上了标记

#### 滑动窗口：
- C(2,n)问题
- 如果这个2是两个端点，求的是这段区间内的一个总和类型的目标比如累加，则可能有加速结构
- 如果这个2是这段区间内的任意全排量的两个点的情况的问题，则可能没有加速结构，但可以小规模(窗口k)排序辅助

#### 最大/小K个：
- 堆反向使用
- 小根堆过滤一遍后，堆里面的剩下的是这个队列里面最大的k个(小的全被吐出去了，只有顺位前k的才能一直留下来不会被洗牌)

#### 目标分离的二分迭代：
- 普通二分target_val就是key本身，用target_val进行比较就能驱动key移动缩小范围
- 变种的二分target_val跟key可能是不一样的，key二分之后要算最新的val跟target_val比较，但是驱动key移动

---
🟡适合作为基础练手/一般是基础且经典数据结构和算法
🔽水题/随时做得出来的难度
🟩有技巧点/没整理过可能会被卡住
🟥难度toMyself

---
#### 树：
- 🔽[043.往完全二叉树添加节点](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20043.%20%E5%BE%80%E5%AE%8C%E5%85%A8%E4%BA%8C%E5%8F%89%E6%A0%91%E6%B7%BB%E5%8A%A0%E8%8A%82%E7%82%B9/README.md)
  - 练手
- 🔽[044. 二叉树每层的最大值](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20044.%20%E4%BA%8C%E5%8F%89%E6%A0%91%E6%AF%8F%E5%B1%82%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC/README.md#%E5%89%91%E6%8C%87-offer-ii-044-%E4%BA%8C%E5%8F%89%E6%A0%91%E6%AF%8F%E5%B1%82%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC)
  - 层序遍历练手
- 🔽[045. 二叉树最底层最左边的值](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20045.%20%E4%BA%8C%E5%8F%89%E6%A0%91%E6%9C%80%E5%BA%95%E5%B1%82%E6%9C%80%E5%B7%A6%E8%BE%B9%E7%9A%84%E5%80%BC/README.md)
- 🔽[046. 二叉树的右侧视图](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20046.%20%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E5%8F%B3%E4%BE%A7%E8%A7%86%E5%9B%BE/README.md)
  - 1.层序遍历
    
  - 2.深度遍历，记录层深度并记录每层的第一个
  - 一定要遍历全树，没法同层后面的结点剪枝，因为同层后面的结点可能继续往下拓展得到新层的第一个结点

- 🟩[047. 二叉树剪枝](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20047.%20%E4%BA%8C%E5%8F%89%E6%A0%91%E5%89%AA%E6%9E%9D/README.md)
  - 后序遍历，两个操作：左右子节点谁是ret0的砍对应子结点(null也算0)，左&右&自己都为0的则告诉父节点坎自己，否则都不砍

- 🔽[048. 序列化与反序列化二叉树](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20048.%20%E5%BA%8F%E5%88%97%E5%8C%96%E4%B8%8E%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E4%BA%8C%E5%8F%89%E6%A0%91/README.md)
  - 从数据构建树
  - 1.中序遍历+前/后序遍历要存两份结点树数据
  - 2.满二叉树法要村一份，但是多了很多展位标记，取决于树的稀疏程度

- 🔽[049. 从根节点到叶节点的路径数字之和]()
  - 简单思路用个栈，栈顶记录是根节点到当前结点的总和(从根节点到叶子)，然后不断用栈顶的值累加
  - 题解思路是后续遍历，关注每个子树的返回值(从子树往根，没啥特别的)

- 🟩[050. 向下的路径节点之和]()
  - 离散间距填空+前缀和问题([前缀和](.#前缀和))
  - 用在树上的变体，每个路径就是一个前缀和，回退子节点的时候就清理已有的总和记录就行(总和只是为了在一个位置打标记，相当于收回标记)

- 🟥[051. 节点之和最大的路径](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20051.%20%E8%8A%82%E7%82%B9%E4%B9%8B%E5%92%8C%E6%9C%80%E5%A4%A7%E7%9A%84%E8%B7%AF%E5%BE%84/README.md)
  - 求的可能是一个竖向路径(起点终点未定)，也可能是横向路径，每个结点要在三个分支(1父2子)中选两个，选了父就是代表横连就要舍弃1子
  - 竖向路径的最上端结点可能不是树根结点，所有要断头，用一个全局的ans收集最大值来实现
  - 竖向的最下端可能不是叶子，要断尾，在往上反馈的时候可能舍弃两个子节点来实现
  - 横向的可以靠累加左右两个不断头的最大路径(只断尾)来得到，所以还要返回不断头的竖向路径
  - 每个结点坐三件事情:1.两个子都测试是否舍弃(断尾)并返回连接头的断尾值给上一层测试是否横连，2.统计断头最大值，3.然后还有横向衔接的情况与断头最大值比较
  - 这道题是杂糅了竖向最值dp和横向dp，并且比较条件是两种dp的结果用来比比较大小
  - ![image](https://github.com/user-attachments/assets/4d288dc6-c31c-4d05-9853-2cd542bb2fbf)

- 🟩[052. 展平二叉搜索树](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20052.%20%E5%B1%95%E5%B9%B3%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91/README.md)
  - 🔽1.简单的中序遍历用栈辅助
  - 🟩2.练手中序后继化，in遍历的时候多附带一个结点(前序)，ret是每个子树的尾巴，handle是吧左子树的ret连接上自己，吧自己作为右子树的in前序(左子树的前序就null)
  - 一遍完成中序后继化

- 🟩[053. 二叉搜索树中的中序后继](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20053.%20%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E4%B8%AD%E7%9A%84%E4%B8%AD%E5%BA%8F%E5%90%8E%E7%BB%A7/README.md)
  - 在一个二叉搜索树中找后继，两个点
  - 1.左移的时候后继结点前进，右移时候后继不动，差别处理
  - 2.**技巧**:对于找到的情况不是停止，而是当作没找到且是归入继续往右找
  - > 一个是解决全是右结点的情况下后继完全不动，一个是当找到的时候后继其实还可能继续逼近目标(目标结点本身有子树，子树有更靠近的左节点)
    > 找到如果继续归入往左的情况就是找前继

- 🔽[054. 所有大于等于节点的值之和](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20054.%20%E6%89%80%E6%9C%89%E5%A4%A7%E4%BA%8E%E7%AD%89%E4%BA%8E%E8%8A%82%E7%82%B9%E7%9A%84%E5%80%BC%E4%B9%8B%E5%92%8C/README.md)
  - 是二叉搜索树，结点的值从左到右有顺序约束，所以只是个后序遍历

- 🟩[055. 二叉搜索树迭代器](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20055.%20%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E8%BF%AD%E4%BB%A3%E5%99%A8/README.md)
  - 核心是练手非递归遍历
  - 前序遍历：处理结点，然后入栈右结点，再入栈左节点(**右结点顶替本结点**)，退栈的时候就不会有问题
  - > 如果存的是本届点而不是右节点，回退回去会有问题，比如要回退两次才能用父节点的右子树入栈
  - 中序遍历：入栈直到最左端结点，处理结点后出栈本结点，入栈右节点已经右节点往下直到最左，**如果没有右子树就是表现为出栈本节点而已**，会继续退栈处理上一级本节点

- 🔽[056. 二叉搜索树中两个节点之和](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20056.%20%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%E4%B8%AD%E4%B8%A4%E4%B8%AA%E8%8A%82%E7%82%B9%E4%B9%8B%E5%92%8C/README.md)
  - 粗暴的可以用栈存，再从有序表中左右同时出发判定目标加和
  - 也可以用哈希表标记已经找过的值，然后每次都跟跟目标值运算出插值后直接跟已有的标记对比

- 🟩[057. 值和下标之差都在给定的范围内](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20057.%20%E5%80%BC%E5%92%8C%E4%B8%8B%E6%A0%87%E4%B9%8B%E5%B7%AE%E9%83%BD%E5%9C%A8%E7%BB%99%E5%AE%9A%E7%9A%84%E8%8C%83%E5%9B%B4%E5%86%85/README.md)
  - [滑动窗口类型](.#滑动窗口)，限定一个x范围内y的波动范围，没有办法用滑动窗口的合并情况来加速，但也可以用小型有序结构辅助
  - 对于k的窗口用个辅助排序树排序，然后不断的加一个元素和减一个元素，有序结构加速判断是否有符合k区间的差值在t范围内的

- 🟩[058. 日程表](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20058.%20%E6%97%A5%E7%A8%8B%E8%A1%A8/README.md)
  - 线段树？？
  - 左括号和右括号的关联结构
  - 非法情况： [ 【 )，以及【 [ ）

- 🟩[059. 数据流的第 K 大数值](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20059.%20%E6%95%B0%E6%8D%AE%E6%B5%81%E7%9A%84%E7%AC%AC%20K%20%E5%A4%A7%E6%95%B0%E5%80%BC/README.md)
- 🟩[060. 出现频率最高的 k 个数字](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20060.%20%E5%87%BA%E7%8E%B0%E9%A2%91%E7%8E%87%E6%9C%80%E9%AB%98%E7%9A%84%20k%20%E4%B8%AA%E6%95%B0%E5%AD%97/README.md)
- 🟩[061. 和最小的 k 个数对](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20061.%20%E5%92%8C%E6%9C%80%E5%B0%8F%E7%9A%84%20k%20%E4%B8%AA%E6%95%B0%E5%AF%B9/README.md)
  - [都是堆的反向使用问题](.#最大/小K个)
  - 61的k个数对，不是归并的时候队头吐完就不能用了，直接可以交叉匹配的，简单的所有组合方式中的前k..

- 🔽[062. 实现前缀树](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20062.%20%E5%AE%9E%E7%8E%B0%E5%89%8D%E7%BC%80%E6%A0%91/README.md)
- 🔽[063. 替换单词](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20063.%20%E6%9B%BF%E6%8D%A2%E5%8D%95%E8%AF%8D/README.md)
- 🟩[064. 神奇的字典](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20064.%20%E7%A5%9E%E5%A5%87%E7%9A%84%E5%AD%97%E5%85%B8/README.md)
- 🔽[066. 单词之和](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20066.%20%E5%8D%95%E8%AF%8D%E4%B9%8B%E5%92%8C/README.md)
  - 前缀树能应用的场景，也可以用吧前缀的所有情况冗余展开作为hashkey进行hash来做
  - 64就是用冗余展开然后对每个展开的情况设置一个占位符*，来进行匹配
  - 不这样查找字典树，就是遇到第一个不匹配下一次就要用队列存所有子节点的开头(多叉分开了)，如果容忍度是n那就可能有更多分叉的情况，然后统计不匹配超过容忍度的就剪枝
  - 66如果用的是冗余展开的方式，遇到同key的要对key上的所有冗余展开情况都重置一下val(减去差值的形式)
    
- 🟩[065. 最短的单词编码](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20065.%20%E6%9C%80%E7%9F%AD%E7%9A%84%E5%8D%95%E8%AF%8D%E7%BC%96%E7%A0%81/README.md)
  - 后缀树，对字符串进行反序再进行构建的前缀树
  - 这里题意有一点，复用的字符不可能夸#的，所有只可能abcd，cd，xxabcd，对于abcdef不能复用的

- 🟩[067. 最大的异或](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20067.%20%E6%9C%80%E5%A4%A7%E7%9A%84%E5%BC%82%E6%88%96/README.md)
  - 数字按位的前缀树，自己是处理为高尾存储与叶子(没有冗余的0)，题解是高位存在根节点(好处是所有路径长度一致)
  - 创树是n，创完暴力排列组合就是n2
  - 加速的点可以用树的01有序性，也是要遍历n个节点，只是每个节点有目的的反向找存在路径与否并求异或值
  - > 如果想用最长路径，然后定向匹配另一个反向分支，没法o(1)找到最优分支的，而且前面构建树已经是o(n)不要被这个思路误导

---
### 二分：
- 🟡[068. 查找插入位置](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20068.%20%E6%9F%A5%E6%89%BE%E6%8F%92%E5%85%A5%E4%BD%8D%E7%BD%AE/README.md)
  - 二分查找：基础练手

- 🔽[069. 山峰数组的顶部](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20069.%20%E5%B1%B1%E5%B3%B0%E6%95%B0%E7%BB%84%E7%9A%84%E9%A1%B6%E9%83%A8/README.md)
  - 无脑二分查找，都不用去判断什么左右，二分查找的鲁棒性会让他mid命中山峰后继续迭代最后停在山峰（条件只需要l>r）
  - 注意二分的条件，右界左移不用-1，左界右移需要+1

- 🟩[070. 排序数组中只出现一次的数字](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20070.%20%E6%8E%92%E5%BA%8F%E6%95%B0%E7%BB%84%E4%B8%AD%E5%8F%AA%E5%87%BA%E7%8E%B0%E4%B8%80%E6%AC%A1%E7%9A%84%E6%95%B0%E5%AD%97/README.md)
  - 题意条件是除了一个是单个元素，其余都是成对的，且有序排列
  - (aa)*(bb)或者(aa)(bb)*(cc)阵型
  - (aa)*(bb)类型的只要 *不在中间，如果在左边会挤左边的(aa)对到中间(mid-1,mid)相同，如果 * 在右边则(mid-1,mid)是a)(b
  - (aa)(bb)*(cc)类型的判断 * 的条件刚好相反
  - 🟥至于细分(费解的技巧)，自己实现的时候应该可以成对的偏移？？
  - ![image](https://github.com/user-attachments/assets/601b1836-a588-43a2-a0d1-5186e91cfeef)

- 🔽[071. 按权重生成随机数](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20071.%20%E6%8C%89%E6%9D%83%E9%87%8D%E7%94%9F%E6%88%90%E9%9A%8F%E6%9C%BA%E6%95%B0/README.md)
  - 用随机库函数模拟的没有什么
  - 但面试考到了模拟随机发牌相关...

- 🟩[072. 求平方根](https://github.com/doocs/leetcode/blob/main/lcof2/%E5%89%91%E6%8C%87%20Offer%20II%20072.%20%E6%B1%82%E5%B9%B3%E6%96%B9%E6%A0%B9/README.md)
  - 二分快速判定(0-target)内的x，用x^2与target进行比较，驱动的是x
- 🟩[073. 狒狒吃香蕉]()
  - 每个x对应一个y高度
  - 比较key是一个y%n的n范围，初始值取上界maxY
  - 比较函数是每个floor(y%n)的累加 sum(floor(Y(i)%n))与target_val
  - 这种是需要key的单调性与func(key, target_val)的单调性一致才行

- []()
  - 

- []()
  - 

- []()
  - 

- []()
  - 





