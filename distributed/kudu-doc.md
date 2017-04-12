# 基本概念
- MemRowSet    
  内存中的B+树，节点大小 1 K

- DiskRowset    
  分为base 和 delta    
  更新的数据写到delta，定期compact到base    
  base是列式存储，不同的数据类型有不同的编码。    
  使用了b+树对主键进行索引，也使用了bloom filter加速查找。
# Compaction
## 目标
- 重新安排物理布局，使后续操作更加高效
- compaction本身不能消耗太多资源
- 平滑。随着时间的推移，操作的性能要可预测、一定程度上保持恒定不变

## RowSets的成本测量指标
- Width    
  RowSet key范围占tablet key范围的比例   
  width越大，读操作时查询这个RowSet的可能性越大（随机读的情况下）   

- Height    
指定一个key，有多少个RowSet包含这个key

## 不同操作的开销  
### Insert   
插入需要先查询是否有重复的key。     
因为使用区间树存储RowSet的范围，可以很高效地找到区间包含要查入的key的RowSets，复杂度是n的线性复杂度。   

记 n = 给定key的Height    
   B = bloom filter 假阳性的概率     
   C_bf =  检查bloom filter的开销    
   C_pk = 查找主键的开销   
则 Cost = n * C_bf  + n *B * C_pk = n * (C_bf + B * C_pk)

B 一般在 %1 或更低，所以检查bloom filter 这个操作在整个开销中是最大的部分。    
但是在一些情况下，主键列非常大，每次检查主键都会引发磁盘寻道（disk seek），导致C_pk比C_bf高出几个数量级，   
所以我们不能完全忽略bloom filter失误导致的开销。   

### Random read（随机读）
跟insert的开销类似，给出指定的key，每个有潜在的有重叠的RowSet都要查询

### Short Scan
Scan操作无法利用bloom filter，所以开始跟上面的类似，除了所有有重叠的RowSet必须用主键查找   
    Cost = n * C_pk       
short scan的定义：找到起始key后的顺序IO的开销比寻道要小（比如假设寻道10ms，少于1M的顺序IO）        
