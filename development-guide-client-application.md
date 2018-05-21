### 1. Configuring maven pom.xml

#### Maven repository
```xml
<repository>
   <id>pdx-release</id>
   <name>biz.pdxtech</name>
   <url>http://repo.pdxtech.biz:8081/nexus/content/repositories/releases</url>
</repository>
```
#### PDX dependency
```xml
<dependency>
    <groupId>biz.pdxtech.baap</groupId>
    <artifactId>baap-driver-ethereum</artifactId>
    <version>${sdk.version}</version>
</dependency>
```
### 2. BaaP SDK 

#### `biz.pdxtech.baap.driver.BlockchainDriver`
> **state query**
> ```java
> byte[] query(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockChainDriverException;
> ```

> **tx execution**
> ```java
> String apply(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockChainDriverException;
> ```

> **stream stub**
> ```java
> default StreamStub stream(StreamListener listener) throws BlockchainDriverException ;
> ```

#### `biz.pdxtech.baap.api.stream.StreamStub`

> **create a stream**  

> Parameters：
> ​	dst    - peer address，scheme is cc://{chaincode}?pub-key={pub-key}, or cli://{cliId}?pub-key={pub-key}
> ​	extra - extra data 
> Return:
> ​	stream ID 
> ```java
> String open(String dst, byte[] extra) throws BaapStreamException;
> ```

> **send stream data, maxed at 1MB when PUBLIC-MOST-RESTRICTED sandboxing type is applied.**  

> Parameters：
> ​	stream  -   stream ID 
> ​	data    -   data itself
> ```java
> void data(String stream, byte[] data) throws BaapStreamException;
> ```

> **close a stream** 

> Parameters：  
> ​	stream    -   stream ID
> ​	status     -    status
> ​	reason    -    reason to close
> ​	extra      -   extra data, not guaranteed to arrive
> ```java
> void close(String stream, int status, String reason, byte[] extra) throws BaapStreamException;
> ```

> **send ping-pong message** 

> Parameters： 
> ​	stream   -   stream ID
> ​	random    - random number
> ```java
> void echo(String stream, int random) throws BaapStreamException;
> ```

> **send status message** 

> Parameters： 
> ​	stream  - stream ID
> ​	status   - status
> ​	reason  - reason
> ​	extra    - extra data if necessary  
> ```java
> void stat(String stream, int status, String reason, byte[] extra) throws BaapStreamException;
> ```


### 3. A sample client app

```java
    public class ClientApplication {
    	public static void main(String[] args) throws Exception {
    		// init BlockchainDriver
    		Properties properties = new Properties();
    		properties.setProperty(Constants.BAAP_ENGINE_TYPE_KEY, Constants.BAAP_ENGINE_TYPE_ETHEREUM);
    		properties.setProperty(Constants.BAAP_ENGINE_URL_HOST_KEY, "http://10.0.0.136:8545/");
    		properties.setProperty(Constants.BAAP_SENDER_PRIVATE_KEY, "510c37d6ed45d8fd179276bfd785b610dac329ee578b245e9f693a1e1bd34065");
    		// driver
    		BlockchainDriver driver = BlockChainDriverFactory.get(properties);
    		// execute TX
    		apply(driver);
    		// query state
    		query(driver);
    		// streaming
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
    		//tx = his("car:1");
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
    	
    	private static Transaction his(String key) {
    		return Transaction.builder()
    				.fcn("his").params(new ArrayList<byte[]>() {{
    					add(key.getBytes());
    				}})
    				.build();
    	}
    	
    	private static void stream(BlockchainDriver driver) throws Exception {
    		StreamStub streamStub = driver.stream(new RejectedStreamListener());
    		String stream = streamStub.open(String.format("cc://%s?pub-key=%s", "mycc:v1.0", ""), null);
    		streamStub.data(stream, "Hello PDX".getBytes());
    		streamStub.close(stream, Constants.BAAP_STAT_SUCCESS, "Normal", null);
    	}
    }
```

**Notes：**
* *One BaaP Driver per project for interaction with PDX BaaP platform*
* *BaaP Driver driver instantiation properties：*
   * `baap-stack-type` type of the blockchain engine
   * `baap-stack-url` address of the blockchain engine
   * `baap-private-key` private key of the client app

