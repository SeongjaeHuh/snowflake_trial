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
![image](https://user-images.githubusercontent.com/52474199/229073373-1cb1248c-6749-48e8-a11f-baf0cdc14612.png)


## 1. 환경구성
-- 웨어하우스 명령어로 생성 
-- ***에 user 이름 변경

### 1. Create WAREHOUSE

1. Create warehouse 명령어로 생성 
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
![image](https://user-images.githubusercontent.com/52474199/217409306-bbed788c-c6a8-4cee-9bae-8b6b252005d5.png)
![image](https://user-images.githubusercontent.com/52474199/217409423-8c97aaff-655a-4f13-8be0-389697f470ca.png)



## 2. LAB. 공통
1. USE WAREHOUSE
```
USE WAREHOUSE [username]_WH;
```
![image](https://user-images.githubusercontent.com/52474199/217408750-dd373abb-d10c-413f-b75d-c9e3cd931d56.png)



2. UI를 통해서 [username]_db database 생성해보기.
![image](https://user-images.githubusercontent.com/52474199/217410429-080446eb-0475-474b-a4b0-8d348effff26.png)
![image](https://user-images.githubusercontent.com/52474199/217410601-22c85774-9d60-4de7-bb40-5f9e9d5bdee4.png)

![image](https://user-images.githubusercontent.com/52474199/217410759-4f58ff74-fddb-4136-9ec0-1d9a8ac004e8.png)

3. Schema 생성
![image](https://user-images.githubusercontent.com/52474199/217410948-3ace4f0a-e5e1-440c-86b5-d70f4a104b27.png)
![image](https://user-images.githubusercontent.com/52474199/217411024-26b0282b-0d6d-4043-a775-35c26c9fa91f.png)
![image](https://user-images.githubusercontent.com/52474199/217411103-7c3f1a02-ad9a-4842-911b-62d4ced23d95.png)


4. Table 생성
![image](https://user-images.githubusercontent.com/52474199/217411393-90710ff5-e830-4028-8ee3-0c5306c6e9b5.png)
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


5. 권한주기

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
```
USE WAREHOUSE [username]_WH;
USE DATABASE [username]_DB;
USE SCHEMA PUBLIC;

DROP DATABASE [username]_DB;
```

-- 명령어 통해서 DB 생성

```
DROP WAREHOUSE [username]_WH;
CREATE DATABASE [username]_DB;
```

-- 테이블 생성 해보기 :
```
CREATE OR REPLACE TABLE [username]_DB.PUBLIC.[username]_TBL
    (id NUMBER(38,0), name STRING(10),
    country VARCHAR(20), order_date DATE);
```

------------------ 테이블 UI 탐색 ----------------
-- (PDF UI 부분 참조)

-- 왼쪽 Pane에서 SNOWFLAKE_SAMPLE_DATA DB에서 TPCH_SF1 스키마 찾아보기 

-- TPCH_SF1 오른쪽 클릭하여 **Set as Context** 수행

-- 오른쪽 위 컨텍스트 정보 바뀌는지 확인

-- TPCH_SF1 클릭하여  ORDERS 확인.

-- ORDER 테이블의 상세 정보 왼쪽 아래 확인

-- ORDER table 상세정보에 있는 preview data 눌러보기(혹은 ... 눌러서 preview data )



-- 아래 쿼리 실행 해보기
```
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
```
ALTER WAREHOUSE [username]_WH SUSPEND; /*RESUME (시작)*/
```




## 1. LAB. LOADING

1. table 생성
```
use warehouse [username]_wh; -- 생성한 웨어하우스 입력

CREATE DATABASE [username]_VEGE_DB;
use schema "[username]"."PUBLIC";

create table vegetable_details
(
plant_name varchar(25)
, root_depth_code varchar(1)
);

```
2. Load 할 file 선택 (https://bit.ly/3RHDM5w)
> [로컬 다운로드](https://bit.ly/3RHDM5w)


3. file format 생성
```
CREATE FILE FORMAT [DB명].[SCHEMA_명].[FILE_FORMAT명] 
       COMPRESSION = 'AUTO' FIELD_DELIMITER = ',' RECORD_DELIMITER = '\n' 
       SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '\042' TRIM_SPACE = FALSE 
       ERROR_ON_COLUMN_COUNT_MISMATCH = TRUE 
       ESCAPE = 'NONE' ESCAPE_UNENCLOSED_FIELD = '\134' DATE_FORMAT = 'AUTO' TIMESTAMP_FORMAT = 'AUTO' NULL_IF = ('\\N');
```

--제공된 txt파일을 data load wizard를 통해 로딩(pdf 자료 참고)

![image](https://user-images.githubusercontent.com/52474199/177914492-5daca478-5789-4f9c-8e69-8b8ec285ca63.png)

![image](https://user-images.githubusercontent.com/52474199/177914538-cc2a1a58-9d49-4305-985a-1076470f4adc.png)

![image](https://user-images.githubusercontent.com/52474199/177914554-736b0569-1c28-44d0-b223-788eba423427.png)

![image](https://user-images.githubusercontent.com/52474199/177914597-213f9646-56a0-47e2-ba4d-31f8a334d49e.png)



![image](https://user-images.githubusercontent.com/52474199/177914907-fd64dac4-e92d-4c02-a565-7b2571c81a8d.png)

![image](https://user-images.githubusercontent.com/52474199/217207935-a7a385d0-ef43-45bd-ad3c-26fb1754e272.png)


## 2. LAB. CACHE

-- WAREHOUSE 및 스키마 설정 

```
USE ROLE SYSADMIN;
USE WAREHOUSE [username]_WH;
USE SCHEMA SNOWFLAKE_SAMPLE_DATA.TPCH_SF100;
```

-- 1. Metdata cache 사용 확인

--아래 쿼리 실행
```
SELECT MIN(l_orderkey), MAX(l_orderkey), COUNT(*) FROM lineitem;
```
![image](https://user-images.githubusercontent.com/52474199/177480728-b91bd3ab-29fb-4380-b03f-64082534b856.png)

-- 아래의 Result 나오는 Pane에서 Query ID 클릭 하여 Pofile 확인. 메타 데이터 cache 사용됨 확인

-- result cache 실습용. Cache_result 사용하지 않도록
```
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

-- warehouse 자동 resume 되므로 아래 실행

```
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
![image](https://user-images.githubusercontent.com/52474199/177269953-2812941b-59e1-41c1-880a-ee15f63bc3f6.png)

-- Query ID눌러서 Profile에서 "Percentage Scanned from" 확인

-- WHERE만 조금 바뀐 바뀐 유사한 쿼리 실행.
```
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
![image](https://user-images.githubusercontent.com/52474199/177270492-bf7b17d8-45b3-45a9-8a54-0bec2da6f8bb.png)

-- Query ID 눌러서 "Percentage Scanned from" 확인


-- warehouse 종료
```
ALTER WAREHOUSE [username]_WH SUSPEND;
```





## 3. query profile 및 explain plan  

```
USE SNOWFLAKE_SAMPLE_DATA.TPCDS_SF10TCL;
ALTER SESSION SET USE_CACHED_RESULT=FALSE;
```

--   LIMIT있는 예제 쿼리 explain plan 실행:
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

## 4. LAB. FUNCTION

```
USE WAREHOUSE [username]_WH;
CREATE DATABASE IF NOT EXISTS [username]_DB;
USE [username]_DB.PUBLIC;
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

-- 대문자로 바꾸는 function 
```
SELECT c_name, UPPER(c_name) 
    FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER";
```
![image](https://user-images.githubusercontent.com/52474199/177478660-73570f90-5a46-4874-aff4-e50653ead7a7.png)

-- IFF function 확인  
```
SELECT  
    o_orderkey,
    o_totalprice,
    o_orderpriority,
    IFF(o_orderpriority like '1-URGENT', o_totalprice * 0.01, o_totalprice * 0.005)::number(16,2)       ShippingCost  
FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."ORDERS";
```
![image](https://user-images.githubusercontent.com/52474199/177478584-a1f9fed1-5321-4410-8811-b54f7da341f4.png)

-- CASE expression으로 Preffered Customer만 확인
```
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

![image](https://user-images.githubusercontent.com/52474199/177478763-b0a77375-c6e6-4b43-86ff-e3ba18bb75c7.png)


-- random() function으로 난수 생성
-- 매번 다른 난수가 나오지만 같은 seed를 줄 경우 같은 결과 

```
SELECT RANDOM() AS random_variable;
```
![image](https://user-images.githubusercontent.com/52474199/177479532-ea955876-7244-4790-afa4-28c7ab8c80c4.png)

```
SELECT RANDOM(100) AS random_fixed;
```
![image](https://user-images.githubusercontent.com/52474199/177479572-c116f9fa-c9b7-4832-aec3-00fbf841a0c2.png)

-- time, date 관련 함수
```
SELECT CURRENT_DATE(), DATE_PART('DAY', CURRENT_DATE()), CURRENT_TIME();
```
![image](https://user-images.githubusercontent.com/52474199/177488289-f84436ab-ea7c-4763-9568-be2d81892c7e.png)

-- 1시간 동안 쿼리 히스토리 조회
```
SELECT * FROM TABLE(information_schema.query_history
      (DATEADD('hours', -1, CURRENT_TIMESTAMP()), CURRENT_TIMESTAMP()))
ORDER BY start_time;
```
![image](https://user-images.githubusercontent.com/52474199/177479698-848db49e-8445-4d1d-b73a-7fc5427149c6.png)

-- 11.3.2  마지막 결과 조회 function
```
SELECT * FROM TABLE(RESULT_SCAN(LAST_QUERY_ID()));
```
![image](https://user-images.githubusercontent.com/52474199/177479742-69c1bcb8-11be-4e0d-8c3b-fa13c2d1da9d.png)


-- system function (스노우플레이크를 사용하기 위한 허용 IP, port 정보)
```
SELECT SYSTEM$ALLOWLIST();
```
![image](https://user-images.githubusercontent.com/52474199/177479809-7706989e-eef9-4c7e-b513-09312def3f24.png)

--  동일한 내용 읽기 쉽게 변환 
```
SELECT VALUE:type AS type,
       VALUE:host AS host,
       VALUE:port AS port
FROM TABLE(FLATTEN(INPUT => PARSE_JSON(SYSTEM$ALLOWLIST())));
```
![image](https://user-images.githubusercontent.com/52474199/177479874-ff08cdeb-c73f-47ba-9c56-d6f702c090db.png)

### javascript UDF 예제
-- 영어권 길이 측정 단위 -> 미터로 변환
```
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
![image](https://user-images.githubusercontent.com/52474199/177479984-d8b88ab5-f63f-4967-8b3a-f433909098df.png)

```
SELECT Convert2Meters(10, 'yard');
```
![image](https://user-images.githubusercontent.com/52474199/177480021-92ab1663-58aa-4a2f-9ad0-e0232904935d.png)


### SQL UDF 예제
-- 고객 별 주문 수 확인
```
CREATE OR REPLACE FUNCTION order_cnt(custkey number(38,0))
  RETURNS number(38,0)
  AS 
  $$
    SELECT COUNT(1) FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."ORDERS"WHERE o_custkey = custkey
  $$;
```  
![image](https://user-images.githubusercontent.com/52474199/177298252-7fc53bc5-6a68-4939-b9ac-da193f110218.png)
```  
SELECT C_name, C_address, order_cnt(C_custkey)
FROM "SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER" LIMIT 10;
```

![image](https://user-images.githubusercontent.com/52474199/177298146-a5d25c9d-94c7-46fe-894a-c45a02475bc2.png)


### Stored Procedure 예제

-- warehouse size 변경 함수 
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
![image](https://user-images.githubusercontent.com/52474199/177480272-406af3d2-54d3-4b2a-b61f-f24950c844a4.png)

-- 웨어 하우스 사이즈 넣어서 변경
```
CALL changewhsize ('[username]_wh', 'small');
````
![image](https://user-images.githubusercontent.com/52474199/177480356-2f80c5ef-bcc2-47df-8f14-5b28ac3a306b.png)

-- 오른쪽 위 context 정보에서 웨어하우스 사이즈 변경 되었는지 확인
![image](https://user-images.githubusercontent.com/52474199/177480467-d8be2544-8f41-476a-82c1-373b9adbeb55.png)
→
![image](https://user-images.githubusercontent.com/52474199/177480385-249fbd09-253d-429b-9813-0e77547d192f.png)

```
ALTER WAREHOUSE [username]_WH SET WAREHOUSE_SIZE=XSmall;
ALTER WAREHOUSE [username]_WH SUSPEND;
```

-- JAVASCRIPT STORED PROCEDURE
```
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
![image](https://user-images.githubusercontent.com/52474199/177303428-e61ab3cb-cff7-4047-ac05-8ca207a4fc15.png)

```
CREATE OR REPLACE TABLE STPROC_TEST_TABLE1 (NUM_COL1 FLOAT);
```
![image](https://user-images.githubusercontent.com/52474199/177303554-0e670666-d700-469e-9e08-1f335538d824.png)

```
CALL STPROC1(3.14::FLOAT);
```
![image](https://user-images.githubusercontent.com/52474199/177303626-51196c9c-ac3e-424e-a0d4-4f9a0826e0ee.png)


```
SELECT * FROM STPROC_TEST_TABLE1;
```
![image](https://user-images.githubusercontent.com/52474199/177303758-77d4e61c-a9c0-4301-9e87-f95fa630c35e.png)




![image](https://user-images.githubusercontent.com/52474199/217226996-9fcfc65e-49f4-4b05-bcd8-ecc40ffbd543.png)
