### 1. Configuring maven pom.xml

Maven repository

```xml
<repository>
   <id>pdx-release</id>
   <name>biz.pdxtech</name>
   <url>http://repo.pdxtech.biz:8081/nexus/content/repositories/releases</url>
</repository>
```

PDX dependency

```xml
<dependency>
    <groupId>biz.pdxtech.baap</groupId>
    <artifactId>baap-driver-ethereum</artifactId>
    <version>${sdk.version}</version>
</dependency>
```

### 2. Deploy, start, stop & remove

PDX BaaP provides a simple class for provisioning, start, stop and de-provisioning of smart contracts. Note that PDX BaaP employs lazy-loading mode on smart contracts. The deployer can manually start it or it'll auto-started on executing first TX. 

Please refer to class `ChaincodeDeploy.class` and the following code excerpt for details:

#### Deploy

```java
private static void deploy(BlockchainDriver driver, String chaincode, String version, String jarPath) throws Exception {
    Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
            .name(chaincode).version(version).build();
    
    String txid = ChaincodeDeploy.builder().alias("chaincode-name").desc("chaincode-desc").chaincode(cc)
            .driver(driver).file(new File(jarPath))
            .build().deploy();
    System.out.println(txid);
}
```



#### Start

```java
private static void start(BlockchainDriver driver, String chaincode, String version) throws Exception {
    Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
            .name(chaincode).version(version).build();
    String txId = ChaincodeDeploy.builder().driver(driver).chaincode(cc).build().start();
    System.out.println(txId);
}
```



#### Stop

```java
private static void stop(BlockchainDriver driver, String chaincode, String version) throws Exception {
    Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
            .name(chaincode).version(version).build();
    String txId = ChaincodeDeploy.builder().driver(driver).chaincode(cc).build().stop();
    System.out.println(txId);
}
```



#### Remove

```java
private static void withdraw(BlockchainDriver driver, String chaincode, String version) throws Exception {
    Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
            .name(chaincode).version(version).build();
    String txId = ChaincodeDeploy.builder().driver(driver).chaincode(cc).build().withdraw();
    System.out.println(txId);
}
```
