# BaaP SDK使用说明文档

### 我要开发合约
#### 1. maven工程创建pom.xm配置
配置Maven repo
```xml
<repository>
   <id>pdx-release</id>
   <name>biz.pdxtech</name>
   <url>http://dap.pdx.life:8081/nexus/content/repositories/releases</url>
</repository>
```
配置PDX依赖包`baap-shim`
```xml
<dependency>
    <groupId>biz.pdxtech.baap</groupId>
    <artifactId>baap-shim</artifactId>
    <version>${sdk.version}</version>
</dependency>
```
#### 2. BaaP合约开发说明
开发合约，继承`biz.pdxtech.baap.fabric.shim.ChaincodeBaseX`，重写的方法有
> **合约初始化方法（暂未实现）**
```java
public abstract Response init(ChaincodeStub var1);
```

> **合约执行方法**
```java
public abstract Response invoke(ChaincodeStub var1);
```

#### 3. 开发一个简单的合约项目
**开发合约`MyCc`如下**
```java
public class MyCc extends ChaincodeBaseX {
	private static Logger logger = LoggerFactory.getLogger(MyCc.class);
	@Override
	public Response init(ChaincodeStub stub) {
		return newSuccessResponse();
	}
  
	@Override
	public Response invoke(ChaincodeStub stub) {
		String payload = "";
		try {
			final String function = stub.getFunction();
			final List<String> params = stub.getParameters();
			
			System.out.println("invoke method receive function :" + function);
			System.out.println("invoke method receive params :" + params);
			switch (function) {
				case "getHis":
					QueryResultsIterator<KeyModification> historyForKey = stub.getHistoryForKey(params.get(0));
					historyForKey.forEach(e -> {
												logger.info("tid : {} /key : {} /value : {} /isDelete : {}", e.getTxId(), params.get(0), e.getStringValue(), e.isDeleted());
					});
					historyForKey.close();
					break;
				case "get":
					String state = stub.getStringState(params.get(0));
					logger.info("get key : {} result : {}", params.get(0), state);
					payload = state;
					break;
				case "put":
					stub.putStringState(params.get(0), params.get(1));
					logger.info("put key : {} value : {}", params.get(0), params.get(1));
					break;
				case "del":
					stub.delState(params.get(0));
					logger.info("delete key : {}", params.get(0));
					break;
				case "getRange":
					QueryResultsIterator<KeyValue> stateByRange = stub.getStateByRange(params.get(0), params.get(1));
					stateByRange.forEach(e -> {
						logger.info("key : {} /value : {}", e.getKey(), e.getStringValue());
					});
					stateByRange.close();
					break;
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return newSuccessResponse(payload.getBytes());
	}
	
	private StreamStub stub;
	public static void main(String[] args) throws Exception {
		Arrays.stream(args).forEach(System.out::println);
		MyCc myCc = new MyCc();
		myCc.start(args);
		// if you want the stream function
		// myCc.stub = myCc.stream(new MyCcListener());
	}
	
	//monitor receiving message
	private static class MyCcListener implements StreamListener {
		@Override
		public BaapStream.StatFrame onOpen(String stream, BaapStream.OpenFrame openFrame) {
			return FrameUtil.successStatFrame();
        }
      
		@Override
		public BaapStream.StatFrame onData(StreamStub stub, String stream, BaapStream.DataFrame dataFrame) {
			return null;
		}
		
		@Override
		public void onClose(String stream, BaapStream.CloseFrame closeFrame) {
		}
		
		@Override
		public BaapStream.EchoFrame onEcho(String stream, BaapStream.EchoFrame echoFrame) {
			return null;
		}
		
		@Override
		public void onStat(String stream, BaapStream.StatFrame statFrame) {
			logger.info(String.format("onStat stream:%s, status:%s, reason:%s", stream, statFrame.getStatus(), statFrame.getReason()));
		}
	}
}
```

> 注意：目前chancode暂不支持`invokeChaincode*`、`getStateByPartialCompositeKey` 

### 我要发送交易

#### 1. maven工程创建pom.xm配置
配置Maven repo
```xml
<repository>
   <id>pdx-release</id>
   <name>biz.pdxtech</name>
   <url>http://daap.pdx.life:8081/nexus/content/repositories/releases</url>
</repository>
```
配置PDX依赖包`baap-driver-ethereum`
```xml
<dependency>
    <groupId>biz.pdxtech.baap</groupId>
    <artifactId>baap-driver-ethereum</artifactId>
    <version>${sdk.version}</version>
</dependency>
```
#### 2. BaaP驱动接口说明
`biz.pdxtech.baap.driver.BlockchainDriver`包含以下方法：
> **查询合约方法**
> ```java
> byte[] query(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockChainDriverException;
> ```

> **执行交易方法**
> ```java
> String apply(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockChainDriverException;
> ```

> **获取流方法**
> ```java
> default StreamStub stream(StreamListener listener) throws BlockchainDriverException {
>    return null; 
> }
> ```


#### 3. Baap Stream接口说明
`biz.pdxtech.baap.api.stream.StreamStub`包含以下方法：
> **请求建立一个流通道**  
> 请求参数：
> ​	dst    - 对方地址，规则cc://{chaincode}?pub-key={pub-key}或者cli://{cliId}?pub-key={pub-key}
> ​	extra - 备注
> 响应结果
> ​	通道ID
> ```java
> String open(String dst, byte[] extra) throws BaapStreamException;
> ```

> **发送流数据，最大不能超过1M**  
> 请求参数：
> ​	stream  -   通道ID 
> ​	data    -    数据
> ```java
> void data(String stream, byte[] data) throws BaapStreamException;
> ```

> **关闭指定流通道**
> 请求参数：  
> ​	stream    -   通道ID
> ​	status     -    状态  
> ​	reason    -    原因
> ​	extra      -   备注
> ```java
> void close(String stream, int status, String reason, byte[] extra) throws BaapStreamException;
> ```

> **发送PingPong**
> 请求参数： 
> ​	stream   -   通道ID
> ​	random    - 随机数 
> ```java
> void echo(String stream, int random) throws BaapStreamException;
> ```

> **发送状态**
> 请求参数： 
> ​	stream  - 通道ID
> ​	status   - 状态 
> ​	reason  - 原因  
> ​	extra    - 非必填**  
> ```java
> void stat(String stream, int status, String reason, byte[] extra) throws BaapStreamException;
> ```


#### 4. 开发一个简单的BaaP驱动程序
驱动程序`MyEthereumDriver`如下：
```java
    public static void main(String[] args) throws BlockChainDriverException{
        // init BlockchainDriver
        Properties properties = new Properties();
        properties.setProperty(Constants.BAAP_ENGINE_TYPE_KEY, Constants.BAAP_ENGINE_TYPE_ETHEREUM);
        properties.setProperty(Constants.BAAP_ENGINE_URL_HOST_KEY, "http://127.0.0.1:8545/");
        properties.setProperty(Constants.BAAP_SENDER_PRIVATE_KEY, "510c37d6ed45d8fd179276bfd785b610dac329ee578b245e9f693a1e1bd34065");
        // driver
        BlockchainDriver driver = BlockChainDriverFactory.get(properties);
		// 执行交易
        apply(driver);
        // 查询
        query(driver);
        // 流
        stream(driver);
    }
    
    private static void apply(BlockchainDriver driver) throws BlockchainDriverException {
        Chaincode chaincode = Chaincode.builder()
                .chain("739").name("deploy").version("v1.0")
                .build();
        Transaction tx;
        tx = put("car:1", "99");
        //tx = get("car:1");
        //tx = put("car:2", "97");
        //tx = put("car:3", "98");
        //tx = put("car:4", "95");
        //tx = getRange("car:1", "car:4");
        //tx = put("car:1", "66");
        //tx = getHis("car:1");
        //tx = del("car:1");

        String txid = driver.apply(chaincode, tx);
        System.out.println(txid);
    }

    private static void query(BlockchainDriver driver) throws BlockchainDriverException {
        Chaincode chaincode = Chaincode.builder()
                .chain("739").name("mycc").version("v1.0")
                .build();
        byte[] result = driver.query(chaincode, Transaction.builder().fcn("get").params(new ArrayList<byte[]>() {{
            add("car:1".getBytes());
        }}).build());
        System.out.println(new String(result));
    }

    private static Transaction put(String key, String value) {
        return Transaction.builder()
                .fcn("put").params(new ArrayList<byte[]>() {{
                    add(key.getBytes());
                    add(value.getBytes());
                }})
                .build();
    }

    private static Transaction del(String key) {
        return Transaction.builder()
                .fcn("del").params(new ArrayList<byte[]>() {{
                    add(key.getBytes());
                }})
                .build();
    }

    private static Transaction getRange(String keyStart, String keyEnd) {
        return Transaction.builder()
                .fcn("getRange").params(new ArrayList<byte[]>() {{
                    add(keyStart.getBytes());
                    add(keyEnd.getBytes());
                }})
                .build();
    }

    private static Transaction get(String key) {
        return Transaction.builder()
                .fcn("get").params(new ArrayList<byte[]>() {{
                    add(key.getBytes());
                }})
                .build();
    }

    private static Transaction getHis(String key) {
        return Transaction.builder()
                .fcn("getHis").params(new ArrayList<byte[]>() {{
                    add(key.getBytes());
                }})
                .build();
    }

    private static void stream(BlockchainDriver driver) throws Exception {
        StreamStub streamStub = driver.stream(new RejectedStreamListener());
        String stream = streamStub.open(dst, null);
        streamStub.data(stream, "Hello PDX".getBytes());
        streamStub.close(stream, Constants.BAAP_STAT_SUCCESS, "Normal", null);
    }
```

**注意项：**
* *1 我们推荐一个项目里实例化一个BaaP Driver用于BaaP平台交互*
* *2 BaaP Driver实例化属性说明，见下：*
   * `baap-stack-type` 协议栈类型
   * `baap-stack-url` 协议栈地址
   * `baap-private-key` 用户私钥

### 我要发布

#### 1. maven工程创建pom.xm配置

配置Maven repo

```xml
<repository>
   <id>pdx-release</id>
   <name>biz.pdxtech</name>
   <url>http://daap.pdx.life:8081/nexus/content/repositories/releases</url>
</repository>
```

配置PDX依赖包`baap-driver-ethereum`

```xml
<dependency>
    <groupId>biz.pdxtech.baap</groupId>
    <artifactId>baap-util</artifactId>
    <version>${sdk.version}</version>
</dependency>
```



#### 2. PDX提供了简单的工具类来操作合约发布、停止、运行、下线操作。

但请注意，BaaP平台的合约属于懒启动模式，意味着发布完后，你可以自行启动或者当合约交易执行时平台自行启动。

参见`EthereumDeploy.class`和以下示例。

#### 发布

```java
private static void deploy(BlockchainDriver driver, String chaincode, String version, String jarPath) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		
		String txid = EthereumDeploy.builder().alias("北京车辆管理").desc("北京车辆管理").chaincode(cc)
				.driver(driver).file(new File(jarPath))
				.build().deploy();
		System.out.println(txid);
	}
```



#### 停止

```java
private static void start(BlockchainDriver driver, String chaincode, String version) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		String txId = EthereumDeploy.builder().driver(driver).chaincode(cc).build().start();
		System.out.println(txId);
	}
```



#### 运行

```java
private static void stop(BlockchainDriver driver, String chaincode, String version) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		String txId = EthereumDeploy.builder().driver(driver).chaincode(cc).build().stop();
		System.out.println(txId);
	}
```



#### 下线

```java
private static void withdraw(BlockchainDriver driver, String chaincode, String version) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		String txId = EthereumDeploy.builder().driver(driver).chaincode(cc).build().withdraw();
		System.out.println(txId);
	}
```



