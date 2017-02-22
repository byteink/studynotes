写入完成后再更新seq，不然先更新可能被读看到    

level0是MemTable的dump，level0的key范围是有重叠的    
因此compact level0时需要找到所有与之有重叠的level0 文件    

## sst文件格式：    

 <beginning_of_file>    
  [data block 1]    
  [data block 2]    
  ...    
  [data block N]    
  [meta block 1]    
  ...    
  [meta block K]    
  [metaindex block]    
  [index block]    
  [Footer] \(fixed size; starts at file_size - sizeof(Footer))    
  <end_of_file>

### data block
包含多个kv条目和一个block trailer      
每个datablock有个大小限制blockSize，超出开始写新的一个datablock     
- 数据部分：kv entries+restarts（包含restart len）    
- trailer：一字节type（加密类型）+四子节crc    

### metaindex block：
索引meta block     
每个条目的内容是 metablock的名字和对应的blockhandle    

### index block：
索引datablock    
每datablock一个条目，条目的key是datablock的最后一个key，vlaue是datablock的blockhandle    

### footer：
固定长度40字节，包括    
- metaindex block的blockhandle    
- index block的blockhandle    
- padding    
- magic // 8字节小端0xdb4775248b80fb57    

### filter meta block:    
metaindex查找filter.<N>可以filter block的blockhandle    

### stat meta block：    
     统计block    

## Manifest:    
     存储各个层次的sst文件，和对应的key范围    
     每次重新打开数据库时创建（新的文件名），    
