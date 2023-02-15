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

![image](https://user-images.githubusercontent.com/52474199/218958011-0c759ce7-7f19-4a03-887b-f267b4ee67e4.png)



![image](https://user-images.githubusercontent.com/52474199/218958067-1bc3614a-f215-4238-815c-725738ba8eb6.png)

![image](https://user-images.githubusercontent.com/52474199/218958118-a25f68a7-cabb-41ee-a43e-9f83af5200ba.png)

![image](https://user-images.githubusercontent.com/52474199/218958159-2ba50fca-c97d-4a61-951c-87b7410e0711.png)

![image](https://user-images.githubusercontent.com/52474199/218958197-4df69f5a-d64f-48d8-9e06-d5b519beb245.png)
