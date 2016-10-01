## BucketTree

---

#### 1. 介绍
`BucketTree`是组织`worldstate`的一种数据结构，
#### 2. 数据结构
```
type dataKey struct {              type bucketKey struct {
    bucketKey     *bucketKey           level         int
    compositeKey  []byte               bucketNumber  int
}                                  }
```
其中，`compositeKey = statemgmt.ConstructCompositeKey(chaincodeID,key)`, 目前`fabric`的实现是`chaincodeID + 0x00 + key`,其前提假设是`chaincodeID`中不含`0x00`字节；未来将要做：强制`chaincodeID`不含`0x00`字节或者以`len(chaincodeID) + chaincodeID + key`的方式实现。


