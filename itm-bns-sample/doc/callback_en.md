## Build the Callback Applications

### About the Callback

The Callbacks send the API events which occurs between BNS Client and BNS Server to your system. We define 7 events that you can Callback. We will introduce these Callbacks in the following document.
You can go through BNS API Doc [here](https://bns.itrustmachines.com/api).

### Prerequisites
- Complete quickstarts document
- Complete build the CMD document

### Events

1. `register` : When the BNS Client initalizes, the BNS Client will send `registerRequest` to the BNS Server and receive `registerResult` from the BNS Server. Developers can implement the code in `register` method to Callback the information in `registerRequest` and `registerResult`.

2. `createLedgerInputByCmd` : After successfully initializing the BNS Client, BNS Client will store CMD and other data in `ledgerInputRequest` and do **ledgerInput** to send `ledgerInputRequest` to the BNS Server. Developers can implement the code in `createLedgerInputByCmd` method to Callback the information in `ledgerInputRequest`.

3. `obtainLedgerInputResponse` : The BNS Client will receive `ledgerInputResponse` from the BNS Server after sending `ledgerInputRequest`. Developers can implement the code in `obtainLedgerInputResponse` to Callback the information in `ledgerInputResponse`.

4. `obtainReceiptEvent` : The `receipt` is contained in `ledgerInputResponse`. Developers can implement the code in `obtainReceiptEvent` to Callback the information in `receipt`.

5. `obtainDoneClearanceOrderEvent` : The `doneClearanceOrder` is contained in `ledgerInputResponse`. The BNS Client will use `doneClearanceOrder` to find out which receipts need to be verified. Developers can implement the code in `obtainDoneClearanceOrderEvent` to Callback the information in `doneClearanceOrder`.

6. `obtainMerkleProof` : Before verifying the receipt, the BNS Client will request the `merkleProof` of not yet verified receipts from the Server. The Merkle Proof is evidence of receipt verification. The BNS Client will use the Merkle proof to verify whether receipt is in the TP-merkle tree. Developers can implement the code in `obtainMerkleProof` to Callback the information in `merkleProof`.

7. `getVerifyReceiptResult` : After receiving the Merkle Proof. The BNS Client will start to verify the receipt and store the result to `verifyReceiptResult`. Developers can implement the code in `getVerifyReceiptResult`  to Callback the information in `verifyReceiptResult`.


### register

**When initializing the BNS Client, the BNS Client will send `registerRequest` to the BNS Server and receive `registerResult` from the BNS Server. Developers can implement the code in the `register` Callback method to Callback the `registerRequest` and `registerResult`.**

- [RegisterRequest.java](../../bns-client/src/main/java/com/itrustmachines/client/register/vo/RegisterRequest.java)

  ```java
  public class RegisterRequest implements Serializable, Cloneable {
    private String address;         // MetaMask wallet address
    private String email;           // The email you sign up for BNS service
    private String toSignMessage;   // address
    private SpoSignature sig;       // Use SEC256K1 algorithm to get signature of toSignMessage
  }
  ```

- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)

  ```java
  public void register(RegisterRequest registerRequest, Bool registerResult) {
  }
  ```

### createLedgerInputByCmd

**After successfully initializing the BNS Client, the BNS Client will store the CMD and other attestation data in `ledgerInputRequest` and do ledgerInput to send `ledgerInputRequest` to the BNS Server. Developers can implement the code in `createLedgerInputByCmd` to Callback the `ledgerInputRequest`.**

- [LedgerInputRequest.java](../../bns-client/src/main/java/com/itrustmachines/client/input/vo/LedgerInputRequest.java)
  ```java
  public class LedgerInputRequest implements Serializable, Cloneable {
    private String callerAddress;   // MetaMask wallet address
    private String timestamp;       // The time when you POST ledgerInputRequest
    private String cmd;             // Attestation data
    private String indexValue;      // IndexValueLey + 'R' + SN，indexValueKey is your MetaMask wallet address
    private String metadata;        // Description of attestation data 
    private Long clearanceOrder;    // current clearanceOrder of BNS Client
    private SpoSignature sigClient; // Signature of your ledgerInputRequest
  }
  ```

- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)
  We recommend you store `indexValue` and `clearanceOrder` in `receiptLocator` so that you can search your records conveniently.

  ```java
  public void createLedgerInputByCmd(ReceiptLocator receiptLocator, LedgerInputRequest ledgerInputRequest) {
  }
  ```

### obtainLedgerInputResponse

**The BNS Client will receive `ledgerInputResponse` from the BNS Server after sending `ledgerInputRequest`. Developers can implement the code in `obtainLedgerInputResponse` to Callback the `ledgerInputResponse`.**

- [LedgerInputResponse.java](../../bns-client/src/main/java/com/itrustmachines/client/input/vo/LedgerInputResponse.java)
  ```java
  public class LedgerInputResponse {
    private String status;                      // The status of your ledgerInputRequest
    private String description;                 // The description of this ledgerInputRequest
    private Receipt receipt;                    // Receipt
    private List<Long> doneClearanceOrderList;  // Current BNS Server done clearanceOrder
  }
  ```

- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)
  We recommend you store `indexValue` and `clearanceOrder` in `receiptLocator` so that you can search your records conveniently.
  ```java
  public void obtainLedgerInputResponse(ReceiptLocator locator, String cmdJson,
      LedgerInputResponse ledgerInputResponse) {
  }
  ```

### obtainReceiptEvent

**The `receipt` is contained in `ledgerInputResponse`. Developers can implement the code in `obtainReceiptEvent` to Callback the `receipt`.**

- [Receipt.java](../../spo-common-domain-objects/src/main/java/com/itrustmachines/common/vo/Receipt.java)
  ```java
  public class Receipt implements Serializable, Cloneable {
    private String callerAddress;       // MetaMask wallet address
    private Long timestamp;             // The time when you POST ledgerInputRequest
    private String cmd;                 // Attestation data
    private String indexValue;          // IndexValueLey + 'R' + SN，indexValueKey is your MetaMask wallet address
    private String metadata;            // Description of attestation data
    private Long clearanceOrder;        // current clearanceOrder of BNS Client
    private SpoSignature sigClient;     // Signature of your ledgerInputRequest
    private Long timestampSPO;          // The time when BNS Server receive your ledgerInputRequest
    private String result;              // The result of ledgerInput
    private SpoSignature sigServer;     // The signature from BNS
  }
  ```
- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)
  
  We recommend you store `indexValue` and `clearanceOrder` in `receiptLocator` so that you can search your records conveniently.
  ```java
  public void obtainReceiptEvent(ReceiptEvent receiptEvent) {
  }
  ```

### obtainDoneClearanceOrderEvent

**The `doneClearanceOrder` is contained in `ledgerInputResponse` . The BNS Client will use `doneClearanceOrder` to find out which receipts need to be verified. Developers can implement the code in `obtainDoneClearanceOrderEvent` to Callback the `doneClearanceOrder`.**


- [DoneClearanceOrderEvent.java](../../bns-client/src/main/java/com/itrustmachines/client/verify/vo/DoneClearanceOrderEvent.java)

  ```java
  public class DoneClearanceOrderEvent {
    private long doneClearanceOrder;        // current BNS Server done clearanceOrder
  }
  ```

- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)

  ```java
  public void obtainDoneClearanceOrderEvent(DoneClearanceOrderEvent doneClearanceOrderEvent) {
  }
  ```


### obtainMerkleProof

**Before verifying the receipt, the BNS Client will request the `merkleProof` of to be verified receipts from the Server. The Merkle Proof is evidence of receipt verification. The BNS Client will use Merkle proof to verify whether the receipt is in the TP-merkle tree. Developers can implement the code in `obtainMerkleProof` to Callback the `merkleProof`.**

- [MerkleProof.java](../../spo-common-domain-objects/src/main/java/com/itrustmachines/common/vo/MerkleProof.java)

  ```java
  public class MerkleProof implements Serializable {
    private String slice;               // The slice of TP-Merkle Tree
    private List<PBPairValue> pbPair;   // The information in the leaf node of TP-Merkle Tree
    private Long clearanceOrder;        // The clearanceOrder of this to be verified receipt
    private SpoSignature sigServer;     // The signature from BNS Server
  }
  ```

- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)

  ```java
  public void obtainMerkleProof(ReceiptLocator receiptLocator, MerkleProof merkleProof) {
  }
  ```

### getVerifyReceiptResult

**After receiving the Merkle Proof, the BNS Client will start to verify the receipt and store the result to `verifyReceiptResult`. Developers can implement the code in `getVerifyReceiptResult` to Callback the `verifyReceiptResult`.**

- [VerifyReceiptAndMerkleProofResult.java](../../spo-common-verification/src/main/java/com/itrustmachines/verification/vo/VerifyReceiptAndMerkleProofResult.java)

  ```java
  public class VerifyReceiptAndMerkleProofResult {
    private Long clearanceOrder;                // The clearanceOrder field in ledgerInputRequest body.
    private String indexValue;                  // The indexValue field in ledgerInputRequest body.
    private ExistenceType existenceType;        // Whether the attestation in proof.
    private boolean pass;                       // The result of verification.
    private String status;                      // The status of request.
    private ProofExistStatus proofExistStatus;  // The exist status of proof.
    private String txHash;                      // The hash value of transaction.
    private Long timestamp;                     // The time when verify receipt
    private Long ledgerInputTimestamp;          // The timestamp in ledgerInputRequest.
    private Long receiptTimestamp;              // The timestamp in receipt.
    private String merkleProofRootHash;         // The root hash of merkle proof.
    private String contractRootHash;            // The hash value of attestation data on blockchain.
    private String description;                 // The description of request.
    private String cmd;                         // Attestation data.
    private String descriptionReport;           // The report of description
    private String verifyPbPairsReport;         // The log when verify PbPair
    private String verifySliceReport;           // The log when verify slice
    private boolean merkleproofSignatureOk;     // The verify result of merkle proof
    private boolean receiptSignatureOk;         // The verify result of receipt signature
    private boolean clearanceOrderOk;           // The verify result of clearanceOrder
    private boolean pbPairOk;                   // The verify result of pbPair
    private boolean sliceOk;                    // The verify result of slice
    private boolean clearanceRecordRootHashOk;  // The verify result of clearanceRecord root hash
    private VerifyNotExistProofStatus verifyNotExistProofResult; // The result of not exist proof verification
  }
  ```

- [CallbackSample.java](../src/main/java/com/itrustmachines/sample/CallbackSample.java)

  ```java
  public void getVerifyReceiptResult(Receipt receipt, MerkleProof merkleProof,
      VerifyReceiptAndMerkleProofResult verifyReceiptAndMerkleProofResult) {
  }
  ```

----
Building the Callback Application document is now complete. Next, we learn how to build the ReceiptDao Application.

## Next Steps

Next Page : [Build the ReceiptDao Applications](./receiptDao_en.md)  
Last Page : [Build the CMD](./cmd_en.md)
