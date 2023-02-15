## 01. Training

### 참고사항

```
[username] -> 본인 부여받은 user_name으로 변경 
e.g.) login ID: user98 일때,  [username]_WH -> user98_WH 
```

### 환경구성
-- 웨어하우스 명령어로 생성 
-- ***에 user 이름 변경


### 웨어하우스 생성 

```
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
![image](https://user-images.githubusercontent.com/52474199/177908998-00db7227-f1a4-4d6a-b2c7-bb2966df334a.png)



## 0. LAB. 공통

```
USE WAREHOUSE [username]_WH;
```

-- WEB UI 탐색
-- UI를 통해서 [username]_db database 생성해보기.

![image](https://user-images.githubusercontent.com/52474199/177910901-0cf15b01-c17e-41af-8da9-4589000e0b59.png)

![image](https://user-images.githubusercontent.com/52474199/177910940-9c5b0a61-fcd0-413a-9730-9c3bc7a6bf08.png)

![image](https://user-images.githubusercontent.com/52474199/177910995-f796bc10-8e0d-41a3-a45c-cad313d1ba54.png)

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

```
use warehouse [username]_wh; -- 생성한 웨어하우스 입력

CREATE DATABASE [username]_vege_DB;
--use schema "[username]_vege_DB"."PUBLIC";

create table vegetable_details_sfuser
(
plant_name varchar(25)
, root_depth_code varchar(1)
);

```

--제공된 txt파일을 data load wizard를 통해 로딩(pdf 자료 참고)

![image](https://user-images.githubusercontent.com/52474199/177914492-5daca478-5789-4f9c-8e69-8b8ec285ca63.png)

![image](https://user-images.githubusercontent.com/52474199/177914538-cc2a1a58-9d49-4305-985a-1076470f4adc.png)

![image](https://user-images.githubusercontent.com/52474199/177914554-736b0569-1c28-44d0-b223-788eba423427.png)

![image](https://user-images.githubusercontent.com/52474199/177914597-213f9646-56a0-47e2-ba4d-31f8a334d49e.png)

![image](https://user-images.githubusercontent.com/52474199/179532848-0215899f-067e-4193-8903-2fd36773f1e0.png)

![image](https://user-images.githubusercontent.com/52474199/179532930-84c52c23-689c-4610-877a-a16188f6c79c.png)

![image](https://user-images.githubusercontent.com/52474199/179532976-2156b50a-9485-4eee-b734-a7c39864020d.png)


![image](https://user-images.githubusercontent.com/52474199/177914907-fd64dac4-e92d-4c02-a565-7b2571c81a8d.png)

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
![image](https://user-images.githubusercontent.com/52474199/179520076-0188fefd-bcea-44d8-b82e-f13223a292c2.png)

-- 아래의 Result 나오는 Pane에서 Query ID 클릭 하여 Pofile 확인. 메타 데이터 cache 사용됨 확인
![image](https://user-images.githubusercontent.com/52474199/179520112-5c318543-17e4-4c0c-90a1-d6bd426dbded.png)


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
-- Query ID눌러서 Profile에서 "Percentage Scanned from" 확인

![image](https://user-images.githubusercontent.com/52474199/179536273-9857be75-610e-4f9f-a968-ff1a8f5b7b4f.png)


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
-- Query ID 눌러서 "Percentage Scanned from" 확인

![image](https://user-images.githubusercontent.com/52474199/179536568-9f2b74dc-1da0-4700-8217-5a023853efc1.png)


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
![image](https://user-images.githubusercontent.com/52474199/179523774-15b27c03-3609-499c-bcdb-037f1b485491.png)

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
![image](https://user-images.githubusercontent.com/52474199/179524378-a7a54c27-2365-4656-9c32-f474b086b951.png)
![image](https://user-images.githubusercontent.com/52474199/179524413-65d2685f-a3f1-42cb-88f9-0fdeb471e72e.png)


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
![image](https://user-images.githubusercontent.com/52474199/179524929-ec72ee5c-81ea-4064-a71c-7bb40dceca19.png)

-- query ID 클릭으로 detail page 탐색
-- history tab에서 결과 확인
-- 본인 user_id로 필터 걸어서 결과 확인

![image](https://user-images.githubusercontent.com/52474199/179525226-32113128-59e7-48c5-b89e-d6eb8a159942.png)

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
![image](https://user-images.githubusercontent.com/52474199/179525303-038b2363-32bb-410d-b7af-4107f90ec3c3.png)

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
SELECT SYSTEM$WHITELIST();
```
![image](https://user-images.githubusercontent.com/52474199/177479809-7706989e-eef9-4c7e-b513-09312def3f24.png)

--  동일한 내용 읽기 쉽게 변환 
```
SELECT VALUE:type AS type,
       VALUE:host AS host,
       VALUE:port AS port
FROM TABLE(FLATTEN(INPUT => PARSE_JSON(system$whitelist())));
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
