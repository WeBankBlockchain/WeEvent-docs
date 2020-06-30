## 错误码
### 错误码说明

所有错误以检查异常的形式抛出，通过异常类`BrokerException`的 `getCode()`和`getMessage()`方法获取详细信息。  

### 错误码区间

按错误是发生在客户端还是服务端分为两类。

- 客户端错误码在(100000, 200000)之间，为用户输入错误。一般通过检查输入参数或者接口调用的方式就能解决这类问题。
- 服务端错误码在(200000, 300000)之间，一般是服务环境变化引起的错误。请按照FAQ列表排查，或者联系`WeEvent`技术支持。  

错误码文件，请参见[ErrorCode.java](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-client/src/main/java/com/webank/weevent/client/ErrorCode.java) 。

