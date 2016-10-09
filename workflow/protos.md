### fabric 消息

#### ChaincodeMessage

ChaincodeMessage 是 VP 与 chaincode container 之间交互的消息，ChaincodeMessage 定义如下：

```
message ChaincodeMessage {

    enum Type {
        UNDEFINED = 0;
        REGISTER = 1;
        REGISTERED = 2;
        INIT = 3;
        READY = 4;
        TRANSACTION = 5;
        COMPLETED = 6;
        ERROR = 7;
        GET_STATE = 8;
        PUT_STATE = 9;
        DEL_STATE = 10;
        INVOKE_CHAINCODE = 11;
        INVOKE_QUERY = 12;
        RESPONSE = 13;
        QUERY = 14;
        QUERY_COMPLETED = 15;
        QUERY_ERROR = 16;
        RANGE_QUERY_STATE = 17;
        RANGE_QUERY_STATE_NEXT = 18;
        RANGE_QUERY_STATE_CLOSE = 19;
        KEEPALIVE = 20;
    }

    Type type = 1;
    google.protobuf.Timestamp timestamp = 2;
    bytes payload = 3;
    string txid = 4;
    ChaincodeSecurityContext securityContext = 5;

    //event emmited by chaincode. Used only with Init or Invoke.
    // This event is then stored (currently)
    //with Block.NonHashData.TransactionResult
    ChaincodeEvent chaincodeEvent = 6;
}
```

其中 `payload` 是调用 chaincode 所需要的函数以及参数字符串数组构成的 `ChaincodeInput` 结构序列化过后的字节数组，注意： 与 `securityContext` 里包含的 `payload` 是相同的。

#### ChaincodeSecurityContext

这个结构维护了我们向 chaincode container shim 发送所需要数据的结构，允许 chaincode 通过 shim 接口来访问。

TODO：考虑移除该消息，然后将 transaction 对象直接传给 shim 并且(或者)允许 chaincode 查询 transaction。

```
message ChaincodeSecurityContext {
    bytes callerCert = 1;
    bytes callerSign = 2;
    bytes payload = 3;
    bytes binding = 4;
    bytes metadata = 5;
    bytes parentMetadata = 6;
    google.protobuf.Timestamp txTimestamp = 7; // transaction timestamp
}
```