# 结构
每个节点的扇出度数为Θ(m)
每个节点分配一个大小Θ(m)个blocks的buffer，叶节点没有buffer

更新像查询一样会被延迟，以插入为例，不需要找到叶节点的插入位置，
相反收集若干插入（或其他操作）后，组成一个block插入到root的buffer中
当buffer满了，就把buffer推到下一层，这个操作叫做 buffer-emptying process
删除或者其他复杂更新操作也基本跟插入是一样的流程

这意味者在树中会记录多个对同一元素的更新或删除操作，因此需要有个时间戳表示插入到top buffer时的先后
同时查询变得批量化意味着一次查询的结果会因为几次buffer-emptying操作延迟产生

# 细节
buffer tree是一颗(a,b)树，a = m/4，b=m
除了root之外所有节点的扇出度数在 m/4 和 m 之间

更新操作：
   
  构造一个新元素包含：
  1. 需要插入或删除的元素
  2. 一个时间戳 timestamp
  3. 一个操作标记，表明是删除还是插入

  当我们收集了B个这样的元素后，插入到root的buffer中。
  如果root的buffer包含小于m/2个block时，停止；
  否则，清空buffer

我们定义内部节点为children不是叶节点的节点
处理的最基本部分都是在这些节点上

buffer-emptying操作只会在内部节点上递归进行
只有完成了所有内部节点的buffer-emptying处理后，才会清空leaf节点的buffer
buffer-emptying操作可以在线性次I/O操作内完成

内部结点的buffer-emptying操作：
- 加载节点的partitioning元素到内存
- 重复加载最多 m/2 block个buffer到内存中，做如下操作：
 1. 排序内存中buffer里的元素。如果出现对同一个元素的插入和删除，比较时间戳也满足要求，则两个操作抵消
 2. 重排，保证最多只有一个block是不满的
- 如果任何children的buffer包含超过 m/2 个blocks，并且为内部结点，递归地对这些结点应用buffer-emptying操作


清空leaf的buffer稍微复杂些，需要考虑树的重平衡
需要修改删除的重平衡算法因为涉及buffer的问题
在每次rebalance操作之前做一次buffer-emptying处理

内部结点的buffer-emptying处理保持不变性：
如果一个leaf nodes的buffer满了，那么它到root这条路径上的结点的buffer都是空的

因此当重平衡时，





