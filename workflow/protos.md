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

#### ChaincodeSpec

这个结构包含了 chaincode 的具体内容，比如，使用 CLI 或者 REST 接口与 NVP 或者 VP 进行交互时，传给 NVP 或者 VP 的就是 ChaincodeSpec 消息，NVP 或者 VP 收到该消息后，根据传进该消息时的请求类别(invoke、deploy)，将 ChaincodeSpec 消息封装成相应的 ChaincodeDeploymentSpec 或者 ChaincodeInvocationSpec 消息；然后再次将处理后的消息进行封装成 Transaction 消息。

```
message ChaincodeSpec {

    enum Type {
        UNDEFINED = 0;
        GOLANG = 1;
        NODE = 2;
        CAR = 3;
        JAVA = 4;
    }

    Type type = 1;
    ChaincodeID chaincodeID = 2;
    ChaincodeInput ctorMsg = 3;
    int32 timeout = 4;
    string secureContext = 5;
    ConfidentialityLevel confidentialityLevel = 6;
    bytes metadata = 7;
    repeated string attributes = 8;
}
```

#### ChaincodeDeploymentSpec

该结构具体定义了 chaincode deploy 的内容

TODO：定义 codePackage

```
message ChaincodeDeploymentSpec {

    enum ExecutionEnvironment {
        DOCKER = 0;
        SYSTEM = 1;
    }

    ChaincodeSpec chaincodeSpec = 1;
    // Controls when the chaincode becomes executable.
    google.protobuf.Timestamp effectiveDate = 2;
    bytes codePackage = 3;
    ExecutionEnvironment execEnv=  4;

}
```

#### ChaincodeInvocationSpec

```
message ChaincodeInvocationSpec {

    ChaincodeSpec chaincodeSpec = 1;
    // This field can contain a user-specified ID generation algorithm
    // If supplied, this will be used to generate a ID
    // If not supplied (left empty), sha256base64 will be used
    // The algorithm consists of two parts:
    //  1, a hash function
    //  2, a decoding used to decode user (string) input to bytes
    // Currently, SHA256 with BASE64 is supported (e.g. idGenerationAlg='sha256base64')
    string idGenerationAlg = 2;
}
```

#### Transaction

Transaction 定义了对某一个合约的函数调用，其中 chaincodeID 是 ChaincodeSpec 中的 chaincodeID 通过 protobuf 协议序列号化过后的字节序列
```
message Transaction {
    enum Type {
        UNDEFINED = 0;
        // deploy a chaincode to the network and call `Init` function
        CHAINCODE_DEPLOY = 1;
        // call a chaincode `Invoke` function as a transaction
        CHAINCODE_INVOKE = 2;
        // call a chaincode `query` function
        CHAINCODE_QUERY = 3;
        // terminate a chaincode; not implemented yet
        CHAINCODE_TERMINATE = 4;
    }
    Type type = 1;
    //store ChaincodeID as bytes so its encrypted value can be stored
    bytes chaincodeID = 2;
    bytes payload = 3;
    bytes metadata = 4;
    string txid = 5;
    google.protobuf.Timestamp timestamp = 6;

    ConfidentialityLevel confidentialityLevel = 7;
    string confidentialityProtocolVersion = 8;
    bytes nonce = 9;

    bytes toValidators = 10;
    bytes cert = 11;
    bytes signature = 12;
}
```