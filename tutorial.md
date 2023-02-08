# Snowflake Training

```
-- git hub (this page)
http://bit.ly/3lhZS2m

-- USER 선택 (google spread sheet)
https://bit.ly/3YhGH7i
user##/Abc123!@

-- snowflake 접속(아래 둘 다 같은 주소)
https://bit.ly/3RH086V
https://notzlle-gs39826.snowflakecomputing.com

--load sample
https://drive.google.com/file/d/1orzu44Gb6lorBxqVaTIeG6zBov3O-TLx/view?usp=sharing
```


![image](https://user-images.githubusercontent.com/52474199/215970816-12cbbbeb-42d3-486b-9a84-fdf0b5da2ed1.png)


### Create User (only for Instructor)
```sql
create user user100 password='abc123' default_role = sysadmin default_secondary_roles = ('ALL') must_change_password = true;
grant role sysadmin to user user100;
```


## 1. 환경구성
-- 웨어하우스 명령어로 생성 
-- ***에 user 이름 변경

### 1. Create WAREHOUSE 
> e.g.) login ID: user98 일 때,  [username]_WH -> user98_WH 

```sql
USE ROLE SYSADMIN;
CREATE WAREHOUSE [username]_WH 
  WAREHOUSE_SIZE = 'XSMALL'  -- 웨어하우스 사이즈
  WAREHOUSE_TYPE = 'STANDARD'
  AUTO_SUSPEND = 180  -- 사용 안할지 중지되는 시간 
  AUTO_RESUME = true --쿼리 실행시 자동으로 resume
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 1
  INITIALLY_SUSPENDED = true  -- 생성시 시작되지 않도록 
  SCALING_POLICY = 'STANDARD'
 ;
```
![image](https://user-images.githubusercontent.com/52474199/217408674-5cef7e16-ae43-46e8-8512-c5854595c3de.png)

2. Snowsight UI에서 생성

![image](https://user-images.githubusercontent.com/52474199/217409146-fc0c253b-bcf0-481b-862f-d010f87aa811.png)


## 2. LAB. 공통 실습
1. USE WAREHOUSE
```
USE WAREHOUSE [username]_WH;
```
![image](https://user-images.githubusercontent.com/52474199/217408750-dd373abb-d10c-413f-b75d-c9e3cd931d56.png)



2. UI를 통해서 [username]_db database 생성
> (Data > Database > (우측 상단) + Database)

![image](https://user-images.githubusercontent.com/52474199/217410429-080446eb-0475-474b-a4b0-8d348effff26.png)

> Database 명 입력

![image](https://user-images.githubusercontent.com/52474199/217410601-22c85774-9d60-4de7-bb40-5f9e9d5bdee4.png)

3. Schema 생성
> (가운데) Search > Database 선택 >(우측 상단) + Schema

![image](https://user-images.githubusercontent.com/52474199/217410948-3ace4f0a-e5e1-440c-86b5-d70f4a104b27.png)

> Schema 이름 입력

![image](https://user-images.githubusercontent.com/52474199/217411024-26b0282b-0d6d-4043-a775-35c26c9fa91f.png)

4. Table 생성
> (가운데) Search > Database 선택 > Schema 선택 > (우측 상단) + Create > Table > Standard

![image](https://user-images.githubusercontent.com/52474199/217411393-90710ff5-e830-4028-8ee3-0c5306c6e9b5.png)

> DDL 입력

![image](https://user-images.githubusercontent.com/52474199/217411457-c7f74104-9d5f-4785-83d4-fdcc1bb463ef.png)

> Execute DDL
```sql
create or replace TABLE USER01_DB.USER01_SCHEMA.USER01_TBL (
	ID NUMBER(38,0),
	NAME VARCHAR(10),
	COUNTRY VARCHAR(20),
	ORDER_DATE DATE
);
```
![image](https://user-images.githubusercontent.com/52474199/217411686-ab5934fb-41ca-49fe-a7d7-0deef58021ba.png)
![image](https://user-images.githubusercontent.com/52474199/217411723-8babe24a-007a-4d73-9bf9-e2414b9140ca.png)

<details>
5. (optional) 권한주기

```sql
CREATE ROLE NEW_ROLE;
grant usage on database USER01_DB to role NEW_ROLE;
grant usage on all schemas in database USER01_DB to role NEW_ROLE;
grant select on all tables in database USER01_DB to role NEW_ROLE;
grant select on all views in database USER01_DB to role NEW_ROLE;

grant usage on future schemas in database USER01_DB to role NEW_ROLE;
grant select on future tables in database USER01_DB to role NEW_ROLE;
grant select on future views in database USER01_DB to role NEW_ROLE;

GRANT ROLE NEW_ROLE TO USER USER50;
```
![image](https://user-images.githubusercontent.com/52474199/217415805-4b62a9a1-e77f-424a-adea-277680d8b5c5.png)
![image](https://user-images.githubusercontent.com/52474199/217412833-4e4a57dc-c55c-4edf-aa6e-eb00750fa9e7.png)

6. 권한을 받은 USER가 Login 했을 때 조회됨

![image](https://user-images.githubusercontent.com/52474199/217416107-cfc9a5e3-84e9-47e3-be9e-65df35f56103.png)



-- UI통해서 만든 DB 삭제
```sql
USE WAREHOUSE [username]_WH;
USE DATABASE [username]_DB;
USE SCHEMA PUBLIC;

DROP DATABASE [username]_DB;
```

-- 명령어 통해서 DB 생성

```sql
DROP WAREHOUSE [username]_WH;
CREATE DATABASE [username]_DB;
```

-- 테이블 생성 해보기 :
```sql
CREATE OR REPLACE TABLE [username]_DB.PUBLIC.[username]_TBL
    (id NUMBER(38,0), name STRING(10),
    country VARCHAR(20), order_date DATE);
```

### Table UI 탐색

-- 왼쪽 Pane에서 SNOWFLAKE_SAMPLE_DATA DB에서 TPCH_SF1 스키마 찾아보기 

-- TPCH_SF1 오른쪽 클릭하여 **Set as Context** 수행

-- 오른쪽 위 컨텍스트 정보 바뀌는지 확인

-- TPCH_SF1 클릭하여  ORDERS 확인.

-- ORDER 테이블의 상세 정보 왼쪽 아래 확인

-- ORDER table 상세정보에 있는 preview data 눌러보기(혹은 ... 눌러서 preview data )



-- 아래 쿼리 실행 해보기
```sql
SHOW TABLES;

SELECT COUNT(*) FROM orders;

SELECT * FROM supplier LIMIT 10;

SELECT MAX(o_totalprice) FROM orders;

SELECT o_orderpriority, SUM(o_totalprice)
FROM orders
GROUP BY o_orderpriority
ORDER BY SUM(o_totalprice);

SELECT o_orderpriority, SUM(o_totalprice)
FROM orders
GROUP BY o_orderpriority
ORDER BY o_orderpriority;
```

-- 웨어하우스 중지(시작)
```sql
ALTER WAREHOUSE [username]_WH SUSPEND; /*RESUME (시작)*/
```



</details>


## 1. LAB. LOADING (import Data into Table) & UNLOADING (export Data into Storage)

### 1. Create a Table

```sql
use warehouse [username]_wh; -- 생성한 웨어하우스 입력
USE DATABASE USER01_DB;
USE SCHEMA USER01_DB.USER01_SCHEMA;
```
```sql
CREATE OR REPLACE TABLE USER01_DB.USER01_SCHEMA.CUSTOMERS (
  id int,
  first_name string,
  last_name string,
  email string,
  gender string,
  Job string);
```
```sql
TRUNCATE TABLE CUSTOMERS;
INSERT INTO CUSTOMERS
VALUES
(1,'Bourke','Treble','btreble0@g.co','M','Health Coach'),
(2,'Iormina','Lahy','ilahy1@sciencedaily.com','M','Web Designer'),
(3,'Tracy','Curwen','tcurwen2@spiegel.de','M','Analyst Programmer'),
(4,'Megan','Omond','momond3@cyberchimps.com','F','Data Engineer'),
(5,'Bryana','Coton','bcoton4@google.nl','F','Web Designer'),
(6,'Christoffer','Woolward','cwoolward5@imgur.com','F','Health Coach'),
(7,'Olympia','Pappin','opappin6@unesco.org','F','Analyst Programmer'),
(8,'Grant','Sandal','gsandal7@cpanel.net','M','Analyst Programmer'),
(9,'Elias','Bertomier','ebertomier8@tmall.com','M','Data Engineer'),
(10,'Rex','Kinzel','rkinzel9@yellowpages.com','M','Design Engineer')
;
```
![image](https://user-images.githubusercontent.com/52474199/217439044-6f6c4f3c-8bd8-4020-8a3e-1b0b6abdea27.png)

### 2. Create a File Format
> describes a set of staged data to access or load into Snowflake tables.

```sql
CREATE OR REPLACE FILE FORMAT USER01_DB.USER01_SCHEMA.USER01_FF_UNLOAD
TYPE = CSV COMPRESSION = NONE FIELD_DELIMITER = ',' FILE_EXTENSION = 'CSV'  SKIP_HEADER = 0;

DESC FILE FORMAT USER01_FF_UNLOAD;
```
![image](https://user-images.githubusercontent.com/52474199/217439166-8ba5cf30-10ec-4196-ae6a-8aa447708e23.png)


### 3. Create a Stage
> Snowflake Stages are locations where data files are stored (“staged”) which helps in loading data into and unloading data out of database tables. The stage locations could be internal or external to Snowflake environment.

[More About Stage](https://thinketl.com/types-of-snowflake-stages-data-loading-and-unloading-features/)
```sql
CREATE OR REPLACE STAGE USER01_STAGE_UNLOAD
FILE_FORMAT = USER01_FF_UNLOAD;

DESC STAGE USER01_STAGE_UNLOAD;
```
![image](https://user-images.githubusercontent.com/52474199/217439551-d5166a16-be05-4d40-a55e-00e935c244ba.png)

### 4. Unload files from table to Stage

1. Internal Named Stage에 저장
```sql
COPY INTO @USER01_STAGE_UNLOAD 
     FROM CUSTOMERS 
     FILE_FORMAT = (FORMAT_NAME = USER01_FF_UNLOAD);
     
```
2. Table로부터 저장된 CSV
```sql
LIST @USER01_STAGE_UNLOAD;
```
![image](https://user-images.githubusercontent.com/52474199/217439732-be7b008e-ef78-4ddc-869f-189dbb324198.png)

3. Stage 에 있는 파일을 바로 Table형태로 조회 가능
```sql
SELECT $1, $2, $3, $4, $5, $6
  FROM @USER01_STAGE_UNLOAD 
  (FILE_FORMAT => "USER01_FF_UNLOAD");
```
![image](https://user-images.githubusercontent.com/52474199/217440578-eb546b1c-6c69-45f7-9636-deaef97af303.png)

### 5. Load File from Stage to Table

1. Create a New Table

```sql
CREATE OR REPLACE TABLE USER01_DB.USER01_SCHEMA.CUSTOMERS_2 (
  id int,
  first_name string,
  last_name string,
  email string,
  gender string,
  Job string);
```
2. Load File from Stage to Table

```sql
copy into CUSTOMERS_2
     FROM @USER01_STAGE_UNLOAD 
     FILE_FORMAT = (FORMAT_NAME = USER01_FF_UNLOAD);
```
![image](https://user-images.githubusercontent.com/52474199/217440827-a256ede1-072c-463d-8584-25f636a97a9e.png)

3. Select Data
```sql
SELECT * FROM CUSTOMERS_2;
```
![image](https://user-images.githubusercontent.com/52474199/217440972-bcaac0af-efbb-4ab3-b9f9-194306bfc262.png)

## 2. LAB. CACHE

### 1. WAREHOUSE 및 스키마 설정 

```sql
USE ROLE SYSADMIN;
USE WAREHOUSE [username]_WH;
USE SCHEMA SNOWFLAKE_SAMPLE_DATA.TPCH_SF100;
```

### 1. Metdata cache 사용 확인

1. Execute Qeuery
```sql
SELECT MIN(l_orderkey), MAX(l_orderkey), COUNT(*) FROM lineitem;
```
![image](https://user-images.githubusercontent.com/52474199/217444454-9e09131c-5068-4a53-bb4c-8b6e1c0612c6.png)

2. 아래의 Result 옆 Query ID 클릭 하여 Pofile 확인. 메타 데이터 cache 사용됨 확인

![image](https://user-images.githubusercontent.com/52474199/217444273-e8cbc060-3e3b-4991-b3a9-c99f23daf5b5.png)


3. Result cache 실습용. Cache_result 사용하지 않도록 설정
```sql
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

4. Execute a New Query
```sql
SELECT l_returnflag, l_linestatus,
    SUM(l_quantity) AS sum_qty,
    SUM(l_extendedprice) AS sum_base_price,
    SUM(l_extendedprice * (l_discount)) AS sum_disc_price, SUM(l_extendedprice * (l_discount) * (1+l_tax))
   AS sum_charge,
    AVG(l_quantity) AS avg_qty,
    AVG(l_extendedprice) AS avg_price,
    AVG(l_discount) AS avg_disc,
    COUNT(*) as count_order
FROM lineitem
WHERE l_shipdate <= dateadd(day, 90, to_date('1998-12-01'))
GROUP BY l_returnflag, l_linestatus
ORDER BY l_returnflag, l_linestatus;
```
![image](https://user-images.githubusercontent.com/52474199/217442269-be514b04-9d56-4296-826e-b3e0d06f8af0.png)


#### Query ID눌러서 Profile에서 "Percentage Scanned from" 확인
![image](https://user-images.githubusercontent.com/52474199/217473172-c3c4a5a5-eb43-4796-9b92-ea29bdc99a5b.png)


5. Execute a similar Qeuery (it changed only <b> WHERE </b> clause..)
```sql
SELECT l_returnflag, l_linestatus,
    SUM(l_quantity) AS sum_qty,
    SUM(l_extendedprice) AS sum_base_price,
    SUM(l_extendedprice * (l_discount)) AS sum_disc_price, SUM(l_extendedprice * (l_discount) * (1+l_tax))
    AS sum_charge,
    AVG(l_quantity) AS avg_qty,
    AVG(l_extendedprice) AS avg_price,
    AVG(l_discount) AS avg_disc,
    COUNT(*) as count_order
FROM lineitem
WHERE l_shipdate <= dateadd(day, 90, to_date('1998-12-01'))
and l_extendedprice <= 20000
GROUP BY l_returnflag, l_linestatus
ORDER BY l_returnflag, l_linestatus;
```

6. Query ID 눌러서 "Percentage Scanned from" 확인

![image](https://user-images.githubusercontent.com/52474199/217442606-dc53b667-c839-4c61-b4f1-9a0d0b6c79c4.png)


![image](https://user-images.githubusercontent.com/52474199/217443497-aa07fa24-1b04-488d-8534-8328b838ece9.png)
![image](https://user-images.githubusercontent.com/52474199/217443157-9512c21b-9514-4f8e-86b2-ea6c5ef366c0.png)



7. warehouse 종료
```
ALTER WAREHOUSE [username]_WH SUSPEND;
```




<details>
## 3. query profile 및 explain plan  

```
USE SNOWFLAKE_SAMPLE_DATA.TPCDS_SF10TCL;
ALTER SESSION SET USE_CACHED_RESULT=FALSE;
```

###  LIMIT있는 예제 쿼리 explain plan 실행:
```
EXPLAIN
SELECT c_customer_sk,
        c_customer_id, 
        c_last_name, 
        (ca_street_number || ' ' || ca_street_name),
        ca_city,  ca_state  
    FROM customer, customer_address
    WHERE c_customer_id = ca_address_id
    AND c_customer_sk between 100000 and 600000 
    ORDER BY ca_city, ca_state
    LIMIT 10;
```

--   LIMIT있는 동일한 쿼리 실제 실행:
```
SELECT c_customer_sk,
        c_customer_id, 
        c_last_name, 
        (ca_street_number || ' ' || ca_street_name),
        ca_city,  ca_state  
    FROM customer, customer_address
    WHERE c_customer_id = ca_address_id
    AND c_customer_sk between 100000 and 600000
    ORDER BY ca_city, ca_state
    LIMIT 10;
```
--   LIMIT 없는 쿼리 explain plan :

```
EXPLAIN
SELECT c_customer_sk,
        c_customer_id, 
        c_last_name, 
        (ca_street_number || ' ' || ca_street_name),
        ca_city,  ca_state  
    FROM customer, customer_address
    WHERE c_customer_id = ca_address_id
    AND c_customer_sk between 100000 and 600000
    ORDER BY ca_city, ca_state;
```

-- query ID 클릭으로 detail page 탐색
-- history tab에서 결과 확인
-- 본인 user_id로 필터 걸어서 결과 확인


--   위와 동일한 쿼리 실행 :    
```
SELECT c_customer_sk,
        c_customer_id, 
        c_last_name, 
        (ca_street_number || ' ' || ca_street_name),
        ca_city,  ca_state  
    FROM customer, customer_address
    WHERE c_customer_id = ca_address_id
    AND c_customer_sk between 100000 and 600000
    ORDER BY ca_city, ca_state;    
```
	
</details>


## 4. LAB. FUNCTION

### 1. 대문자로 바꾸는 function 
```sql
SELECT c_name, UPPER(c_name) 
    FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER";
```
![image](https://user-images.githubusercontent.com/52474199/217485028-7b3cd627-f5c9-459e-9d0e-f4a19fa6a1e0.png)

### 2. IFF function 확인  
```sql
SELECT  
    o_orderkey,
    o_totalprice,
    o_orderpriority,
    IFF(o_orderpriority like '1-URGENT', o_totalprice * 0.01, o_totalprice * 0.005)::number(16,2)       ShippingCost  
FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."ORDERS";
```
![image](https://user-images.githubusercontent.com/52474199/217484942-4b28a04d-26b9-4b77-b36a-d449eac0c72f.png)

### 3. CASE expression으로 Preffered Customer만 확인
```sql
SELECT (c_salutation || ' ' || c_first_name || ' ' || c_last_name) AS full_name,
    CASE
        WHEN c_preferred_cust_flag LIKE 'Y'
            THEN 'Preferred Customer'
        WHEN c_preferred_cust_flag LIKE 'N'
            THEN 'Not Preferred Customer'
        END AS customer_status
FROM "SNOWFLAKE_SAMPLE_DATA"."TPCDS_SF100TCL"."CUSTOMER"
LIMIT 100;
```
![image](https://user-images.githubusercontent.com/52474199/217484895-d1ed8913-9143-498d-8faa-63116238ef0c.png)

### 4. random() function으로 난수 생성
1. 매번 다른 난수가 나오지만 같은 seed를 줄 경우 같은 결과 

```sql
SELECT RANDOM() AS random_variable;
```
![image](https://user-images.githubusercontent.com/52474199/217485396-4d4cea6a-d32f-43d5-9b0b-2a176361d041.png)

```sql
SELECT RANDOM(100) AS random_fixed;
```
![image](https://user-images.githubusercontent.com/52474199/217485350-17109479-fb71-4fce-90f1-4ffb9137aaf8.png)

2. time, date 관련 함수

```sql
SELECT CURRENT_DATE(), DATE_PART('DAY', CURRENT_DATE()), CURRENT_TIME();
```
![image](https://user-images.githubusercontent.com/52474199/217485466-ee8000a2-c5e2-4536-a6aa-55130e7d42fc.png)

3. 1시간 동안 쿼리 히스토리 조회

```sql
SELECT * FROM TABLE(snowflake.information_schema.query_history
      (DATEADD('hours', -1, CURRENT_TIMESTAMP()), CURRENT_TIMESTAMP()))
ORDER BY start_time;
```
![image](https://user-images.githubusercontent.com/52474199/217485832-ee246d47-0308-4880-91aa-f7afafe40a3a.png)

4. 마지막 결과 조회 function

```sql
SELECT * FROM TABLE(RESULT_SCAN(LAST_QUERY_ID()));
```
![image](https://user-images.githubusercontent.com/52474199/217485993-074ca2e8-f6a4-4611-809b-1d5ea5f0e4b6.png)


 5. system function (스노우플레이크를 사용하기 위한 허용 IP, port 정보)

```sql
SELECT SYSTEM$ALLOWLIST();
```
![image](https://user-images.githubusercontent.com/52474199/217486053-878f757a-c15f-477a-ad84-0be6414bb0cb.png)

6.  동일한 내용 읽기 쉽게 변환
 
```sql
SELECT VALUE:type AS type,
       VALUE:host AS host,
       VALUE:port AS port
FROM TABLE(FLATTEN(INPUT => PARSE_JSON(SYSTEM$ALLOWLIST())));
```
![image](https://user-images.githubusercontent.com/52474199/217486198-f98cec30-1504-4332-b4f3-9580eb4b0f51.png)

### javascript UDF 예제
1. 영어권 길이 측정 단위 -> 미터로 변환
```sql
CREATE OR REPLACE FUNCTION Convert2Meters(lengthInput double, InputScale string )
    RETURNS double
    LANGUAGE javascript
    AS
    $$
    var scale_UC =  INPUTSCALE.toUpperCase();
    switch(scale_UC) {
    case 'INCH':
        return(LENGTHINPUT * 0.0254)
        break;
    case 'FEET':
        return(LENGTHINPUT * 0.3048)
        break;
    case 'YARD':
        return(LENGTHINPUT * 0.9144)
        break;

    default:
        return null;
        break; 
    }
  $$;
```
![image](https://user-images.githubusercontent.com/52474199/217486735-85bfc174-6ab2-4e71-b12a-eed17dd386d9.png)

```sql
SELECT Convert2Meters(10, 'yard');
```
![image](https://user-images.githubusercontent.com/52474199/217486784-6628efe4-5bd4-4ed3-b97c-ad17deef889d.png)


### SQL UDF 예제
1. 고객 별 주문 수 확인
```sql
CREATE OR REPLACE FUNCTION order_cnt(custkey number(38,0))
  RETURNS number(38,0)
  AS 
  $$
    SELECT COUNT(1) FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."ORDERS"WHERE o_custkey = custkey
  $$;
```  
![image](https://user-images.githubusercontent.com/52474199/217486954-5eb5bd9b-9021-411c-8464-903777e70cf6.png)
```sql
SELECT C_name, C_address, order_cnt(C_custkey)
FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER" LIMIT 10;
```
![image](https://user-images.githubusercontent.com/52474199/217487053-22a93439-b3ae-4c6c-8e3d-d6309625a7a1.png)

### Stored Procedure 예제

1. warehouse size 변경 함수 
```
create or replace procedure ChangeWHSize(wh_name STRING, wh_size STRING )
    returns string
    language javascript
    strict
    execute as owner
    as
    $$
    var wh_size_UC = WH_SIZE.toUpperCase();
    switch(wh_size_UC) {
    case 'XSMALL':
    case 'SMALL':
    case 'MEDIUM':
    case 'LARGE':
        break;
    case 'XLARGE':
    case 'X-LARGE':
    case 'XXLARGE':
    case 'X2LARGE':
    case '2X-LARGE':
    case 'XXXLARGE':
    case 'X3LARGE':
    case '3X-LARGE':
    case 'X4LARGE':
    case '4X-LARGE':
        return "Size: " + WH_SIZE + " is too large";
        break; 
    default:
        return "Size: " + WH_SIZE + " is not valid";
        break; 
    }
        
    var sql_command = 
     "ALTER WAREHOUSE IF EXISTS " + WH_NAME + " SET WAREHOUSE_SIZE = "+ WH_SIZE;
    try {
        snowflake.execute (
            {sqlText: sql_command}
            );
        return "Succeeded.";   
        }
    catch (err)  {
        return "Failed: " + err;   
        }
    $$
    ;
```
![image](https://user-images.githubusercontent.com/52474199/217487150-e2b00f38-c157-4266-84ee-c0e9fd8fc3f1.png)

2. 웨어 하우스 사이즈 넣어서 변경
```
CALL changewhsize ('[username]_wh', 'small');
````
![image](https://user-images.githubusercontent.com/52474199/217487258-ea6e9f77-4397-44ca-8bd2-82af8892b8bb.png)
![image](https://user-images.githubusercontent.com/52474199/217487335-72ac2240-ab53-4f5f-850a-8cf7caa8b0b3.png)


3. 오른쪽 위 context 정보에서 웨어하우스 사이즈 변경 되었는지 확인
![image](https://user-images.githubusercontent.com/52474199/177480467-d8be2544-8f41-476a-82c1-373b9adbeb55.png)
→
![image](https://user-images.githubusercontent.com/52474199/177480385-249fbd09-253d-429b-9813-0e77547d192f.png)

```
ALTER WAREHOUSE [username]_WH SET WAREHOUSE_SIZE=XSmall;
ALTER WAREHOUSE [username]_WH SUSPEND;
```

4. JAVASCRIPT STORED PROCEDURE
```javascript
CREATE OR REPLACE PROCEDURE STPROC1(FLOAT_PARAM1 FLOAT) 
RETURNS STRING 
LANGUAGE JAVASCRIPT STRICT 
AS 
   $$ 
      try { 
             snowflake.execute ( 
                                 {sqlText: "INSERT INTO STPROC_TEST_TABLE1 (NUM_COL1) VALUES (" + FLOAT_PARAM1 + ")"} 
                                ); 
                           return "Succeeded."; // Return status
           } catch (err) { 
                           return "Failed: " + err; // status 
                         } 
   $$ ;
```
![image](https://user-images.githubusercontent.com/52474199/217487541-6d42fa93-9314-4246-bc16-4de9db0780b0.png)
```sql
CREATE OR REPLACE TABLE STPROC_TEST_TABLE1 (NUM_COL1 FLOAT);
```
![image](https://user-images.githubusercontent.com/52474199/217487720-fdab64cc-8631-4b1c-8ba4-6f992e194e3e.png)

```sql
CALL STPROC1(3.14::FLOAT);
```
![image](https://user-images.githubusercontent.com/52474199/217487807-3a9fb42b-e384-4cd2-bfc6-aba4730c382d.png)


```
SELECT * FROM STPROC_TEST_TABLE1;
```
![image](https://user-images.githubusercontent.com/52474199/217487910-e1fa3211-d71c-4e27-882d-1cdd34a7d92e.png)

