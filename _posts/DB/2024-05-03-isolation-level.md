---
layout: post
title: MySQL Isolation level
category: DB
tags:
  - MySQL
last_modified_at: 2021-06-30 23:26:00 +0800
---

## 트랜잭션의 격리 수준(isolation level) 이란?
> 여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜젝션에서 변경하거나 조회하는 테이블 데이터를 볼 수 있게 허용하지 말지를 결정하는 것이다.


## 격리수준 종류

|-|DIRTY READ|NON-REPEATABLE READ|PHANTOM READ|
|---|:---:|:---:|:---:|
|READ UNCOMMITTED|발생|발생|발생|
|READ COMMITTED|없음|발생|발생|
|REPEATABLE READ|없음|없음|발생  (InnoDB는 없음)|
|SERIALIZABLE|없음|없음|없음|

### READ UNCOMMITTED
> 각 트랜잭션에서 변경 내용이 COMMIT 이나 ROLLBACK 여부에 상관없이 다른 트랜잭션에서 조회할 수 있다.
> - DIRTY READ 현상이 나타나는 격리수준이고, 데이터가 나타났다가 사라졌다 하는 현상을 초래한다.
> - RDBMS 표준에서는 트랜잭션 격리 수준으로 인정하지 않을 정도로 정합성에 문제가 많은 격리수준이다.

![IMG_0014](https://user-images.githubusercontent.com/50814622/209699103-fb4f5800-815d-4f49-9999-a6ae28b5208e.jpg)


### REPEATABLE READ
> **동일 트랜잭션 내에서는 똑같은 SELECT 쿼리를 실행했을 때 동일한 결과를 보여줄 수 있게 보장한다.**  
> ( 다른 트랜잭션에서 그 데이터를 변경하고 COMMIT 이 됐더라도 )
> - 언두(Undo) 영역에 백업된 이전 데이터를 이용해  동일한 결과를 보여준다.
> - `READ COMMITTED` 격리 수준에서 발생하는  `NON-REPEATABLE READ` 부정합이 발생하지 않는다.
> - MySQL 의 InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준이다.
> - SELECT ..  FOR UPDATE 와 같이 쿼리 실행시 레코드에 쓰기 잠금을 걸어야 하는 경우 언두 레코드에는 잠금을 걸 수 없어  `PHANTOM READ` 현상이 일어날 수 있다. (InnoDB 제외 )

![IMG_0016](https://user-images.githubusercontent.com/50814622/209699273-0a29630b-89e8-4911-beee-fc2cadc8aa21.jpg)

### SERIALIZABLE
> **읽기 작업도 공유 잠금 (읽기 잠금) 을 획득해야만 하며, 동시에 다른 트랜잭션은 그러한 레코드를 변경하지 못하게 한다.**
> - 가장 엄격한 격리 수준이고 그만큼 동시 처리 성능도 다른 격리 수준보다 떨어진다.
> - `PHANTOM READ` 현상이 발생하지 않는다.
> - InnoDB 스토리지 엔진에서는 `REPEATABLE READ` 에서도 `PHANTOM READ`  현상이 발생하지 않기 때문에 굳이 사용할 필요성은 없다.

## 현상

### DIRTY READ
> - **어떤 트랜잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있는 현상이다.**
> - `DIRTY READ` 가 허용되는 격리 수준은 `READ UNCOMMITTED` 이다.

### NON-REPEATABLE READ
> 하나의 트랜잭션내에서 똑같은 SELECT 쿼리를 실행했을 때는 항상 같은 결과를 가져와야 한다는 `REPEATABLE READ` 정합성에 어긋나는 현상이다.

![IMG_0015](https://user-images.githubusercontent.com/50814622/209699727-917977bb-a598-46f4-991d-f36df4cb7035.jpg)

### PHANTOM READ
> - **다른 트랜잭션에서 수행항 변경 작업에 의해 레코드가 보였다 안 보였다 하는 현상이다.**
> - PHANTOM ROW 라고도 한다.


![IMG_0017](https://user-images.githubusercontent.com/50814622/209699788-4e0b991a-ab5e-4637-922b-a5150b69bf14.jpg)


## 격리 수준 설정

### 설정파일 (my.cnf)
```
[mysqld]
transaction-isolation = REPEATABLE-READ
transaction-read-only = OFF
```

### Query
```sql
-- 글로벌 설정
SET GLOBAL TRANSACTION ISOLATION LEVEL <REPEATABLE READ | READ COMMITTED | READ UNCOMMITTED| SERIALIZABLE>;
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- 세션내 설정
SET [SESSION] TRANSACTION ISOLATION LEVEL <REPEATABLE READ | READ COMMITTED | READ UNCOMMITTED| SERIALIZABLE>;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- 조회
SELECT @@GLOBAL.transaction_isolation
SELECT @@SESSION.transaction_isolation
```

## Ref
[Real MySQL](https://www.yes24.com/Product/Goods/103415627)
[Docs](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html#set-transaction-isolation-level)  
[Aurora MySQL](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.html)
