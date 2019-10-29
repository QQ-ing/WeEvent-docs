## 适配Fabric

`WeEvent`的子模块`Broker`对区块链`Fabric 1.4`的支持。以下安装以`CentOS 7.2`为例。

### 前置条件

- 安装Fabric 1.4区块链节点(以官网Fabric Samples安装为例)

  具体安装步骤，请参见[Fabric安装](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)。

### 修改Fabric相关的配置文件
- 添加安装Fabric节点的证书等文件

  - 配置`WeEvent`访问区块链的方式

    修改`weevent.properties`文件中的配置项`broker.blockchain.type=fisco`，替换`fisco`为`fabric`。

  - 添加证书文件
  
    将安装Fabric安装后`/fabric-sample/first-network/crypto-config/`目录下的所有文件，拷贝到weevent-broker安装目录下的
    
    `/conf/fabric/crypto-config/`里。

  - 区块链节点配置文件fabric.properties

    修改`chain.organizations.user.keyfile`配置项，只需要修改该配置路径的最后文件名(从上一步的`crypto-config`,找到对应的文件名即可)。
    
    修改`chain.peer.address=grpcs://127.0.0.1:7051`配置项，替换`127.0.0.1`为部署Fabric节点的ip
    
    修改`chain.orderer.address=grpcs://127.0.0.1:7050`配置项，替换`127.0.0.1`为部署Fabric节点的ip

- 部署系统合约

  运行脚本`./deploy-fabric-topic-control.sh `部署合约。例如:

  ```shell
$ ./deploy-topic-control.sh deploy
  begin deploy topic and topicController contract.
  begin add topic into topicController contract.
  deploy contract success. 
  ```
  
### 启动服务

  通过`./broker.sh start`命令启动服务，正常启动如下：

  ```shell
  $ ./broker.sh start
  start broker success (PID=89054)
  add the crontab job success
  ```
  
### 代码样例
  
  API里所有的groupId就是Fabric的channelname, 以发布事件为例，代码如下：
  
```java
public class Rest {
    public static void main(String[] args) {
        System.out.println("This is WeEvent restful sample.");
        try {
            SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
            RestTemplate rest = new RestTemplate(requestFactory);
            // ensure topic exist "com.weevent.test"
            Boolean result = rest.getForEntity("http://localhost:8080/weevent/rest/open?topic={}&groupId={}",
                    Boolean.class,
                    "com.weevent.test",
                    "mychannel").getBody();
            System.out.println(result);
            // publish event to topic "com.weevent.test"
            SendResult sendResult = rest.getForEntity("http://localhost:8080/weevent/rest/publish?topic={}&groupId={}&content={}",
                    SendResult.class,
                    "com.weevent.test",
                    "mychannel",
                    "hello weevent".getBytes(StandardCharsets.UTF_8)).getBody();
            System.out.println(sendResult.getStatus());
            System.out.println(sendResult.getEventId());
        } catch (RestClientException e) {
            e.printStackTrace();
        }
    }
}
```
