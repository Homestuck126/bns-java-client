## Overview of BNS Client

### About the overview of BNS Client

If you want to explore our SDK in more details or modify our SDK, we recommend you go through this document. In this document, we will combine the previous tutorials and settings to guide you through the operation of the BNS Client.

### Prerequisites

- Complete the quickstarts document
- Complete the CMD document
- Complete the Callback document
- Complete the ReceiptDao document
- Complete the other setting documents

### Configuration File

The BNS Client will find the configuration file in `PROP_PATH_LIST`, and load the configuration file.

- [BnsClientSample.java](../src/main/java/com/itrustmachines/sample/BnsClientSample.java)
  
  ```java
  final String configPath = FileUtil.findFile(SAMPLE_PROPERTIES, PROP_PATH_LIST);
  final BnsClientConfig config = BnsClientConfig.load(configPath);
  log.info("BnsClientConfig={}", config);
  ```

### Callback Applications

Initialize the Callback Application. You can implement the code in the Callback. Please refer to [Build the Callback Applications](callback_en.md) for more information.

- [BnsClientSample.java](../src/main/java/com/itrustmachines/sample/BnsClientSample.java)

  ```java
  final BnsClientCallback callback = new CallbackSample();
  ```

For the code of Callback, please refer to [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)  

### ReceiptDao Applications

Initialize the ReceiptDao Application with the sqlite database address, `JDBC_URL`. We provide several methods to save, find, and delete the receipt. Please refer to [Build the ReceiptDao Applications](receiptDao_en.md) for more information.

- [BnsClientSample.java](../src/main/java/com/itrustmachines/sample/BnsClientSample.java)

  ```java
  final BnsClientReceiptDao receiptDao = new ReceiptDaoSample(JDBC_URL);
  ```

- For the code of the ReceiptDao Application, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java).

- For the code of the database manipulation. Please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

### Initialize BNS Client

Initialize the BNS Client with configuration file, callback, and receiptDao. When initializing the BNS Client, the BNS Client will register with the BNS Server with the `register` Callback method and Callback the `registerRequest` and `registerResult`.

- [BnsClientSample.java](../src/main/java/com/itrustmachines/sample/BnsClientSample.java)

  ```java
  final BnsClient bnsClient = BnsClient.init(config, callback, receiptDao);
  ```

- For the code of initialization, please refer to [BnsClient.java](../../bns-client/src/main/java/com/itrustmachines/client/BnsClient.java)

### LedgerInput

- After successfully initializing the BNS Client, the BNS Client will convert `CMD` to JSON data type, `cmdJSON`, and do the ledgerInput. Check [Build the CMD](./cmd_en.md) for more information.

- `ledgerInput` will store `cmdJSON` and other information in `ledgerInputRequest` then call `createLedgerInputByCmd` to Callback the information in `ledgerInputRequest`.

- After building the `ledgerInputRequest`, the BNS Client will send `ledgerInputRequest` to BNS Server, and receive the `ledgerInputResponse` from the BNS Server. Then the BNS Client will call `obtainLedgerInputResponse` Callback methods to Callback the information in `ledgerInputResponse`.

- The BNS Client calls `handleReceipt` to extract the receipt from `ledgerInputResponse` and store in the database.

- The BNS Client calls `handleDoneClearanceOrderList` to handle the `doneClearanceOrder` then find out that the receipt which clearanceOrder is less than `doneClearanceOrder` to verify.

- The BNS Client calls `obtainReceiptEvent` and  `obtainDoneClearanceOrderEvent` Callback methods to send the information in `receipt` and `doneClearanceOrder`. The `receipt` and `doneClearanceOrder` are contained in `ledgerInputResponse`.
  
- For the code of `ledgerInput`, please refer to [BnsClient.java](../../bns-client/src/main/java/com/itrustmachines/client/BnsClient.java)

  1. `ledgerInput(@NonNull final KeyInfo keyInfo, @NonNull final String cmdJson)`

     If there are multiple device using one bns client program, we recommended that you use this method to specify each device `keyInfo` by yourself.

     ```java
     public LedgerInputResponse ledgerInput(@NonNull final KeyInfo keyInfo, @NonNull final String cmdJson) {
        return ledgerInputService.ledgerInput(keyInfo, cmdJson);
     }
     ```
  
  2. `ledgerInput(@NonNull final String cmdJson)`

      The default ledgerInput method uses the address in `keyInfo` as `indexValueKey`. 

     ```java
     public LedgerInputResponse ledgerInput(@NonNull final String cmdJson) {
        return ledgerInputService.ledgerInput(keyInfo, cmdJson);
     }
     ```

### Verify

- Before verifying the receipt, the BNS Client will call `getNeedVerifyReceiptLocatorMap`, a receiptDao method, to find out the receipts that need to be verified. Then BNS Client will request the merkleProof of those receipts from the BNS Server.

- After obtaining the merkleProof, the BNS Client will call the `obtainMerkleProof` Callback to Callback the information in `obtainMerkleProof` then call `verify` method to start verifying the receipt.

- After verifying the BNS Client, the BNS Client will call `getVerifyReceiptResult` Callback methods to Callback the information in `getVerifyReceiptResult`.

----

The User Guide is now complete. If you have any problem about our SDK, please feel free to contact us.

Next Page : [Home](../../README.md)  
Last Page : [Configure the Setting of BNS Client](./other_setting_en.md)
