# Data Loading Test 1

1. "CITIBIKE"."PUBLIC"."TRIPS"
2. snowflake, TRIPS, 61,468,359 rows1.2 GB
3. 우리가 사용할 데이터는 Citi Bike NYC에서 제공하는 자전거 공유 데이터입니다. 
4. 데이터는 미국 동부 리전의 Amazon AWS S3 버킷에서 내보내지고 사전 준비되었습니다. 
5. 데이터는 여행 시간, 위치, 사용자 유형, 성별, 나이 등에 대한 정보로 구성됩니다.
6. AWS S3에서 데이터는 61.5M 행, 377개의 객체 및 1.9GB의 압축을 나타냅니다.


## 1. sysadmin 역할 할당
```sql
use role sysadmin;
```

## 2. Warehouse 생성
```sql
CREATE OR REPLACE WAREHOUSE DEMO_HE 
WAREHOUSE_SIZE = 'XSMALL'
AUTO_SUSPEND = 300 
AUTO_RESUME = TRUE 
MIN_CLUSTER_COUNT = 1 
MAX_CLUSTER_COUNT = 1
;
```

## 3. Database 생성
```sql
CREATE DATABASE CITIBIKE;

use database CITIBIKE;
use schema PUBLIC;
```

## 4. Table 생성
```sql
create or replace table trips
(tripduration integer,
starttime timestamp,
stoptime timestamp,
start_station_id integer,
start_station_name string,
start_station_latitude float,
start_station_longitude float,
end_station_id integer,
end_station_name string,
end_station_latitude float,
end_station_longitude float,
bikeid integer,
membership_type string,
usertype string,
birth_year integer,
gender integer);
```
## 5. 외부 stage 생성

```sql
create stage citibike_trips
    url = 's3://snowflake-workshop-lab/citibike-trips-csv/'
;
```
```
list @citibike_trips;
```
## 6. fileformat 생성
```sql
create or replace file format csv type='csv'
  compression = 'auto' field_delimiter = ',' record_delimiter = '\n'
  skip_header = 0 field_optionally_enclosed_by = '\042' trim_space = false
  error_on_column_count_mismatch = false escape = 'none' escape_unenclosed_field = '\134'
  date_format = 'auto' timestamp_format = 'auto' null_if = ('') comment = 'file format for ingesting data for zero to snowflake'
;
```

```sql
show file formats in database citibike;
```


## 7. 데이터 로딩
```sql
copy into trips 
from @citibike_trips 
file_format=csv 
PATTERN = '.*csv.*' 
;
```
```sql
select * 
from trips
limit 10;
```


# Data Loading Test 2
```
-- 2. load_employee.csv
https://bit.ly/3k2iGCj
```
![image](https://user-images.githubusercontent.com/52474199/218957945-8d6285a0-c058-4c1c-82cb-582cf3b50053.png)

![image](https://user-images.githubusercontent.com/52474199/218958512-a34b93cb-afe8-4082-a943-83360e71c1c7.png)


![image](https://user-images.githubusercontent.com/52474199/218958569-3ce597c9-0d6d-4cf8-b5b7-de7f804d58ec.png)

![image](https://user-images.githubusercontent.com/52474199/218958639-4f1b33a1-8607-4c38-84d9-1e9896a4ed85.png)

![image](https://user-images.githubusercontent.com/52474199/218958874-4f138833-bbd4-40a3-853a-6d483790ccc0.png)

![image](https://user-images.githubusercontent.com/52474199/218958926-ae6c9f93-b17a-4385-ba4e-29f705a6ac78.png)

<details>
### (참고) cli 명령어 보기
#### 1. 테이블 생성
```sql
CREATE TABLE "TEST_DB"."PUBLIC"."EMPLOYEE_1" ("NAME" STRING, "AGE" INTEGER, "EMAIL" STRING, "JOB" STRING);
```
    
#### 2. Fileformat 생성
```sql
CREATE FILE FORMAT "TEST_DB"."PUBLIC".CSV 
            TYPE = 'CSV' COMPRESSION = 'AUTO' FIELD_DELIMITER = ',' RECORD_DELIMITER = '\n' SKIP_HEADER = 1 
            FIELD_OPTIONALLY_ENCLOSED_BY = 'NONE' TRIM_SPACE = FALSE ERROR_ON_COLUMN_COUNT_MISMATCH = TRUE 
            ESCAPE = 'NONE' ESCAPE_UNENCLOSED_FIELD = '\134' DATE_FORMAT = 'AUTO' TIMESTAMP_FORMAT = 'AUTO' NULL_IF = ('\\N');
```
    
#### 3. Copy into 명령어 수행
```sql
PUT file://<file_path>/load_employee.csv @EMPLOYEE_1/ui1676447718293
COPY INTO "TEST_DB"."PUBLIC"."EMPLOYEE_1" FROM @/ui1676447718293 FILE_FORMAT = '"TEST_DB"."PUBLIC"."CSV_FF"' ON_ERROR = 'ABORT_STATEMENT' PURGE = TRUE;
```
</details>

# Data Loading Test 3


## 1. table 생성
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
## 2. Load 할 file 선택 (https://bit.ly/3RHDM5w)
> [로컬 다운로드](https://bit.ly/3RHDM5w)


## 3. file format 생성
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

