## Build thte ReceiptDao Applications

### About the ReceiptDao

The ReceiptDao ( Receipt Data Access Object ) is the interface which helps developers to store, find, and delete the receipt. Developers can implement the code in ReceiptDao to store the receipt in memory, database, cloud service or other services. In this document, we are going to store the receipt in the database as an example to help you understand the ReceiptDao.

### Prerequisites

- Complete the quickstart document
- Complete the CMD document
- Complete the Callback document

#### jdbcUrl

The BNS Client will create a new database with `JDBC_URL`. Developers can configure the database name or path in `JDBC_URL`

For the `JDBC_URL` setting, pleaser refer to [BnsClientSample.java](../src/main/java/com/itrustmachines/sample/BnsClientSample.java)

```java
public static String JDBC_URL = "jdbc:sqlite:BnsDevice.db";
```

Then initialize the `receiptService` by database. For the code of initialize `receiptService`, please refer to [BnsClientReceiptDao.java](../../bns-client/src/main/java/com/itrustmachines/client/todo/BnsClientReceiptDao.java)

```java
@SneakyThrows
ReceiptDaoSample(@NonNull final String jdbcUrl) {
  this.receiptService = new ReceiptService(new JdbcPooledConnectionSource(jdbcUrl));
}
```

#### save

After receiving the ledgerInputResponse, the BNS Client will call `handleReceiptEvent` to extract the receipt from ledgerInputResponse and call `save` to save the receipt in database.

- For the code of `handleReceiptEvent` method, please refer to [ReceiptEventProcessor.java](../../bns-client/src/main/java/com/itrustmachines/client/service/ReceiptEventProcessor.java)

  ```java
  public void handleReceiptEvent(final @NonNull ReceiptEvent event) {
    log.debug("handleReceiptEvent() begin, event={}", event);
    try {
      callback.obtainReceiptEvent(event);
    } catch (Exception e) {
      ...
      ...
    }
    final boolean isReceiptSaved = receiptService.save(event.getReceipt());
    log.debug("handleReceiptEvent() end, isReceiptSaved={}", isReceiptSaved);
  }
  ```

- For the code of `save`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public boolean save(@NonNull final Receipt receipt) {
    log.info("save() start, receipt={}", receipt);
    if (receipt == null) {
      return false;
    } else {
      final Receipt savedReceipt = receiptService.save(receipt);
      log.info("save() end, savedReceipt={}", savedReceipt);
      return Objects.nonNull(savedReceipt);
    }
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  private ReceiptEntity save(@NonNull final ReceiptEntity receiptEntity) {
    sigClientDao.createIfNotExists(receiptEntity.getSigClient());
    sigServerDao.createIfNotExists(receiptEntity.getSigServer());
    return receiptDao.createIfNotExists(receiptEntity);
  }
  
  public Receipt save(@NonNull final Receipt receipt) {
    final ReceiptEntity createEntity = save(ReceiptUtil.toEntity(receipt));
    return ReceiptUtil.toDomain(createEntity);
  }
  ```

#### saveAll

Instead of storing a receipt, Developers can choose to store the list of receipts in the database. The procedure of `saveAll` is the same as `save`.

- For the code of the `saveAll` method, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public boolean saveAll(@NonNull final List<Receipt> receipts) {
    log.debug("saveAll() start, receiptsSize={}", receipts.size());
    final List<Receipt> savedReceipts = receiptService.saveAll(receipts);
    log.debug("saveAll() end, savedReceiptsSize={}", savedReceipts.size());
    return receipts.size() == savedReceipts.size();
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public List<Receipt> saveAll(@NonNull final List<Receipt> receipts) {
    return receipts.stream()
                   .map(this::save)
                   .collect(Collectors.toList());
  }
  ```

#### findAll

`findAll` will find and return all the receipt in database.

- For the code of `findAll`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)
  
  ```java
  @Override
  public List<Receipt> findAll() {
    log.debug("findAll() start");
    final List<Receipt> receipts = receiptService.findAll();
    log.debug("findAll() end, receiptsSize={}", receipts.size());
    return receipts;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)
  
  ```java
  public List<Receipt> findAll() {
    return receiptDao.queryForAll()
                     .stream()
                     .map(ReceiptUtil::toDomain)
                     .collect(Collectors.toList());
  }
  ```

#### findAll

Instead of finding all the receipts, Developers can use this `findAll` method to find specific receipt by giving `pageNumber` and `pageSize`.

- For the code of `findAll`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)
  
  ```java
  @Override
  public List<Receipt> findAll(final int pageNumber, final int pageSize) {
    log.debug("findAll() start, pageNumber={}, pageSize={}", pageNumber, pageSize);
    final List<Receipt> receipts = receiptService.findAll(pageNumber, pageSize);
    log.debug("findAll() end, receiptsSize={}", receipts.size());
    return receipts;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)
  
  ```java
  public List<Receipt> findAll(final int pageNumber, final int pageSize) {
    final QueryBuilder<ReceiptEntity, Long> queryBuilder = receiptDao.queryBuilder();
    queryBuilder.offset((long) (pageNumber * pageSize))
                .limit((long) pageSize);
    return queryBuilder.query()
                       .stream()
                       .map(ReceiptUtil::toDomain)
                       .collect(Collectors.toList());
  }
  ```

#### countAll

`countAll` wil count and return the number of receipt in database.

- For the code of `countAll` method, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public long countAll() {
    log.debug("countAll() start");
    final long size = receiptService.countAll();
    log.debug("countAll() end, size={}", size);
    return size;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public long countAll() {
    return receiptDao.countOf();
  }
  ```

#### findByLocator

`findByLocator` can find out and return the receipt by giving `receiptLocator`. `receiptLocator` includes `indexValue` and `clearanceOrder`. The BNS Client will use these two variables to find the receipt.

- For the code of `findByLocator`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public Receipt findByLocator(@NonNull final ReceiptLocator receiptLocator) {
    log.debug("findByLocator() start, receiptLocator={}", receiptLocator);
    final Receipt receipt = receiptService.findByLocator(receiptLocator);
    log.debug("findByLocator() end, receipt={}", receipt);
    return receipt;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public Receipt findByLocator(@NonNull final ReceiptLocator receiptLocator) {
    final QueryBuilder<ReceiptEntity, Long> queryBuilder = receiptDao.queryBuilder();
    queryBuilder.where()
                .eq(ReceiptEntity.CLEARANCE_ORDER_KEY, receiptLocator.getClearanceOrder())
                .and()
                .eq(ReceiptEntity.INDEX_VALUE_KEY, receiptLocator.getIndexValue());
    final ReceiptEntity entity = queryBuilder.queryForFirst();
    return ReceiptUtil.toDomain(entity);
  }
  ```

#### findByLocators

`findByLocators` can find out and return the receipts by giving the list of `receiptLocators`. `receiptLocators` includes `indexValues` and `clearanceOrders`. The BNS Client will use these two variables to find out the receipts.

- For the code of `findByLocators` , please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public List<Receipt> findByLocators(@NonNull final List<ReceiptLocator> receiptLocators) {
    log.debug("findByLocators() start, receiptLocatorsSize={}", receiptLocators.size());
    final List<Receipt> result = receiptService.findByLocators(receiptLocators);
    log.debug("findByLocators() end, resultSize={}", result.size());
    return result;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public List<Receipt> findByLocators(@NonNull final List<ReceiptLocator> receiptLocators) {
    return receiptLocators.stream()
                          .map(this::findByLocator)
                          .collect(Collectors.toList());
  }
  ```

#### delete

`delete` will extract the `indexvalue` and `clearanceOrder` from receipt and calling `deleteByLocator` to delete the receipt.

- For the code of `delete`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public boolean delete(@NonNull final Receipt receipt) {
    log.debug("delete() start, receipt={}", receipt);
    final ReceiptLocator receiptLocator = ReceiptLocator.builder()
                                                        .clearanceOrder(receipt.getClearanceOrder())
                                                        .indexValue(receipt.getIndexValue())
                                                        .build();
    final int deleteCount = receiptService.deleteByLocator(receiptLocator);
    log.debug("delete() end, deleteCount={}", deleteCount);
    return deleteCount == 1;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public int deleteByLocator(@NonNull final ReceiptLocator receiptLocator) {
    DeleteBuilder<ReceiptEntity, Long> deleteBuilder = receiptDao.deleteBuilder();
    deleteBuilder.where()
                 .eq(ReceiptEntity.INDEX_VALUE_KEY, receiptLocator.getIndexValue())
                 .and()
                 .eq(ReceiptEntity.CLEARANCE_ORDER_KEY, receiptLocator.getClearanceOrder());
    PreparedDelete<ReceiptEntity> preparedDelete = deleteBuilder.prepare();
    return receiptDao.delete(preparedDelete);
  }
  ```

#### deleteAll

`deleteAll` will delete list of receipt.

- For the code of `deleteAll`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public boolean deleteAll(@NonNull final List<Receipt> receipts) {
    log.debug("deleteAll() start, receiptsSize={}", receipts.size());
    final int deleteCount = receiptService.deleteAll();
    log.debug("deleteAll() end, deleteCount={}", deleteCount);
    return deleteCount >= 0;
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public int deleteAll() {
    final DeleteBuilder<ReceiptEntity, Long> deleteBuilder = receiptDao.deleteBuilder();
    deleteBuilder.where();
    final PreparedDelete<ReceiptEntity> preparedDelete = deleteBuilder.prepare();
    return receiptDao.delete(preparedDelete);
  }
  ```

#### deleteAllByLocators

`deleteAllByLocators` will delete the receipts in the list of `receiptLocators`. `receiptLocators` includes `indexValues` and `clearanceOrders`. the BNS Client will use these two variables to delete the receipts.

- For the code of `deleteAllByLocators`, please refer to [ReceiptDaoSample.java](../src/main/java/com/itrustmachines/sample/ReceiptDaoSample.java)

  ```java
  @Override
  public boolean deleteAllByLocators(@NonNull final List<ReceiptLocator> receiptLocators) {
    log.debug("deleteAllByLocators() start, receiptLocatorsSize={}", receiptLocators.size());
    final int deleteCount = receiptService.deleteByLocatorList(receiptLocators);
    log.debug("deleteAllByLocators() end, deleteCount={}", deleteCount);
    return deleteCount == receiptLocators.size();
  }
  ```

- For the code of database access, please refer to [ReceiptService.java](../src/main/java/com/itrustmachines/sample/ReceiptService.java)

  ```java
  public int deleteByLocatorList(@NonNull final List<ReceiptLocator> receiptLocators) {
    return receiptLocators.stream()
                          .map(this::deleteByLocator)
                          .reduce(Integer::sum)
                          .orElse(0);
  }
  ```

#### getNeedVerifyReceiptLocatorMap

- Before verifying the receipt, the BNS Client will call `getNeedVerifyReceiptLocatorMap` to find out which receipts need to be verified. The receipts which need to be verified are `clearanceOrder` less than BNS Server `doneClearanceOrder`

- [BnsClientReceiptService.java](../../bns-client/src/main/java/com/itrustmachines/client/service/BnsClientReceiptService.java)

  ```java
  public Map<Long, Set<String>> getNeedVerifyReceiptLocatorMap(final long doneClearanceOrder) {
    log.debug("getNeedVerifyReceiptLocatorMap() doneClearanceOrder={}", doneClearanceOrder);
    rwLock.readLock()
          .lock();
    try {
      final Map<Long, Set<String>> result = new LinkedHashMap<>();
      final List<Long> coList = receiptLocatorsMap.keySet()
                                                  .stream()
                                                  .filter(co -> co <= doneClearanceOrder)
                                                  .sorted(Long::compareTo)
                                                  .collect(Collectors.toList());
      coList.forEach(co -> result.put(co, new LinkedHashSet<>(receiptLocatorsMap.get(co))));
      return result;
    } finally {
      rwLock.readLock()
            .unlock();
    }
  }
  ```

----
Building the ReceiptDao Application document is now complete. Next, we learn how to configure the remaining setting of the BNS Client.

## Next Steps

Next Page : [Configure the Setting of BNS Client](./other_setting_en.md)  
Last Page : [Build the Callback Applications](./callback_en.md)

