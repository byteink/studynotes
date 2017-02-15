每个节点最少 a 个孩子，最多有 b 个孩子，b >= 2a
只有内部结点，叶子不叫做结点

定义： p(v)表示结点v的孩子树
      |T|表示树T的叶子个数
      a >= 2 以及 b >= 2a-1
一个树是(a,b)-树，需要满足：
    1. 所有叶子具有相同的深度
    2. 所有的节点v满足 p(v) <= b
    3. 所有除了root以外的节点满足 p(v) >= a
    4. root满足p(r) >= min(2, |T|)


t: sharing threshold
s: how many sons to shift when sharing
当删除操作时，sibling节点的child个数大于 a + t 时才可以借（sharing)
s表示从sibling一次性借多少个
满足：
0 <= t <= b+1 - a
1 <= s <= t + 1
时才能给出一个正确的重平衡算法


