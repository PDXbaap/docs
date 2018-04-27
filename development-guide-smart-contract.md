#### 1. Maven pom.xml configuration

##### Configuring maven repository
``` xml
<repository>
   <id>pdx-release</id>
   <name>biz.pdxtech</name>
   <url>http://daap.pdx.life:8081/nexus/content/repositories/releases</url>
</repository>
```
##### Configuring PDX dependency
``` xml
<dependency>
    <groupId>biz.pdxtech.baap</groupId>
    <artifactId>baap-shim</artifactId>
    <version>${sdk.version}</version>
</dependency>
```
### 2. Developing BaaP chaincode

Inherit biz.pdxtech.baap.fabric.shim.ChaincodeBaseX & override the following:
``` java
//chaincode initialization, NOT-YET-IMPLEMENTED
public abstract Response init(ChaincodeStub stub);

//execute transaction
public abstract Response invoke(ChaincodeStub stub);
```
### 3. A simple chaincode
``` java
public class MyChaincode extends ChaincodeBaseX {
	private static Logger logger = LoggerFactory.getLogger(MyChaincode.class);
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
				case "his":
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
		MyChaincode myChaincode = new MyChaincode();
		myChaincode.start(args);
		// if you want the stream function
		// myChaincode.stub = myChaincode.stream(new MyStreamListener());
	}
	
	//monitor receiving message
	private static class MyStreamListener implements StreamListener {
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
### Notes:

#### 1) Currently PDX chaincode SDK does not support invokeChaincode* and getStateByPartialCompositeKey

#### 2) By default, on PDX public blockchain, sandboxing type PUBLIC-MOST-RESTRICTED (via X-BAAP-SANDBOX-TYPE metadata) is applied on deployment. With PUBLIC-MOST-RESTRICTED sandboxing, the following restrictions apply:

--- no read/write to local directory

--- non inbound or outbound connection

--- Memory consumption capped at 64MB

--- CPU usage capped at 5%

--- No more than 256 (key,value) pair states, where each (key,value) pair's key+value size is no more than 1KB, and its history is limited to the last 100 updates.
