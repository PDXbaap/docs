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

> **chaincode init**
>
> ```java
> String init(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockchainDriverException;
> ```
>
> 

> **chaincode query**
> ```java
> byte[] query(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockChainDriverException;
> ```

> **chaincode invoke**
> ```java
> String apply(@Nonnull Chaincode chaincode, @Nonnull Transaction tx) throws BlockChainDriverException;
> ```



### 3. A sample client app

```java
public class ClientApplication {
	public static void main(String[] args) throws Exception {
		Properties properties = new Properties();
		properties.setProperty(Constants.BAAP_ENGINE_TYPE_KEY, Constants.BAAP_ENGINE_TYPE_ETHEREUM);
		properties.setProperty(Constants.BAAP_ENGINE_URL_HOST_KEY, "http://127.0.0.1:8545/");
		properties.setProperty(Constants.BAAP_SENDER_PRIVATE_KEY, "25b37deb8b1f2a36cea8d53cfb262440d865e1321e88cad931479809b2e54244");
		
		ClientApplication app = new ClientApplication(properties);
		String name = "yourcc", version = "1.0";
		// chaincode invoke
		// app.apply(name, version);
		// chaincode query
		// app.query(name, version);
	}
	
	
	private BlockchainDriver driver;
	
	public ClientApplication(Properties properties) throws BlockchainDriverException {
		// init blockchain driver
		this.driver = BlockChainDriverFactory.get(properties);
	}
	
	private void apply(String name, String version) throws BlockchainDriverException {
		Chaincode chaincode = Chaincode.builder()
				.chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT).name(name).version(version)
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
	
	private void query(String name, String version) throws BlockchainDriverException {
		Chaincode chaincode = Chaincode.builder()
				.chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT).name(name).version(version)
				.build();
		Transaction query = Transaction.builder().fcn("get").params(new ArrayList<byte[]>() {{
			add("car:1".getBytes());
		}}).build();
		byte[] result = driver.query(chaincode, query);
		System.out.println(new String(result));
	}
	
	private Transaction put(String key, String value) {
		return Transaction.builder()
				.fcn("put").params(new ArrayList<byte[]>() {{
					add(key.getBytes());
					add(value.getBytes());
				}})
				.build();
	}
	
	private Transaction del(String key) {
		return Transaction.builder()
				.fcn("del").params(new ArrayList<byte[]>() {{
					add(key.getBytes());
				}})
				.build();
	}
	
	private Transaction getRange(String keyStart, String keyEnd) {
		return Transaction.builder()
				.fcn("getRange").params(new ArrayList<byte[]>() {{
					add(keyStart.getBytes());
					add(keyEnd.getBytes());
				}})
				.build();
	}
	
	private Transaction get(String key) {
		return Transaction.builder()
				.fcn("get").params(new ArrayList<byte[]>() {{
					add(key.getBytes());
				}})
				.build();
	}
	
	private Transaction his(String key) {
		return Transaction.builder()
				.fcn("his").params(new ArrayList<byte[]>() {{
					add(key.getBytes());
				}})
				.build();
	}
}
```

**Notes：**
* *One BaaP Driver per project for interaction with PDX BaaP platform*
* *BaaP Driver driver instantiation properties：*
   * `baap-stack-type` type of the blockchain engine
   * `baap-stack-url` address of the blockchain engine
   * `baap-private-key` private key of the client app

