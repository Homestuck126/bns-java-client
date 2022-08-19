## BNS Auto Folder Attest

### About BNS Auto Folder Attest

The BNS auto folder attest allows developers to attest every file in a specific folder. This tutorial document will guide you through steps to configure and run the auto folder attest service. 

<!-- no toc -->
1. [Configure setting for Sample Code](#1-configure-the-settings-for-sample-code)
2. [Run the Sample code](#2-run-the-sample-code)
3. [BNS Auto Folder Attest Callback](#3-BNS-Auto-Folder-Attest-Callback)

Complete the tutorial for each step, then you will be ready to develop auto folder attest service with your own applications.

### Prerequisites

- Complete the quickstarts document
- Complete the CMD document
- Complete the Callback document
- Complete the ReceiptDao document
- Complete the other setting documents


### 1. Configure the settings for auto folder attest service

This configuration file is very important. The main program uses this configuration file to initialize the BNS auto folder attest service.
Please follow the instructions and configure the setting.

```Java
# absolute path of root folder to auto attest 
rootFolderPath=

# absolute path of verification proof to download
verificationProofDownloadPath=

# scan delay in seconds
scanDelay=300

# bns-auto-folder-attest service will record the history of attestation. You can set the filename and store path of the csv file 
historyCsvPath=history.csv

# bns-auto-folder-attest  will record the history of file scanning. You can set the filename and store path of the csv file
scanHistoryCsvPath=scan.csv
        
# bns-auto-folder-attest service will record the history of attestation. You can set the filename and store path of the csv file
jdbcUrl=jdbc:sqlite:folder-auto-attest.db
        
## optional ##

# Disable downdload the verification proof. Default setting is false
disableDownloadVerificationProof=false

# Disable cache or not. Default setting is false
disableCache=false
        

#### BNS Client ####
# Please refer to quickstarts documents in BNS Java Client.
bnsServerUrl=
nodeUrl=
email=
verifyBatchSize=10
verifyDelaySec=1
retryDelaySec=5
```

### 2. Run the Sample Code

#### Command Line Interface
Move to itm-bns-sample folder

```shell
> cd bns-java-client
> chmod u+x mvnw
```

Execute the program and enter the private key and PIN code. BNS auto folder attest will use the PIN code to encrypt your private key and store in `.itm.encrypted.private.key`.
After the first time you execute the code, the BNS auto folder attest will ask you to enter the PIN code to unlock your private key.
Private Key : Export from you MetaMask Account
PIN code : At least 8 characters or numbers

```shell
> ./mvnw.cmd clean package -DskipTests
> cd bns-auto-folder-attest
> java -jar ./target/itm-bns-sample-1.1.1-SNAPSHOT.jar
```

### 3. BNS Auto Folder Attest Callback

We already defined 7 Callback methods in the BNS Java Client documents. Now, we are going to introduce another 7 Callback methods in BNS Auto Folder Attest service.
In this Callback tutorial, the Callback output will be saved as a CSV file. You can implement the code in each Callback method to integrate with your applications after finishing this tutorial.

- [ScanResult.java](../src/main/java/com/itrustmachines/bnsautofolderattest/vo/ScanResult.java)
    ```java
    public class ScanResult {
      public ZonedDateTime startTime;
      public long totalCount;
      public long totalBytes;
      public long addedCount;
      public long modifiedCount;
      public long attestedCount;
      public ZonedDateTime endTime;
    }
    ```
1. `onScanResult` : This Callback allows the BNS Auto Folder Attest service to return the `scanResult` after finishing scanning the file. The `onScanResult` Callback will be stored in the `scanHistoryCsvPath.CSV`

- [CallbackImpl.java](../src/main/java/com/itrustmachines/bnsautofolderattest/service/CallbackImpl.java)

  ```java
  public void onScanResult(@NonNull final ScanResult scanResult) {
    log.info("onScanResult() scanResult={}", scanResult);
    writeScanHistory(scanResult);
  }
  ```


In the BNS Auto Folder Attest service, each file in the folder will be scanned and ledgerinput to the BNS server to attest. After finishing scanning the entire folder and building the `scanResult`. The BNS Auto Folder Attest service will call the `onScanResult` Callback to return the scan result method.
After the BNS server attests the file, it will return the `ledgerInputResponse`. Then the BNS auto folder attest service will build the `attestationRecord` by the `ledgerInputResponse`.

- [attestationRecord.java](../src/main/java/com/itrustmachines/bnsautofolderattest/vo/AttestationRecord.java)

    ```java
    public class AttestationRecord { 
        @DatabaseField(generatedId = true, columnName = ID_KEY)
        private Long id;

        @DatabaseField(columnName = "type", canBeNull = false, unknownEnumName = "UNKNOWN")
        private AttestationType type;

        @DatabaseField(columnName = STATUS_KEY, canBeNull = false, unknownEnumName = "UNKNOWN")
        private AttestationStatus status;

        private Path filePath;

        private Path relativeFilePath;

        private ZonedDateTime lastModifiedTime;

        @DatabaseField(columnName = "fileHash", canBeNull = false)
        private String fileHash;

        @ToString.Exclude
        @DatabaseField(columnName = "previousRecord", foreign = true)
        private AttestationRecord previousRecord;

        @DatabaseField(columnName = "address", canBeNull = false)
        private String address;

        private ZonedDateTime attestTime;

        @DatabaseField(columnName = CLEARANCE_ORDER_KEY, canBeNull = false)
        private Long clearanceOrder;

        @DatabaseField(columnName = INDEX_VALUE_KEY, canBeNull = false)
        private String indexValue;

        private Path proofPath;
    }
    ```


2. `onAttested` : This callback allows the BNS Auto Folder Attest service to return the `attestationRecord` when the file is attested successfully. 

- [CallbackImpl.java](../src/main/java/com/itrustmachines/bnsautofolderattest/service/CallbackImpl.java)

  ```java
  public void onAttested(@NonNull final AttestationRecord attestationRecord) {
    log.info("onAttested() attestationRecord={}", attestationRecord);
    writeHistory(attestationRecord);
  }
  ```

3. `onAttestFail` : This callback allows the BNS Auto Folder Attest service to return the `attestationRecord` when file attestation fails.
    
    ```java
    public void onAttestFail(@NonNull final AttestationRecord attestationRecord) {
        log.error("onAttestFail() attestationRecord={}", attestationRecord);
        writeHistory(attestationRecord);
    }
    ```

4. `onVerified` : This Callback is extended from `getVerifyReceiptResult` and allows the BNS Auto Folder Attest service to return the `attestationRecord` after verifying the receipt successfully. 
  
   ```java
   @SneakyThrows
   @Override
   public void onVerified(@NonNull final AttestationRecord attestationRecord) {
        log.error("onVerified() attestationRecord={}", attestationRecord);
        writeHistory(attestationRecord);
   }
   ```

5. `onVerifyFail` : This Callback is extended from `getVerifyReceiptResult` and allows the BNS Auto Folder Attest service to return the `attestationRecord` after verifying the receipt fails.
    
    ```java
    @Override
    public void onVerifyFail(@NonNull AttestationRecord attestationRecord) {
        log.error("onVerifyFail() attestationRecord={}", attestationRecord);
        writeHistory(attestationRecord);
    }
    ```
6. `onSaveProof` : This Callback is extended from `getVerifyReceiptResult` and allows BNS Auto Folder Attest service to return the `attestationRecord` after requesting and saving the verification proof successfully.

    ```java
    @Override
    public void onSaveProof(@NonNull AttestationRecord attestationRecord) {
        log.error("onSaveProof() attestationRecord={}", attestationRecord);
        writeHistory(attestationRecord);
    }
    ```

7. `onSaveFail` : This Callback is extended from `getVerifyReceiptResult` and allows BNS Auto Folder Attest service to return the `attestationRecord` after requesting and saving the verification proof fails.
    
    ```java
    @Override
    public void onSaveFail(@NonNull AttestationRecord attestationRecord) {
        log.error("onSaveFail() attestationRecord={}", attestationRecord);
        writeHistory(attestationRecord);
    }
    ```
----
BNS Auto Folder Attest document is now complete. You can return and go through BNS Java Client document 

Next Page : [Overview of BNS Java Client](../../itm-bns-sample/doc/summary_en.md)  
Last Page : [Configure setting of BNS Java Client](../../itm-bns-sample/doc/other_setting_en.md)
