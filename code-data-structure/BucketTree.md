## BucketTree

---

#### 1. 介绍
`BucketTree`是组织`world state`的一种数据结构，为了计算`world state`的加密哈希，这种方法建立了一种基于哈希表的桶之上的`merkle Tree`。

作为该方法的核心，属于`world state`的`key-values`数据对假定被存储在由预先决定好数量的桶组成的哈希表中。一个哈希函数(`hashFunction`)被用来决定包含一个指定的`key`所需要桶的数量。

#### 2. 数据结构
```
type dataKey struct {              type bucketKey struct {
    bucketKey     *bucketKey           level         int
    compositeKey  []byte               bucketNumber  int
}                                  }
```
其中，`compositeKey = statemgmt.ConstructCompositeKey(chaincodeID,key)`, 目前`fabric`的实现是`chaincodeID + 0x00 + key`,其前提假设是`chaincodeID`中不含`0x00`字节；未来将要做：强制`chaincodeID`不含`0x00`字节或者以`len(chaincodeID) + chaincodeID + key`的方式实现。


