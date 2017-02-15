m阶B树：

外部节点的深度统一相等，所有叶节点的深度统一相等

内部节点各有：
   不超过 m -1 个Key  K1,K2,...,Kn  不小于⌈m/2⌉-1
   不超过 m 个分支      A0,A1,...,An  (n+1)个
分支比Key多1

内部节点的分支数 n +1 也不能太少：
   树根： 2 <= n +1
   其余：⌈m/2⌉ <= n+1 // 向上取整

   ⌈m/2⌉ <= 分支数 <= m，亦称为 (⌈m/2⌉, m)树
   根：2 <= 分支数 <= m

插入时分裂:

  分裂前节点的key个数为 m，K0，。。。，Km-1
  取中位数s = ⌊m/2⌋，以关键码Ks为界划分
  K0,...,Ks-1, Ks, Ks+1,...,Km-1

  Ks上升一层，以所得的两个节点作为左、右孩子
  分裂后，左右孩子的key个数仍满足m阶b树的条件

删除时旋转分裂：

  删除操作都转换为对叶节点的删除。如果不是叶节点，则查找其后继（右孩子一直往左）交换
  如果被删除后节点 v 下溢，则分三种情况处理：
  1. v的左兄弟的key >= [m/2]，则从v的父节点的key借给v，v的左兄弟的最大key借给v的父节点
  2. v的右兄弟的key >= [m/2]，同1，对称旋转
  3. 从v的父亲中借出key，与v和v的左或右兄弟合并。有可能v的父亲也下溢，则继续合并或旋转

B+树：
  内部节点只存key，不存值。叶子节点包含所有的values
  优点：叶子节点串联，方便范围扫描；节点（内部）更小，对缓存更友好。

  内部节点是子区间

  分裂时：中间元素仍然留在分裂后的右兄弟，key插入到父节点