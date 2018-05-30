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

### 2. Deploy, start, stop,upgrade & remove

1. PDX BaaP provides relevant functions in the PDX Wallet for or provisioning, start, stop, upgrade and de-provisioning of smart contracts. Please assess http://wallet.pdx.link .
2. PDX BaaP also provides a simple class for provisioning, start, stop, upgrade and de-provisioning of smart contracts. You can use it to customize the release function in your system.

Please refer to class `ChaincodeDeploy.class` and the following code excerpt for details:

```java
package biz.pdxtech.baap.guide;

import biz.pdxtech.baap.api.Chaincode;
import biz.pdxtech.baap.api.Constants;
import biz.pdxtech.baap.driver.BlockChainDriverFactory;
import biz.pdxtech.baap.driver.BlockchainDriver;
import biz.pdxtech.baap.driver.BlockchainDriverException;
import biz.pdxtech.baap.driver.ethereum.ChaincodeDeploy;

import java.io.File;
import java.util.Properties;

@SuppressWarnings("SameParameterValue")
public class DeployApplication {
	public static void main(String[] args) throws Exception {
		Properties properties = new Properties();
		properties.setProperty(Constants.BAAP_ENGINE_TYPE_KEY, Constants.BAAP_ENGINE_TYPE_ETHEREUM);
		properties.setProperty(Constants.BAAP_ENGINE_URL_HOST_KEY, "http://127.0.0.1:8545/");
		properties.setProperty(Constants.BAAP_SENDER_PRIVATE_KEY, "25b37deb8b1f2a36cea8d53cfb262440d865e1321e88cad931479809b2e54244");
		
		DeployApplication app = new DeployApplication(properties);
		
		String name = "mycc", version = "1.0", jarPath = "/home/harrison/mycc-1.0.jar";
		
		// chaincode deploy
		app.deploy(name, version, jarPath);
		// chaincode start
		app.start(name, version);
		// chaincode stop
		app.stop(name, version);
		// chaincode upgrade, if upgrade that will stop the old version and start the newest
		app.upgrade(name, version, jarPath);
		// chaincode withdraw, if withdraw, the private data of the chaincode on the nodes will be deleted
		app.withdraw(name, version);
	}
	
	private BlockchainDriver driver;
	
	public DeployApplication(Properties properties) throws BlockchainDriverException {
		// init blockchain driver
		this.driver = BlockChainDriverFactory.get(properties);
	}
	
	private void deploy(String chaincode, String version, String jarPath) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		
		String txid = ChaincodeDeploy.builder().alias("chaincode-name").desc("chaincode-desc").chaincode(cc)
				.driver(driver).file(new File(jarPath))
				.build().deploy();
		System.out.println(txid);
	}
	
	private void start(String chaincode, String version) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		String txId = ChaincodeDeploy.builder().driver(driver).chaincode(cc).build().start();
		System.out.println(txId);
	}
	
	private void upgrade(String chaincode, String version, String jarPath) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		
		String txid = ChaincodeDeploy.builder().alias("chaincode-name").desc("chaincode-desc").chaincode(cc)
				.driver(driver).file(new File(jarPath))
				.build().upgrade();
		System.out.println(txid);
	}
	
	private void stop(String chaincode, String version) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		String txId = ChaincodeDeploy.builder().driver(driver).chaincode(cc).build().stop();
		System.out.println(txId);
	}
	
	private void withdraw(String chaincode, String version) throws Exception {
		Chaincode cc = Chaincode.builder().chain(Constants.BAAP_ENGIEN_ETHEREUM_CHAIN_DEFAULT)
				.name(chaincode).version(version).build();
		String txId = ChaincodeDeploy.builder().driver(driver).chaincode(cc).build().withdraw();
		System.out.println(txId);
	}
}
```

