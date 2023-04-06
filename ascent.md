
Create or replace user KISHOREK

alter user KISHOREK set default_role = 'SYSADMIN';

alter user KISHOREK set default_warehouse = 'COMPUTE_WH';

alter user KISHOREK set default_namespace = 'DEMO_DB.PUBLIC';

use role accountadmin;

select demo_db.public.grader(step, (actual = expected), actual, expected, description) as graded_results from

(SELECT

 'DORA_IS_WORKING' as step

 ,(select 123 ) as actual

 ,123 as expected

 ,'Dora is working!' as description

);

CReate or replace database AGS_GAME_AUDIENCE

drop  schema IF EXISTS public

create  schema RAW

create table GAME_LOGS

(

RAW_LOG VARIANT

)

Create Stage uni_kishore

url='s3://uni-kishore'

list @uni_kishore/kickoff

//Create a JSON file format in the new database

CREATE FILE FORMAT FF_JSON_LOGS

TYPE = 'JSON'

COMPRESSION = 'AUTO'

ENABLE_OCTAL = FALSE

Advertisements

REPORT THIS AD

ALLOW_DUPLICATE = FALSE

STRIP_OUTER_ARRAY = TRUE

STRIP_NULL_VALUES = FALSE

IGNORE_UTF8_ERRORS = FALSE;

select $1 from @uni_kishore/kickoff

(file_format => ff_json_logs)

select $1 from @uni_kishore/updated_feed

(file_format => ff_json_logs)

updated_feed

TRuncate table game_logs

copy into game_logs

from @uni_kishore/updated_feed

file_format=(format_name = ff_json_logs)

copy into game_logs

from @uni_kishore/kickoff

file_format=(format_name = ff_json_logs)

select * from game_logs

Drop view if exists LOGS

Create View LOGS AS

select raw_log:agent::TEXT As Agent,

       raw_log:user_event::TEXT as user_event,

       raw_log:user_login::TEXT as user_login,

       raw_log:datetime_iso8601::TIMESTAMP_NTZ as datetime_iso8601,

       Raw_Log as Raw_Log

       from game_logs

Create View LOGS AS

       select raw_log:ip_address::TEXT As ip_address,

       raw_log:user_event::TEXT as user_event,

Advertisements

REPORT THIS AD

       raw_log:user_login::TEXT as user_login,

       raw_log:datetime_iso8601::TIMESTAMP_NTZ as datetime_iso8601,

       Raw_Log as Raw_Log

       from game_logs

       where ip_address is not null

select * from LOGS WHERE ip_address ilike '%63.%'

create schema  ENHANCED

select parse_ip('100.41.16.160′,'inet')

select parse_ip('107.217.231.17′,'inet')

select parse_ip('107.217.231.17′,'inet'):host;

select parse_ip('107.217.231.17′,'inet'):family;

select parse_ip('100.41.16.160′,'inet'):ipv4;

select parse_ip('63.235.11.128′,'inet'):ipv4; –kis ip

       –looking for empty AGENT column

select *

from ags_game_audience.raw.LOGS
where agent is null;

–looking for non-empty IP_ADDRESS column

select

RAW_LOG:ip_address::text as IP_ADDRESS

,*

from ags_game_audience.raw.LOGS
where RAW_LOG:ip_address::text is not null;

       select * from LOGS

       — DO NOT EDIT THIS CODE

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

 SELECT

Advertisements

REPORT THIS AD

 'DNGW01' as step

  ,(

      select count(*) 

      from ags_game_audience.raw.logs
      where is_timestamp_ntz(to_variant(datetime_iso8601))= TRUE

   ) as actual

, 250 as expected

, 'Project DB and Log File Set Up Correctly' as description

);

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

   'DNGW02' as step

   ,( select sum(tally) from(

        select (count(*) * -1) as tally

        from ags_game_audience.raw.logs
        union all

        select count(*) as tally

        from ags_game_audience.raw.game_logs)    

     ) as actual

   ,250 as expected

   ,'View is filtered' as description

);

–Look up Kishore and Prajina's Time Zone in the IPInfo share using his headset's IP Address with the PARSE_IP function.

select start_ip, end_ip, start_ip_int, end_ip_int, city, region, country, timezone

from IPINFO_GEOLOC.demo.location
where parse_ip('100.41.16.160', 'inet'):ipv4 –Kishore's Headset's IP Address

BETWEEN start_ip_int AND end_ip_int;

Advertisements

REPORT THIS AD

–Join the log and location tables to add time zone to each row using the PARSE_IP function.

select logs.*

       , loc.city
       , loc.region

       , loc.country

       , loc.timezone

from AGS_GAME_AUDIENCE.RAW.LOGS logs

join IPINFO_GEOLOC.demo.location loc

where parse_ip(logs.ip_address, 'inet'):ipv4

BETWEEN start_ip_int AND end_ip_int;

–a Look Up table to convert from hour number to “time of day name”

create table ags_game_audience.raw.time_of_day_lu
(  hour number

   ,tod_name varchar(25)

);

select * from time_of_day_lu

–insert statement to add all 24 rows to the table

insert into time_of_day_lu

values

(6,'Early morning'),

(7,'Early morning'),

(8,'Early morning'),

(9,'Mid-morning'),

(10,'Mid-morning'),

(11,'Late morning'),

(12,'Late morning'),

(13,'Early afternoon'),

(14,'Early afternoon'),

Advertisements

REPORT THIS AD

(15,'Mid-afternoon'),

(16,'Mid-afternoon'),

(17,'Late afternoon'),

(18,'Late afternoon'),

(19,'Early evening'),

(20,'Early evening'),

(21,'Late evening'),

(22,'Late evening'),

(23,'Late evening'),

(0,'Late at night'),

(1,'Late at night'),

(2,'Late at night'),

(3,'Toward morning'),

(4,'Toward morning'),

(5,'Toward morning');

–Check your table to see if you loaded it properly

select *

from time_of_day_lu

–Use two functions supplied by IPShare to help with an efficient IP Lookup Process!

–Wrap any Select in a CTAS statement

create table ags_game_audience.enhanced.logs_enhanced as(

SELECT logs.ip_address

, logs.user_login as GAMER_NAME

, logs.user_event as GAME_EVENT_NAME

, logs.datetime_iso8601 as GAME_EVENT_UTC

, city

, region

, country

Advertisements

REPORT THIS AD

, timezone

,convert_timezone('UTC',timezone,logs.datetime_iso8601) as GAMER_LTZ_NAME

,dayname(to_date(GAMER_LTZ_NAME)) as DOW_NAME

–,date_part(hour,GAMER_LTZ_NAME) –as hrs

, TOD_NAME

from AGS_GAME_AUDIENCE.RAW.LOGS logs

JOIN IPINFO_GEOLOC.demo.location loc

ON IPINFO_GEOLOC.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key

AND IPINFO_GEOLOC.public.TO_INT(logs.ip_address)

BETWEEN start_ip_int AND end_ip_int

JOIN time_of_day_lu ON time_of_day_lu.hour=date_part(hour,GAMER_LTZ_NAME));

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

  SELECT

   'DNGW03' as step

   ,( select count(*)

      from ags_game_audience.enhanced.logs_enhanced

      where dow_name = 'Sat'

      and tod_name = 'Early evening'  

      and gamer_name like '%prajina'

     ) as actual

   ,2 as expected

   ,'Playing the game on a Saturday evening' as description

);

EXECUTE TASK AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED;

drop  TASK AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED;

Create or replace task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED
warehouse=COMPUTE_WH

Advertisements

REPORT THIS AD

Schedule='5 minute'

as INSERT INTO AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED

SELECT logs.ip_address

, logs.user_login as GAMER_NAME

, logs.user_event as GAME_EVENT_NAME

, logs.datetime_iso8601 as GAME_EVENT_UTC

, city

, region

, country

, timezone

,convert_timezone('UTC',timezone,logs.datetime_iso8601) as GAMER_LTZ_NAME

,dayname(to_date(GAMER_LTZ_NAME)) as DOW_NAME

–,date_part(hour,GAMER_LTZ_NAME) –as hrs

, TOD_NAME

from AGS_GAME_AUDIENCE.RAW.LOGS logs

JOIN IPINFO_GEOLOC.demo.location loc

ON IPINFO_GEOLOC.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key

AND IPINFO_GEOLOC.public.TO_INT(logs.ip_address)

BETWEEN start_ip_int AND end_ip_int

JOIN time_of_day_lu ON time_of_day_lu.hour=date_part(hour,GAMER_LTZ_NAME);

select count(*)

from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

–Run the task to load more rows

execute task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED;

–check to see how many rows were added

select count(*)

from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

–first we dump all the rows out of the table

Advertisements

REPORT THIS AD

truncate table ags_game_audience.enhanced.LOGS_ENHANCED;

–then we put them all back in

INSERT INTO ags_game_audience.enhanced.LOGS_ENHANCED (

SELECT logs.ip_address

, logs.user_login as GAMER_NAME

, logs.user_event as GAME_EVENT_NAME

, logs.datetime_iso8601 as GAME_EVENT_UTC

, city

, region

, country

, timezone as GAMER_LTZ_NAME

, CONVERT_TIMEZONE( 'UTC',timezone,logs.datetime_iso8601) as game_event_ltz

, DAYNAME(game_event_ltz) as DOW_NAME

, TOD_NAME

from ags_game_audience.raw.LOGS logs

JOIN ipinfo_geoloc.demo.location loc

ON ipinfo_geoloc.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key

AND ipinfo_geoloc.public.TO_INT(logs.ip_address)

BETWEEN start_ip_int AND end_ip_int

JOIN ags_game_audience.raw.TIME_OF_DAY_LU tod

ON HOUR(game_event_ltz) = tod.hour);

–clone the table to save this version as a backup

–since it holds the records from the UPDATED FEED file, we'll name it _UF

create table ags_game_audience.enhanced.LOGS_ENHANCED_UF

clone ags_game_audience.enhanced.LOGS_ENHANCED;

–we should do this every 5 minutes from now until the next millenium – Y3K!!!

MERGE INTO ENHANCED.LOGS_ENHANCED e

USING (SELECT logs.ip_address

Advertisements

REPORT THIS AD

, logs.user_login as GAMER_NAME

, logs.user_event as GAME_EVENT_NAME

, logs.datetime_iso8601 as GAME_EVENT_UTC

, city

, region

, country

, timezone as GAMER_LTZ_NAME

, CONVERT_TIMEZONE( 'UTC',timezone,logs.datetime_iso8601) as game_event_ltz

, DAYNAME(game_event_ltz) as DOW_NAME

, TOD_NAME

from ags_game_audience.raw.LOGS logs

JOIN ipinfo_geoloc.demo.location loc

ON ipinfo_geoloc.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key

AND ipinfo_geoloc.public.TO_INT(logs.ip_address)

BETWEEN start_ip_int AND end_ip_int

JOIN ags_game_audience.raw.TIME_OF_DAY_LU tod

ON HOUR(game_event_ltz) = tod.hour)) r

ON r.user_login = e.GAMER_NAME

AND r.datetime_iso8601=e.GAME_EVENT_UTC

AND r.user_event=e.GAME_Event_Name

WHEN MATCHED THEN

UPDATE SET IP_ADDRESS = 'Hey I updated matching rows!';

–let's truncate so we can start the load over again

— remember we have that cloned back up so it's fine

truncate table AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

Create or replace task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED
warehouse=COMPUTE_WH

Schedule='5 minute'

Advertisements

REPORT THIS AD

as

MERGE INTO ENHANCED.LOGS_ENHANCED e

USING (

SELECT logs.ip_address

, logs.user_login as GAMER_NAME

, logs.user_event as GAME_EVENT_NAME

, logs.datetime_iso8601 as GAME_EVENT_UTC

, city

, region

, country

, timezone as GAMER_LTZ_NAME

, CONVERT_TIMEZONE( 'UTC',timezone,logs.datetime_iso8601) as game_event_ltz

, DAYNAME(game_event_ltz) as DOW_NAME

, TOD_NAME

from ags_game_audience.raw.LOGS logs

JOIN ipinfo_geoloc.demo.location loc

ON ipinfo_geoloc.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key

AND ipinfo_geoloc.public.TO_INT(logs.ip_address)

BETWEEN start_ip_int AND end_ip_int

JOIN ags_game_audience.raw.TIME_OF_DAY_LU tod

ON HOUR(game_event_ltz) = tod.hour) r

ON r.GAMER_NAME = e.GAMER_NAME

AND r.GAME_EVENT_UTC=e.GAME_EVENT_UTC

AND r.GAME_Event_Name=e.GAME_Event_Name

WHEN NOT MATCHED THEN

INSERT(IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME, GAME_EVENT_UTC, CITY, REGION, COUNTRY, GAMER_LTZ_NAME,game_event_ltz, DOW_NAME, TOD_NAME)

VALUES

(IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME, GAME_EVENT_UTC, CITY, REGION, COUNTRY, GAMER_LTZ_NAME,game_event_ltz, DOW_NAME, TOD_NAME)

Advertisements

REPORT THIS AD

EXECUTE TASK AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED;

create or replace TABLE AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED (

               IP_ADDRESS VARCHAR(16777216),

               GAMER_NAME VARCHAR(16777216),

               GAME_EVENT_NAME VARCHAR(16777216),

               GAME_EVENT_UTC TIMESTAMP_NTZ(9),

               CITY VARCHAR(16777216),

               REGION VARCHAR(16777216),

               COUNTRY VARCHAR(16777216),

               GAMER_LTZ_NAME  VARCHAR(16777216),

    game_event_ltz TIMESTAMP_NTZ(9),

               DOW_NAME VARCHAR(3),

               TOD_NAME VARCHAR(25)

);

–Testing cycle for MERGE. Use these commands to make sure the Merge works as expected

–Write down the number of records in your table

select * from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

–Run the Merge a few times. No new rows should be added at this time

EXECUTE TASK AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED;

–Check to see if your row count changed

select * from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

–Insert a test record into your Raw Table

–You can change the user_event field each time to create “new” records

–editing the ip_address or datetime_iso8601 can complicate things more than they need to

–editing the user_login will make it harder to remove the fake records after you finish testing

INSERT INTO ags_game_audience.raw.game_logs
select PARSE_JSON('{“datetime_iso8601″:”2025-01-01 00:00:00.000”, “ip_address”:”196.197.196.255″, “user_event”:”fake event”, “user_login”:”fake user”}');

–After inserting a new row, run the Merge again

Advertisements

REPORT THIS AD

EXECUTE TASK AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED;

–Check to see if any rows were added

select * from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

–When you are confident your merge is working, you can delete the raw records

delete from ags_game_audience.raw.game_logs where raw_log like '%fake user%';

–You should also delete the fake rows from the enhanced table

delete from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED

where gamer_name = 'fake user';

–Row count should be back to what it was in the beginning

select * from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

'DNGW04' as step

 ,( select count(*)/iff (count(*) = 0, 1, count(*))

  from table(ags_game_audience.information_schema.task_history

              (task_name=>'LOAD_LOGS_ENHANCED'))) as actual

 ,1 as expected

 ,'Task exists and has been run at least once' as description

 );

 Create or replace Stage UNI_KISHORE_PIPELINE

url='s3://uni-kishore-pipeline'

list @UNI_KISHORE_PIPELINE

//Create a JSON file format in the new database

CREATE FILE FORMAT FF_JSON_LOGS

TYPE = 'JSON'

COMPRESSION = 'AUTO'

ENABLE_OCTAL = FALSE

Advertisements

REPORT THIS AD

ALLOW_DUPLICATE = FALSE

STRIP_OUTER_ARRAY = TRUE

STRIP_NULL_VALUES = FALSE

IGNORE_UTF8_ERRORS = FALSE;

Create or replace table PIPELINE_LOGS

(RAW_LOG Variant)

copy into PIPELINE_LOGS

from @UNI_KISHORE_PIPELINE

file_format=(format_name = ff_json_logs)

Create or replace task AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES
USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE = 'XSMALL'

Schedule='5 minute' AS

copy into PIPELINE_LOGS

from @UNI_KISHORE_PIPELINE

file_format=(format_name = ff_json_logs)

select * from PIPELINE_LOGS

create or replace view AGS_GAME_AUDIENCE.RAW.PL_LOGS(

               IP_ADDRESS,

               USER_EVENT,

               USER_LOGIN,

               DATETIME_ISO8601,

               RAW_LOG

) as

       select raw_log:ip_address::TEXT As ip_address,

       raw_log:user_event::TEXT as user_event,

       raw_log:user_login::TEXT as user_login,

       raw_log:datetime_iso8601::TIMESTAMP_NTZ as datetime_iso8601,

       Raw_Log as Raw_Log

Advertisements

REPORT THIS AD

       from PIPELINE_LOGS

       where ip_address is not null;

select * from PL_LOGS

Create or replace task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED
USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE = 'XSMALL'

after  AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES
as

MERGE INTO ENHANCED.LOGS_ENHANCED e

USING (

SELECT logs.ip_address

, logs.user_login as GAMER_NAME

, logs.user_event as GAME_EVENT_NAME

, logs.datetime_iso8601 as GAME_EVENT_UTC

, city

, region

, country

, timezone as GAMER_LTZ_NAME

, CONVERT_TIMEZONE( 'UTC',timezone,logs.datetime_iso8601) as game_event_ltz

, DAYNAME(game_event_ltz) as DOW_NAME

, TOD_NAME

from ags_game_audience.raw.PL_LOGS logs

JOIN ipinfo_geoloc.demo.location loc

ON ipinfo_geoloc.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key

AND ipinfo_geoloc.public.TO_INT(logs.ip_address)

BETWEEN start_ip_int AND end_ip_int

JOIN ags_game_audience.raw.TIME_OF_DAY_LU tod

ON HOUR(game_event_ltz) = tod.hour) r

ON r.GAMER_NAME = e.GAMER_NAME

Advertisements

REPORT THIS AD

AND r.GAME_EVENT_UTC=e.GAME_EVENT_UTC

AND r.GAME_Event_Name=e.GAME_Event_Name

WHEN NOT MATCHED THEN

INSERT(IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME, GAME_EVENT_UTC, CITY, REGION, COUNTRY, GAMER_LTZ_NAME,game_event_ltz, DOW_NAME, TOD_NAME)

VALUES

(IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME, GAME_EVENT_UTC, CITY, REGION, COUNTRY, GAMER_LTZ_NAME,game_event_ltz, DOW_NAME, TOD_NAME)

EXECUTE TASK AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES;

–Turning on a task is done with a RESUME command

alter task AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES resume;

alter task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED resume;

–Keep this code handy for shutting down the tasks each day

alter task AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES suspend;

alter task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED suspend;

–Step 1 – how many files in the bucket?

list @AGS_GAME_AUDIENCE.RAW.UNI_KISHORE_PIPELINE;

–Step 2 – number of rows in raw table (should be file count x 10)

select count(*) from AGS_GAME_AUDIENCE.RAW.PIPELINE_LOGS;

–Step 3 – number of rows in raw table (should be file count x 10)

select count(*) from AGS_GAME_AUDIENCE.RAW.PL_LOGS;

–Step 4 – number of rows in enhanced table (should be file count x 10 but fewer rows is okay)

select count(*) from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED;

use role accountadmin;

grant EXECUTE MANAGED TASK on account to SYSADMIN;

–switch back to sysadmin

use role sysadmin;

USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE = 'XSMALL'

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

Advertisements

REPORT THIS AD

(

SELECT

'DNGW05' as step

 ,(

   select max(tally) from (

       select CASE WHEN SCHEDULED_FROM = 'SCHEDULE'

                         and STATE= 'SUCCEEDED'

              THEN 1 ELSE 0 END as tally

   from table(ags_game_audience.information_schema.task_history (task_name=>'GET_NEW_FILES')))

  ) as actual

 ,1 as expected

 ,'Task succeeds from schedule' as description

 );

create table ED_PIPELINE_LOGS as

 SELECT

    METADATA$FILENAME as log_file_name –new metadata column

  , METADATA$FILE_ROW_NUMBER as log_file_row_id –new metadata column

  , current_timestamp(0) as load_ltz –new local time of load

  , get($1,'datetime_iso8601′)::timestamp_ntz as DATETIME_ISO8601

  , get($1,'user_event')::text as USER_EVENT

  , get($1,'user_login')::text as USER_LOGIN

  , get($1,'ip_address')::text as IP_ADDRESS   

  FROM @AGS_GAME_AUDIENCE.RAW.UNI_KISHORE_PIPELINE
  (file_format => 'ff_json_logs');

  select * from ED_PIPELINE_LOGS

  –truncate the table rows that were input during the CTAS

truncate table ED_PIPELINE_LOGS;

–reload the table using your COPY INTO

Advertisements

REPORT THIS AD

COPY INTO ED_PIPELINE_LOGS

FROM (

    SELECT

    METADATA$FILENAME as log_file_name

  , METADATA$FILE_ROW_NUMBER as log_file_row_id

  , current_timestamp(0) as load_ltz

  , get($1,'datetime_iso8601′)::timestamp_ntz as DATETIME_ISO8601

  , get($1,'user_event')::text as USER_EVENT

  , get($1,'user_login')::text as USER_LOGIN

  , get($1,'ip_address')::text as IP_ADDRESS   

  FROM @AGS_GAME_AUDIENCE.RAW.UNI_KISHORE_PIPELINE
)

file_format = (format_name = ff_json_logs);

CREATE OR REPLACE PIPE GET_NEW_FILES

auto_ingest=true

aws_sns_topic='arn:aws:sns:us-west-2:321463406630:dngw_topic'

AS

COPY INTO ED_PIPELINE_LOGS

FROM (

    SELECT

    METADATA$FILENAME as log_file_name

  , METADATA$FILE_ROW_NUMBER as log_file_row_id

  , current_timestamp(0) as load_ltz

  , get($1,'datetime_iso8601′)::timestamp_ntz as DATETIME_ISO8601

  , get($1,'user_event')::text as USER_EVENT

  , get($1,'user_login')::text as USER_LOGIN

  , get($1,'ip_address')::text as IP_ADDRESS   

  FROM @AGS_GAME_AUDIENCE.RAW.UNI_KISHORE_PIPELINE
)

file_format = (format_name = ff_json_logs);

alter task AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES resume;

alter task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED suspend;

–create a stream that will keep track of changes to the table

create or replace stream ags_game_audience.raw.ed_cdc_stream
on table AGS_GAME_AUDIENCE.RAW.ED_PIPELINE_LOGS;

–look at the stream you created

show streams;

–check to see if any changes are pending

select system$stream_has_data('ed_cdc_stream');

–query the stream

select *

from ags_game_audience.raw.ed_cdc_stream;

–check to see if any changes are pending

select system$stream_has_data('ed_cdc_stream');

–if your stream remains empty for more than 10 minutes, make sure your PIPE is running

select SYSTEM$PIPE_STATUS('GET_NEW_FILES');

–if you need to pause or unpause your pipe

–alter pipe GET_NEW_FILES set pipe_execution_paused = true;

–alter pipe GET_NEW_FILES set pipe_execution_paused = false;

–process the stream by using the rows in a merge

MERGE INTO AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED e

USING (

        SELECT cdc.ip_address

        , cdc.user_login as GAMER_NAME

        , cdc.user_event as GAME_EVENT_NAME

        , cdc.datetime_iso8601 as GAME_EVENT_UTC

Advertisements

REPORT THIS AD

        , city

        , region

        , country

        , timezone as GAMER_LTZ_NAME

        , CONVERT_TIMEZONE( 'UTC',timezone,cdc.datetime_iso8601) as game_event_ltz

        , DAYNAME(game_event_ltz) as DOW_NAME

        , TOD_NAME

        from ags_game_audience.raw.ed_cdc_stream cdc

        JOIN ipinfo_geoloc.demo.location loc

        ON ipinfo_geoloc.public.TO_JOIN_KEY(cdc.ip_address) = loc.join_key

        AND ipinfo_geoloc.public.TO_INT(cdc.ip_address)

        BETWEEN start_ip_int AND end_ip_int

        JOIN AGS_GAME_AUDIENCE.RAW.TIME_OF_DAY_LU tod

        ON HOUR(game_event_ltz) = tod.hour
      ) r

ON r.GAMER_NAME = e.GAMER_NAME

AND r.GAME_EVENT_UTC = e.GAME_EVENT_UTC

AND r.GAME_EVENT_NAME = e.GAME_EVENT_NAME

WHEN NOT MATCHED THEN

INSERT (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME

        , GAME_EVENT_UTC, CITY, REGION

        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ

        , DOW_NAME, TOD_NAME)

        VALUES

        (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME

        , GAME_EVENT_UTC, CITY, REGION

        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ

        , DOW_NAME, TOD_NAME);

–Did all the rows from the stream disappear?

select *

from ags_game_audience.raw.ed_cdc_stream;

alter task AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES suspend;

alter task AGS_GAME_AUDIENCE.RAW.LOAD_LOGS_ENHANCED suspend;

–Create a new task that uses the MERGE you just tested

create or replace task AGS_GAME_AUDIENCE.RAW.CDC_LOAD_LOGS_ENHANCED
               USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE='XSMALL'

               SCHEDULE = '5 minutes'

               as

MERGE INTO AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED e

USING (

        SELECT cdc.ip_address

        , cdc.user_login as GAMER_NAME

        , cdc.user_event as GAME_EVENT_NAME

        , cdc.datetime_iso8601 as GAME_EVENT_UTC

        , city

        , region

        , country

        , timezone as GAMER_LTZ_NAME

        , CONVERT_TIMEZONE( 'UTC',timezone,cdc.datetime_iso8601) as game_event_ltz

        , DAYNAME(game_event_ltz) as DOW_NAME

        , TOD_NAME

        from ags_game_audience.raw.ed_cdc_stream cdc

        JOIN ipinfo_geoloc.demo.location loc

        ON ipinfo_geoloc.public.TO_JOIN_KEY(cdc.ip_address) = loc.join_key

        AND ipinfo_geoloc.public.TO_INT(cdc.ip_address)

        BETWEEN start_ip_int AND end_ip_int

        JOIN AGS_GAME_AUDIENCE.RAW.TIME_OF_DAY_LU tod

        ON HOUR(game_event_ltz) = tod.hour
      ) r

ON r.GAMER_NAME = e.GAMER_NAME

AND r.GAME_EVENT_UTC = e.GAME_EVENT_UTC

AND r.GAME_EVENT_NAME = e.GAME_EVENT_NAME

WHEN NOT MATCHED THEN

INSERT (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME

        , GAME_EVENT_UTC, CITY, REGION

        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ

        , DOW_NAME, TOD_NAME)

        VALUES

        (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME

        , GAME_EVENT_UTC, CITY, REGION

        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ

        , DOW_NAME, TOD_NAME);

alter task AGS_GAME_AUDIENCE.RAW.GET_NEW_FILES suspend;

alter task AGS_GAME_AUDIENCE.RAW.CDC_LOAD_LOGS_ENHANCED suspend;

–Create a new task that uses the MERGE you just tested

create or replace task AGS_GAME_AUDIENCE.RAW.CDC_LOAD_LOGS_ENHANCED
               USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE='XSMALL'

               SCHEDULE = '5 minutes'

When system$stream_has_data('ed_cdc_stream')

               as

MERGE INTO AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED e

USING (

        SELECT cdc.ip_address

        , cdc.user_login as GAMER_NAME

        , cdc.user_event as GAME_EVENT_NAME

        , cdc.datetime_iso8601 as GAME_EVENT_UTC

        , city

        , region

        , country

        , timezone as GAMER_LTZ_NAME

        , CONVERT_TIMEZONE( 'UTC',timezone,cdc.datetime_iso8601) as game_event_ltz

        , DAYNAME(game_event_ltz) as DOW_NAME

        , TOD_NAME

        from ags_game_audience.raw.ed_cdc_stream cdc

        JOIN ipinfo_geoloc.demo.location loc

        ON ipinfo_geoloc.public.TO_JOIN_KEY(cdc.ip_address) = loc.join_key

        AND ipinfo_geoloc.public.TO_INT(cdc.ip_address)

        BETWEEN start_ip_int AND end_ip_int

        JOIN AGS_GAME_AUDIENCE.RAW.TIME_OF_DAY_LU tod

        ON HOUR(game_event_ltz) = tod.hour
      ) r

ON r.GAMER_NAME = e.GAMER_NAME

AND r.GAME_EVENT_UTC = e.GAME_EVENT_UTC

AND r.GAME_EVENT_NAME = e.GAME_EVENT_NAME

WHEN NOT MATCHED THEN

INSERT (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME

        , GAME_EVENT_UTC, CITY, REGION

        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ

        , DOW_NAME, TOD_NAME)

        VALUES

        (IP_ADDRESS, GAMER_NAME, GAME_EVENT_NAME

        , GAME_EVENT_UTC, CITY, REGION

        , COUNTRY, GAMER_LTZ_NAME, GAME_EVENT_LTZ

        , DOW_NAME, TOD_NAME);

select* from AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED

–Truncate table AGS_GAME_AUDIENCE.ENHANCED.LOGS_ENHANCED

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

'DNGW06' as step

 ,(

   select CASE WHEN pipe_status:executionState::text = 'RUNNING' THEN 1 ELSE 0 END

   from(

   select parse_json(SYSTEM$PIPE_STATUS( 'ags_game_audience.raw.GET_NEW_FILES' )) as pipe_status)

  ) as actual

 ,1 as expected

 ,'Pipe exists and is RUNNING' as description

 );

 alter pipe mypipe set pipe_execution_paused = true;

 create schema curated

 select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

'DNGW07' as step

 ,(

   select 1

  ) as actual

 ,1 as expected

 ,'Dashboard Exists' as description

 );

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

'DNGW05' as step

 ,(

   select max(tally) from (

       select CASE WHEN SCHEDULED_FROM = 'SCHEDULE'

                         and STATE= 'SUCCEEDED'

              THEN 1 ELSE 0 END as tally

   from table(ags_game_audience.information_schema.task_history (task_name=>'GET_NEW_FILES')))

  ) as actual

 ,1 as expected

 ,'Task succeeds from schedule' as description

 );

 create database ZENAS_ATHLEISURE_DB

 drop schema public

 create schema PRODUCTS

 Create or replace  stage  UNI_KLAUS_CLOTHING

 url='s3://uni-klaus/clothing'

  Create or replace  stage  UNI_KLAUS_ZMD

 url='s3://uni-klaus/zenas_metadata'

   Create or replace  stage  UNI_KLAUS_SNEAKERS

 url='s3://uni-klaus/sneakers'

 select *

      from ZENAS_ATHLEISURE_DB.INFORMATION_SCHEMA.STAGES

 list @UNI_KLAUS_ZMD

 select $1

from @uni_klaus_zmd;

select $1

from @uni_klaus_zmd/product_coordination_suggestions.txt;

 select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

 SELECT

 'DLKW01' as step

  ,(

      select count(*) 

      from ZENAS_ATHLEISURE_DB.INFORMATION_SCHEMA.STAGES

      where stage_url ilike ('%/clothing%')

      or stage_url ilike ('%/zenas_metadata%')

      or stage_url like ('%/sneakers%')

   ) as actual

, 3 as expected

, 'Stages for Klaus bucket look good' as description

);

 create or replace  file format zmd_file_format_1

RECORD_DELIMITER = ';';

select $1,$2, $3, $4,$5,$6, $7, $8,$9,$10,$11

from @uni_klaus_zmd/product_coordination_suggestions.txt
(file_format => zmd_file_format_2);

create or replace file format zmd_file_format_3

FIELD_DELIMITER = '='

RECORD_DELIMITER = '^';

create view zenas_athleisure_db.products.SWEATBAND_COORDINATION as

select $1 Product_code, $2 HAS_MATCHING_SWEATSUIT

from @uni_klaus_zmd/product_coordination_suggestions.txt
(file_format => zmd_file_format_3);

select $1

from @uni_klaus_zmd/product_coordination_suggestions.txt
(file_format => zmd_file_format_1);

select $1,s2

from @uni_klaus_zmd/product_coordination_suggestions.txt
(file_format => zmd_file_format_3);

select $1 as sizes_available

from @uni_klaus_zmd/sweatsuit_sizes.txt
(file_format => zmd_file_format_1 );

create or replace file format zmd_file_format_2

FIELD_DELIMITER = '|'

TRIM_SPACE =TRUE

RECORD_DELIMITER = ';';

Create or replace  View zenas_athleisure_db.products.SWEATBAND_PRODUCT_LINE as

select replace($1,chr(13)||chr(10)) PRODUCT_CODE,$2 as HEADBAND_DESCRIPTION, $3 as WRISTBAND_DESCRIPTION

from @uni_klaus_zmd/swt_product_line.txt
(file_format => zmd_file_format_2 );

 SWT_LMG

drop file format  zmd_file_format_1

create or replace  file format zmd_file_format_1

RECORD_DELIMITER = ';';

create or replace  view zenas_athleisure_db.products.sweatsuit_sizes as

select replace($1,chr(13)||chr(10)) as sizes_available

from @uni_klaus_zmd/sweatsuit_sizes.txt
(file_format => zmd_file_format_1)

where sizes_available<>”

select * from zenas_athleisure_db.products.sweatsuit_sizes

select * from zenas_athleisure_db.products.SWEATBAND_PRODUCT_LINE

select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

 SELECT

   'DLKW02' as step

   ,(select sum(tally) from

        (select count(*) as tally

        from ZENAS_ATHLEISURE_DB.PRODUCTS.SWEATBAND_PRODUCT_LINE

        where length(product_code) > 7

        union

        select count(*) as tally

        from ZENAS_ATHLEISURE_DB.PRODUCTS.SWEATSUIT_SIZES

        where LEFT(sizes_available,2) = char(13)||char(10))    

     ) as actual

   ,0 as expected

   ,'Leave data where it lands.' as description

);

select $1

from @uni_klaus_clothing/90s_tracksuit.png;

select metadata$filename, count(metadata$file_row_number) Number_of_rows

from @uni_klaus_clothing/90s_tracksuit.png

Group by metadata$filename

select metadata$filename, metadata$file_row_number

from @uni_klaus_clothing/90s_tracksuit.png

–Directory Tables

select * from directory(@uni_klaus_clothing);

— Oh Yeah! We have to turn them on, first

alter stage uni_klaus_clothing

set directory = (enable = true);

–Now?

select * from directory(@uni_klaus_clothing);

–Oh Yeah! Then we have to refresh the directory table!

alter stage uni_klaus_clothing refresh;

–Now?

select * from directory(@uni_klaus_clothing);

–testing UPPER and REPLACE functions on directory table

select UPPER(RELATIVE_PATH) as uppercase_filename

, REPLACE(uppercase_filename,'/') as no_slash_filename

, REPLACE(no_slash_filename,'_',' ') as no_underscores_filename

, REPLACE(no_underscores_filename,'.PNG') as just_words_filename

from directory(@uni_klaus_clothing);

select Replace(REPLACE(RELATIVE_PATH,'/'),'.png') as product_name

from  directory(@uni_klaus_clothing);

–create an internal table for some sweat suit info

create or replace TABLE ZENAS_ATHLEISURE_DB.PRODUCTS.SWEATSUITS (

               COLOR_OR_STYLE VARCHAR(25),

               DIRECT_URL VARCHAR(200),

               PRICE NUMBER(5,2)

);

–fill the new table with some data

insert into  ZENAS_ATHLEISURE_DB.PRODUCTS.SWEATSUITS

          (COLOR_OR_STYLE, DIRECT_URL, PRICE)

values

('90s', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/90s_tracksuit.png&#8217;,500)

,('Burgundy', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/burgundy_sweatsuit.png&#8217;,65)

,('Charcoal Grey', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/charcoal_grey_sweatsuit.png&#8217;,65)

,('Forest Green', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/forest_green_sweatsuit.png&#8217;,65)

,('Navy Blue', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/navy_blue_sweatsuit.png&#8217;,65)

,('Orange', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/orange_sweatsuit.png&#8217;,65)

,('Pink', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/pink_sweatsuit.png&#8217;,65)

,('Purple', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/purple_sweatsuit.png&#8217;,65)

,('Red', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/red_sweatsuit.png&#8217;,65)

,('Royal Blue',     'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/royal_blue_sweatsuit.png&#8217;,65)

,('Yellow', 'https://uni-klaus.s3.us-west-2.amazonaws.com/clothing/yellow_sweatsuit.png&#8217;,65);

create or replace  view ZENAS_ATHLEISURE_DB.PRODUCTS.CATALOG as

 — 3 way join – internal table, directory table, and view based on external data

select color_or_style

, direct_url

, price

, size as image_size

, last_modified as image_last_modified

, sizes_available

from sweatsuits

join directory(@uni_klaus_clothing)

on relative_path = SUBSTR(direct_url,54,50)

cross join sweatsuit_sizes;

select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

 SELECT

 'DLKW03' as step

 ,(select count(*) from ZENAS_ATHLEISURE_DB.PRODUCTS.CATALOG) as actual

 ,198 as expected

 ,'Cross-joined view exists' as description

);

— Add a table to map the sweat suits to the sweat band sets

create table ZENAS_ATHLEISURE_DB.PRODUCTS.UPSELL_MAPPING

(

SWEATSUIT_COLOR_OR_STYLE varchar(25)

,UPSELL_PRODUCT_CODE varchar(10)

);

–populate the upsell table

insert into ZENAS_ATHLEISURE_DB.PRODUCTS.UPSELL_MAPPING

(

SWEATSUIT_COLOR_OR_STYLE

,UPSELL_PRODUCT_CODE

)

VALUES

('Charcoal Grey','SWT_GRY')

,('Forest Green','SWT_FGN')

,('Orange','SWT_ORG')

,('Pink', 'SWT_PNK')

,('Red','SWT_RED')

,('Yellow', 'SWT_YLW');

— Zena needs a single view she can query for her website prototype

create view catalog_for_website as

select color_or_style

,price

,direct_url

,size_list

,coalesce('BONUS: ' ||  headband_description || ' & ' || wristband_description, 'Consider White, Black or Grey Sweat Accessories')  as upsell_product_desc

from

(   select color_or_style, price, direct_url, image_last_modified,image_size

    ,listagg(sizes_available, ' | ') within group (order by sizes_available) as size_list

    from catalog

    group by color_or_style, price, direct_url, image_last_modified, image_size

) c

left join upsell_mapping u

on u.sweatsuit_color_or_style = c.color_or_style

left join sweatband_coordination sc

on sc.product_code = u.upsell_product_code

left join sweatband_product_line spl

on spl.product_code = sc.product_code

where price < 200 — high priced items like vintage sweatsuits aren't a good fit for this website

and image_size < 1000000 — large images need to be processed to a smaller size

;

select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

'DLKW04' as step

 ,(select count(*)

  from zenas_athleisure_db.products.catalog_for_website

  where upsell_product_desc like '%NUS:%') as actual

 ,6 as expected

 ,'Relentlessly resourceful' as description

show tasks

json

create database MELS_SMOOTHIE_CHALLENGE_DB

drop schema public

Create schema TRAILS

create or replace stage  trails_geojson

    url = 's3://uni-lab-files-more/dlkw/trails/trails_geojson'

    create or replace stage  trails_parquet

    url = 's3://uni-lab-files-more/dlkw/trails/trails_parquet'

    list @trails_geojson

    list @trails_parquet

    //Create a JSON file format in the new database

CREATE FILE FORMAT FF_JSON

TYPE = 'JSON'

COMPRESSION = 'AUTO'

ENABLE_OCTAL = FALSE

ALLOW_DUPLICATE = FALSE

STRIP_OUTER_ARRAY = TRUE

STRIP_NULL_VALUES = FALSE

IGNORE_UTF8_ERRORS = FALSE;

    //Create a JSON file format in the new database

CREATE or replace  FILE FORMAT FF_PARQUET

TYPE = 'PARQUET'

COMPRESSION = 'AUTO' 

select $1 from @trails_geojson (file_format=> ff_json)

select $1 from @trails_parquet (file_format=> FF_PARQUET)

create or replace view CHERRY_CREEK_TRAIL as

–Nicely formatted trail data

select

 $1:sequence_1 as point_id,

 $1:trail_name::varchar as trail_name,

 $1:latitude::number(11,8) as lng, –remember we did a gut check on this data

 $1:longitude::number(11,8) as lat

from @trails_parquet

(file_format => ff_parquet)

order by point_id;

select * from CHERRY_CREEK_TRAIL

select top 100

 lng||' '||lat as coord_pair

,'POINT('||coord_pair||')' as trail_point

from cherry_creek_trail;

–To add a column, we have to replace the entire view

–changes to the original are shown in red

create or replace view cherry_creek_trail as

select

 $1:sequence_1 as point_id,

 $1:trail_name::varchar as trail_name,

 $1:latitude::number(11,8) as lng,

 $1:longitude::number(11,8) as lat,

 lng||' '||lat as coord_pair

from @trails_parquet

(file_format => ff_parquet)

order by point_id;

select

'LINESTRING('||

listagg(coord_pair, ',')

within group (order by point_id)

||')' as my_linestring

from cherry_creek_trail

where point_id <= 10

group by trail_name;

create or replace view DENVER_AREA_TRAILS as

select

$1:features[0]:properties:Name::string as feature_name

,$1:features[0]:geometry:coordinates::string as feature_coordinates

,$1:features[0]:geometry::string as geometry

,$1:features[0]:properties::string as feature_properties

,$1:crs:properties:name::string as specs

,$1 as whole_object

,st_length(to_geography(geometry)) as trail_length

from @trails_geojson (file_format => ff_json);

select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

SELECT

'DLKW06' as step

 ,(select count(*) as tally

      from mels_smoothie_challenge_db.information_schema.views
      where table_name in ('CHERRY_CREEK_TRAIL','DENVER_AREA_TRAILS')) as actual

 ,2 as expected

 ,'Mel\'s views on the geospatial data from Camila' as description

 );

create view DENVER_AREA_TRAILS_2 as

 –Remember this code?

select

trail_name as feature_name

,'{“coordinates”:['||listagg('['||lng||','||lat||']',',')||'],”type”:”LineString”}' as geometry

,st_length(to_geography(geometry)) as trail_length

from cherry_creek_trail

group by trail_name;

–Create a view that will have similar columns to DENVER_AREA_TRAILS

select feature_name, geometry, trail_length

from DENVER_AREA_TRAILS

union all

select feature_name, geometry, trail_length

from DENVER_AREA_TRAILS_2;

Create or replace view  trails_and_boundaries as

–Add more GeoSpatial Calculations to get more GeoSpecial Information!

select feature_name

, to_geography(geometry) as my_linestring

, st_xmin(my_linestring) as min_eastwest

, st_xmax(my_linestring) as max_eastwest

, st_ymin(my_linestring) as min_northsouth

, st_ymax(my_linestring) as max_northsouth

, trail_length

from DENVER_AREA_TRAILS

union all

select feature_name

, to_geography(geometry) as my_linestring

, st_xmin(my_linestring) as min_eastwest

, st_xmax(my_linestring) as max_eastwest

, st_ymin(my_linestring) as min_northsouth

, st_ymax(my_linestring) as max_northsouth

, trail_length

from DENVER_AREA_TRAILS_2;

select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

 SELECT

  'DLKW07' as step

   ,(select round(max(max_northsouth))

      from MELS_SMOOTHIE_CHALLENGE_DB.TRAILS.TRAILS_AND_BOUNDARIES)

      as actual

 ,40 as expected

 ,'Trails Northern Extent' as description

 );

— Melanie's Location into a 2 Variables (mc for melanies cafe)

set mc_lat='-104.97300245114094′;

set mc_lng='39.76471253574085′;

–Confluence Park into a Variable (loc for location)

set loc_lat='-105.00840763333615′;

set loc_lng='39.754141917497826′;

–Test your variables to see if they work with the Makepoint function

select st_makepoint($mc_lat,$mc_lng) as melanies_cafe_point;

select st_makepoint($loc_lat,$loc_lng) as confluent_park_point;

–use the variables to calculate the distance from

–Melanie's Cafe to Confluent Park

select st_distance(

        st_makepoint($mc_lat,$mc_lng)

        ,st_makepoint($loc_lat,$loc_lng)

        ) as mc_to_cp;

CREATE FUNCTION distance_to_mc(loc_lat number(38,32), loc_lng number(38,32))

  RETURNS FLOAT

  AS

  $$

   st_distance(

        st_makepoint('-104.97300245114094′,'39.76471253574085')

        ,st_makepoint(loc_lat,loc_lng)

        )

  $$

  ;

  –Tivoli Center into the variables

set tc_lat='-105.00532059763648′;

set tc_lng='39.74548137398218′;

select distance_to_mc($tc_lat,$tc_lng);

Create or replace view COMPETITION  as

select *

from OPENSTREETMAP_DENVER.DENVER.V_OSM_DEN_AMENITY_SUSTENANCE

where

    ((amenity in ('fast_food','cafe','restaurant','juice_bar'))

    and

    (name ilike '%jamba%' or name ilike '%juice%'

     or name ilike '%superfruit%'))

 or

    (cuisine like '%smoothie%' or cuisine like '%juice%');

create or replace view DENVER_BIKE_SHOPS as

select *,distance_to_mc(coordinates) AS distance_to_melanies

from OPENSTREETMAP_DENVER.DENVER.V_OSM_DEN_SHOP where shop ilike '%bicycle%'

     select truncate(distance_to_melanies)

      from denver_bike_shops

      where name like '%Mojo%'

select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

  SELECT

  'DLKW08' as step

  ,(select truncate(distance_to_melanies)

      from mels_smoothie_challenge_db.trails.denver_bike_shops

      where name like '%Mojo%') as actual

  ,14084 as expected

  ,'Bike Shop View Distance Calc works' as description

 );

DENVER_BIKE_SHOPS

    SELECT

 name

 ,cuisine

 , ST_DISTANCE(

    st_makepoint('-104.97300245114094′,'39.76471253574085')

    , coordinates

  ) AS distance_from_melanies

 ,*

FROM  competition

ORDER by distance_from_melanies;

CREATE OR REPLACE FUNCTION distance_to_mc(lat_and_lng GEOGRAPHY)

  RETURNS FLOAT

  AS

  $$

   st_distance(

        st_makepoint('-104.97300245114094′,'39.76471253574085')

        ,lat_and_lng

        )

  $$

  ;

  SELECT

 name

 ,cuisine

 ,distance_to_mc(coordinates) AS distance_from_melanies

 ,*

FROM  competition

ORDER by distance_from_melanies;

— Tattered Cover Bookstore McGregor Square

set tcb_lat='-104.9956203′;

set tcb_lng='39.754874′;

–this will run the first version of the UDF

select distance_to_mc($tcb_lat,$tcb_lng);

–this will run the second version of the UDF, bc it converts the coords

–to a geography object before passing them into the function

select distance_to_mc(st_makepoint($tcb_lat,$tcb_lng));

–this will run the second version bc the Sonra Coordinates column

— contains geography objects already

select name

, distance_to_mc(coordinates) as distance_to_melanies

, ST_ASWKT(coordinates)

from SONRA_DENVER_CO_USA_FREE.DENVER.V_OSM_DEN_SHOP

where shop='books'

and name like '%Tattered Cover%'

and addr_street like '%Wazee%';

 create or replace stage  CHERRY_CREEK_TRAIL

    url = 's3://uni-lab-files-more/dlkw/trails/trails_parquet/cherry_creek_trail.parquet'

    list @CHERRY_CREEK_TRAIL

select $1 from @CHERRY_CREEK_TRAIL (file_format=> FF_PARQUET)

create or replace external table T_CHERRY_CREEK_TRAIL(

               my_filename varchar(50) as (metadata$filename::varchar(50))

)

location= @trails_parquet

auto_refresh = true

file_format = (type = parquet);

Create or replace view v_cherry_creek_trail as

select

 $1:sequence_1 as point_id,

 $1:trail_name::varchar as trail_name,

 $1:latitude::number(11,8) as lng,

 $1:longitude::number(11,8) as lat,

 lng||' '||lat as coord_pair

from @trails_parquet

(file_format => ff_parquet)

order by point_id;

Create or replace external table  T_cherry_creek_trail (

   point_id number as ($1:sequence_1::number),

  trail_name  varchar(50) as  ($1:trail_name::varchar),

  lng  number(11,8) as ($1:latitude::number(11,8)) ,

   lat number(11,8) as  ($1:longitude::number(11,8)),

 coord_pair varchar(50) as  (lng::varchar||' '||lat::varchar  )

 )

location= @trails_parquet

auto_refresh=true

file_format = ff_parquet

create or replace external table mels_smoothie_challenge_db.trails.T_CHERRY_CREEK_TRAIL(

               POINT_ID number as ($1:sequence_1::number),

               TRAIL_NAME varchar(50) as  ($1:trail_name::varchar),

               LNG number(11,8) as ($1:latitude::number(11,8)),

               LAT number(11,8) as ($1:longitude::number(11,8)),

               COORD_PAIR varchar(50) as (lng::varchar||' '||lat::varchar)

)

location= @mels_smoothie_challenge_db.trails.trails_parquet

auto_refresh = true

file_format = mels_smoothie_challenge_db.trails.ff_parquet;

select get_ddl('view','mels_smoothie_challenge_db.trails.v_cherry_creek_trail');

select row_count

     from mels_smoothie_challenge_db.information_schema.tables

     where table_schema = 'TRAILS'

    and table_name = 'SMV_CHERRY_CREEK_TRAIL'

    create secure materialized view SMV_CHERRY_CREEK_TRAIL

    — comment = '<comment>'

    as select * from T_CHERRY_CREEK_TRAIL

    select DEMO_DB.PUBLIC.GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

  SELECT

  'DLKW09' as step

  ,(select row_count

     from mels_smoothie_challenge_db.information_schema.tables

     where table_schema = 'TRAILS'

    and table_name = 'SMV_CHERRY_CREEK_TRAIL')  

   as actual

  ,3526 as expected

  ,'Secure Materialized View Created' as description

 );

 SELECT GET_DDL('DATABASE', 'mels_smoothie_challenge_db')

;

use role sysadmin;

create or replace table GARDEN_PLANTS.VEGGIES.ROOT_DEPTH (

   ROOT_DEPTH_ID number(1),

   ROOT_DEPTH_CODE text(1),

   ROOT_DEPTH_NAME text(7),

   UNIT_OF_MEASURE text(2),

   RANGE_MIN number(2),

   RANGE_MAX number(2)

   );

   USE WAREHOUSE COMPUTE_WH;

INSERT INTO ROOT_DEPTH (

               ROOT_DEPTH_ID ,

               ROOT_DEPTH_CODE ,

               ROOT_DEPTH_NAME ,

               UNIT_OF_MEASURE ,

               RANGE_MIN ,

               RANGE_MAX

)

VALUES

(

    1,

    'S',

    'Shallow',

    'cm',

    30,

    45

)

;

SELECT *

FROM ROOT_DEPTH

LIMIT 1;

USE WAREHOUSE COMPUTE_WH;

INSERT INTO ROOT_DEPTH (

               ROOT_DEPTH_ID ,

               ROOT_DEPTH_CODE ,

               ROOT_DEPTH_NAME ,

               UNIT_OF_MEASURE ,

               RANGE_MIN ,

               RANGE_MAX

)

VALUES

(

    3,

    'S',

    'Shallow',

    'cm',

    30,

    45

)

;

create table vegetable_details

(

plant_name varchar(25)

, root_depth_code varchar(1)   

);

select * from vegetable_details order by “Plant Name” asc

delete from vegetable_details where “Rooting Depth” ='D' and “Plant Name”='Spinach'

TRUNCATE TABLE GARDEN_PLANTS.VEGGIES.VEGETABLE_DETAILS;

SELECT * FROM GARDEN_PLANTS.VEGGIES.VEGETABLE_DETAILS BEFORE(STATEMENT => '01aa9e7b-0000-fe6f-0003-7b52000192ce');

SELECT * FROM my_table AT(TIMESTAMP => TO_TIMESTAMP(1432669154242, 3));

select * from vegetable_details order by “Plant Name” asc

create file format garden_plants.veggies.PIPECOLSEP_ONEHEADROW

    TYPE = 'CSV'–csv is used for any flat file (tsv, pipe-separated, etc)

    FIELD_DELIMITER = '|' –pipes as column separators

    SKIP_HEADER = 1 –one header row to skip

    ;

create file format garden_plants.veggies.COMMASEP_DBLQUOT_ONEHEADROW

    TYPE = 'CSV'–csv for comma separated files

    SKIP_HEADER = 1 –one header row 

    FIELD_OPTIONALLY_ENCLOSED_BY = '”' –this means that some values will be wrapped in double-quotes bc they have commas in them

;

use role accountadmin;

create or replace api integration dora_api_integration

api_provider = aws_api_gateway

api_aws_role_arn = 'arn:aws:iam::321463406630:role/snowflakeLearnerAssumedRole'

enabled = true

api_allowed_prefixes = ('https://awy6hshxy4.execute-api.us-west-2.amazonaws.com/dev/edu_dora&#8217;);

show  integrations;

 use role accountadmin; 

create or replace external function demo_db.public.grader(

      step varchar

    , passed boolean

    , actual integer

    , expected integer

    , description varchar)

returns variant

api_integration = dora_api_integration

context_headers = (current_timestamp,current_account, current_statement)

as 'https://awy6hshxy4.execute-api.us-west-2.amazonaws.com/dev/edu_dora/grader&#8217;

;

use role accountadmin;

use database demo_db; –change this to a different database if you prefer

use schema public; –change this to a different schema if you prefer

select grader(step, (actual = expected), actual, expected, description) as graded_results from

(SELECT

 'DORA_IS_WORKING' as step

 ,(select 123) as actual

 ,123 as expected

 ,'Dora is working!' as description

);

SELECT *

FROM GARDEN_PLANTS.INFORMATION_SCHEMA.SCHEMATA;

SELECT *

FROM GARDEN_PLANTS.INFORMATION_SCHEMA.SCHEMATA

where schema_name in ('FLOWERS','FRUITS','VEGGIES');

SELECT count(*) as SCHEMAS_FOUND, '3' as SCHEMAS_EXPECTED

FROM GARDEN_PLANTS.INFORMATION_SCHEMA.SCHEMATA

where schema_name in ('FLOWERS','FRUITS','VEGGIES');

use database DEMO_DB;

use schema PUBLIC;

use role ACCOUNTADMIN;

–Do NOT EDIT ANYTHING BELOW THIS LINE

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT

 'DWW01' as step

 ,( select count(*) 

   from GARDEN_PLANTS.INFORMATION_SCHEMA.SCHEMATA

   where schema_name in ('FLOWERS','VEGGIES','FRUITS')) as actual

  ,3 as expected

  ,'Created 3 Garden Plant schemas' as description

);

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW02' as step

 ,( select count(*)

   from GARDEN_PLANTS.INFORMATION_SCHEMA.SCHEMATA

   where schema_name = 'PUBLIC') as actual

 , 0 as expected

 ,'Deleted PUBLIC schema.' as description

);

–Do NOT EDIT ANYTHING BELOW THIS LINE

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW03' as step

 ,( select count(*)

   from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

   where table_name = 'ROOT_DEPTH') as actual

 , 1 as expected

 ,'ROOT_DEPTH Table Exists' as description

);

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW04' as step

 ,( select count(*) as SCHEMAS_FOUND

   from UTIL_DB.INFORMATION_SCHEMA.SCHEMATA) as actual

 , 2 as expected

 , 'UTIL_DB Schemas' as description

);

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW05' as step

 ,( select count(*)

   from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

   where table_name = 'VEGETABLE_DETAILS') as actual

 , 1 as expected

 ,'VEGETABLE_DETAILS Table' as description

);

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW06' as step

,( select row_count

  from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

  where table_name = 'ROOT_DEPTH') as actual

, 3 as expected

,'ROOT_DEPTH row count' as description

); 

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW07' as step

 ,( select row_count

   from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

   where table_name = 'VEGETABLE_DETAILS') as actual

 , 41 as expected

 , 'VEG_DETAILS row count' as description

);

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

   SELECT 'DWW08' as step

   ,( select count(*)

     from GARDEN_PLANTS.INFORMATION_SCHEMA.FILE_FORMATS

     where FIELD_DELIMITER =','

     and FIELD_OPTIONALLY_ENCLOSED_BY ='”') as actual

   , 1 as expected

   , 'File Format 1 Exists' as description

);

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW09' as step

 ,( select count(*)

   from GARDEN_PLANTS.INFORMATION_SCHEMA.FILE_FORMATS

   where FIELD_DELIMITER ='|'

   ) as actual

 , 1 as expected

 ,'File Format 2 Exists' as description

);

 create stage garden_plants.veggies.like_a_window_into_an_s3_bucket

 url = 's3://uni-lab-files';

list @like_a_window_into_an_s3_bucket;

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

 SELECT 'DWW10' as step

  ,( select count(*)

    from GARDEN_PLANTS.INFORMATION_SCHEMA.stages

    where stage_url='s3://uni-lab-files'

    and stage_type='External Named') as actual

  , 1 as expected

  , 'External stage created' as description

 );

 create or replace table garden_plants.veggies.vegetable_details_soil_type

( plant_name varchar(25)

 ,soil_type number(1,0)

);

copy into garden_plants.veggies.vegetable_details_soil_type

from @like_a_window_into_an_s3_bucket

files = ( 'VEG_NAME_TO_SOIL_TYPE_PIPE.txt')

file_format = ( format_name='PIPECOLSEP_ONEHEADROW' );

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

  SELECT 'DWW11' as step

  ,( select row_count

    from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

    where table_name = 'VEGETABLE_DETAILS_SOIL_TYPE') as actual

  , 42 as expected

  , 'Veg Det Soil Type Count' as description

 );

 –The data in the file, with no FILE FORMAT specified

select $1

from @garden_plants.veggies.like_a_window_into_an_s3_bucket/LU_SOIL_TYPE.tsv;

–Same file but with one of the file formats we created earlier 

select $1, $2, $3

from @garden_plants.veggies.like_a_window_into_an_s3_bucket/LU_SOIL_TYPE.tsv
(file_format => garden_plants.veggies.COMMASEP_DBLQUOT_ONEHEADROW);

–Same file but with the other file format we created earlier

select $1, $2, $3

from @garden_plants.veggies.like_a_window_into_an_s3_bucket/LU_SOIL_TYPE.tsv
(file_format => garden_plants.veggies.TABCOLSEP_ONEHEADROW );

create file format garden_plants.veggies.L8_CHALLENGE_FF

    TYPE = 'CSV'–csv is used for any flat file (tsv, pipe-separated, etc)

    FIELD_DELIMITER = '   ' –pipes as column separators

    SKIP_HEADER = 1 –one header row to skip

    ;

    create or replace table garden_plants.veggies.LU_SOIL_TYPE(

SOIL_TYPE_ID number,  

SOIL_TYPE varchar(15),

SOIL_DESCRIPTION varchar(75)

 );

 copy into garden_plants.veggies.LU_SOIL_TYPE

from @like_a_window_into_an_s3_bucket

files = ( 'LU_SOIL_TYPE.tsv')

file_format = ( format_name='TABCOLSEP_ONEHEADROW' );

select * from garden_plants.veggies.LU_SOIL_TYPE

create or replace table VEGETABLE_DETAILS_PLANT_HEIGHT

(

plant_name varchar(75),

UOM varchar(1),

Low_End_of_Range number,

High_End_of_Range number

)

 copy into garden_plants.veggies.VEGETABLE_DETAILS_PLANT_HEIGHT

from @like_a_window_into_an_s3_bucket

files = ( 'veg_plant_height.csv')

file_format = ( format_name='COMMASEP_DBLQUOT_ONEHEADROW' );

select * from VEGETABLE_DETAILS_PLANT_HEIGHT

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from ( 

      SELECT 'DWW12' as step

      ,( select row_count

        from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

        where table_name = 'VEGETABLE_DETAILS_PLANT_HEIGHT') as actual

      , 41 as expected

      , 'Veg Detail Plant Height Count' as description  

);

–Set your worksheet drop list role to ACCOUNTADMIN

–Set your worksheet drop list database and schema to the location of your GRADER function

— DO NOT EDIT ANYTHING BELOW THIS LINE. THE CODE MUST BE RUN EXACTLY AS IT IS WRITTEN

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from ( 

     SELECT 'DWW13' as step

     ,( select row_count

       from GARDEN_PLANTS.INFORMATION_SCHEMA.TABLES

       where table_name = 'LU_SOIL_TYPE') as actual

     , 8 as expected

     ,'Soil Type Look Up Table' as description  

);

— Set your worksheet drop lists

— DO NOT EDIT THE CODE

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (

     SELECT 'DWW14' as step

     ,( select count(*)

       from GARDEN_PLANTS.INFORMATION_SCHEMA.FILE_FORMATS

       where FILE_FORMAT_NAME='L8_CHALLENGE_FF'

       and FIELD_DELIMITER = '\t') as actual

     , 1 as expected

     ,'Challenge File Format Created' as description 

);

use role sysadmin;

// Create a new database and set the context to use the new database

CREATE DATABASE LIBRARY_CARD_CATALOG COMMENT = 'DWW Lesson 9 ';

USE DATABASE LIBRARY_CARD_CATALOG;

// Create Author table

CREATE OR REPLACE TABLE AUTHOR (

   AUTHOR_UID NUMBER

  ,FIRST_NAME VARCHAR(50)

  ,MIDDLE_NAME VARCHAR(50)

  ,LAST_NAME VARCHAR(50)

);

// Insert the first two authors into the Author table

INSERT INTO AUTHOR(AUTHOR_UID,FIRST_NAME,MIDDLE_NAME, LAST_NAME)

Values

(1, 'Fiona', ”,'Macdonald')

,(2, 'Gian','Paulo','Faleschini');

// Look at your table with it's new rows

SELECT *

FROM AUTHOR;

use role sysadmin;

//See how the nextval function works

SELECT SEQ_AUTHOR_UID.nextval;

show sequences

select seq_author_UID.nextval,seq_author_UID.nextval;

use role sysadmin;

//Drop and recreate the counter (sequence) so that it starts at 3

// then we'll add the other author records to our author table

CREATE OR REPLACE SEQUENCE “LIBRARY_CARD_CATALOG”.”PUBLIC”.”SEQ_AUTHOR_UID”

START 3

INCREMENT 1

COMMENT = 'Use this to fill in the AUTHOR_UID every time you add a row';

//Add the remaining author records and use the nextval function instead

//of putting in the numbers

INSERT INTO AUTHOR(AUTHOR_UID,FIRST_NAME,MIDDLE_NAME, LAST_NAME)

Values

(SEQ_AUTHOR_UID.nextval, 'Laura', 'K','Egendorf')

,(SEQ_AUTHOR_UID.nextval, 'Jan', ”,'Grover')

,(SEQ_AUTHOR_UID.nextval, 'Jennifer', ”,'Clapp')

,(SEQ_AUTHOR_UID.nextval, 'Kathleen', ”,'Petelinsek');

select * from AUTHOR

USE DATABASE LIBRARY_CARD_CATALOG;

// Create a new sequence, this one will be a counter for the book table

CREATE OR REPLACE SEQUENCE “LIBRARY_CARD_CATALOG”.”PUBLIC”.”SEQ_BOOK_UID”

START 1

INCREMENT 1

COMMENT = 'Use this to fill in the BOOK_UID everytime you add a row';

// Create the book table and use the NEXTVAL as the

// default value each time a row is added to the table

CREATE OR REPLACE TABLE BOOK

( BOOK_UID NUMBER DEFAULT SEQ_BOOK_UID.nextval

 ,TITLE VARCHAR(50)

 ,YEAR_PUBLISHED NUMBER(4,0)

);

// Insert records into the book table

// You don't have to list anything for the

// BOOK_UID field because the default setting

// will take care of it for you

INSERT INTO BOOK(TITLE,YEAR_PUBLISHED)

VALUES

 ('Food',2001)

,('Food',2006)

,('Food',2008)

,('Food',2016)

,('Food',2015);

// Create the relationships table

// this is sometimes called a “Many-to-Many table”

CREATE TABLE BOOK_TO_AUTHOR

(  BOOK_UID NUMBER

  ,AUTHOR_UID NUMBER

);

//Insert rows of the known relationships

INSERT INTO BOOK_TO_AUTHOR(BOOK_UID,AUTHOR_UID)

VALUES

 (1,1) // This row links the 2001 book to Fiona Macdonald

,(1,2) // This row links the 2001 book to Gian Paulo Faleschini

,(2,3) // Links 2006 book to Laura K Egendorf

,(3,4) // Links 2008 book to Jan Grover

,(4,5) // Links 2016 book to Jennifer Clapp

,(5,6);// Links 2015 book to Kathleen Petelinsek

//Check your work by joining the 3 tables together

//You should get 1 row for every author

select *

from book_to_author ba

join author a

on ba.author_uid = a.author_uid

join book b

on b.book_uid=ba.book_uid;

— Set your worksheet drop lists

— DO NOT EDIT THE CODE

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from ( 

     SELECT 'DWW15' as step

     ,( select count(*)

      from LIBRARY_CARD_CATALOG.PUBLIC.Book_to_Author ba

      join LIBRARY_CARD_CATALOG.PUBLIC.author a

      on ba.author_uid = a.author_uid

      join LIBRARY_CARD_CATALOG.PUBLIC.book b

      on b.book_uid=ba.book_uid) as actual

     , 6 as expected

     , '3NF DB was Created.' as description 

);

// JSON DDL Scripts

USE LIBRARY_CARD_CATALOG;

// Create an Ingestion Table for JSON Data

CREATE TABLE LIBRARY_CARD_CATALOG.PUBLIC.AUTHOR_INGEST_JSON

(

  RAW_AUTHOR VARIANT

);

//Create File Format for JSON Data@list

CREATE FILE FORMAT JSON_FILE_FORMAT

TYPE = 'JSON'

COMPRESSION = 'AUTO'

ENABLE_OCTAL = TRUE

ALLOW_DUPLICATE = TRUE 

STRIP_OUTER_ARRAY =TRUE

STRIP_NULL_VALUES = TRUE

IGNORE_UTF8_ERRORS = TRUE ;

 create stage like_a_window_into_an_s3_bucket

 url = 's3://uni-lab-files';

list @like_a_window_into_an_s3_bucket;

copy into LIBRARY_CARD_CATALOG.PUBLIC.AUTHOR_INGEST_JSON

from @like_a_window_into_an_s3_bucket

files = ( 'author_with_header.json')

file_format = ( format_name='JSON_FILE_FORMAT' );

select * from AUTHOR_INGEST_JSON

//returns AUTHOR_UID value from top-level object's attribute

select raw_author:AUTHOR_UID

from author_ingest_json;

//returns the data in a way that makes it look like a normalized table

SELECT

 raw_author:AUTHOR_UID

,raw_author:FIRST_NAME::STRING as FIRST_NAME

,raw_author:MIDDLE_NAME::STRING as MIDDLE_NAME

,raw_author:LAST_NAME::STRING as LAST_NAME

FROM AUTHOR_INGEST_JSON;

— Set your worksheet drop lists. DO NOT EDIT THE DORA CODE.

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

  SELECT 'DWW16' as step

  ,( select row_count

    from LIBRARY_CARD_CATALOG.INFORMATION_SCHEMA.TABLES

    where table_name = 'AUTHOR_INGEST_JSON') as actual

  ,6 as expected

  ,'Check number of rows' as description

 );

 CREATE OR REPLACE TABLE LIBRARY_CARD_CATALOG.PUBLIC.NESTED_INGEST_JSON

(

  “RAW_NESTED_BOOK” VARIANT

);

asset+block/json_book_author_nested.txt
copy into LIBRARY_CARD_CATALOG.PUBLIC.NESTED_INGEST_JSON

from @like_a_window_into_an_s3_bucket

files = ( 'json_book_author_nested.txt')

file_format = ( format_name='JSON_FILE_FORMAT' );

//a few simple queries

SELECT RAW_NESTED_BOOK

FROM NESTED_INGEST_JSON;

SELECT RAW_NESTED_BOOK:year_published

FROM NESTED_INGEST_JSON;

SELECT RAW_NESTED_BOOK:authors

FROM NESTED_INGEST_JSON;

//try changing the number in the brackets to return authors from a different row

SELECT RAW_NESTED_BOOK:authors[0].first_name

FROM NESTED_INGEST_JSON;

//Use these example flatten commands to explore flattening the nested book and author data

SELECT value:first_name

FROM NESTED_INGEST_JSON

,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors);

SELECT value:first_name

FROM NESTED_INGEST_JSON

,table(flatten(RAW_NESTED_BOOK:authors));

//Add a CAST command to the fields returned

SELECT value:first_name::VARCHAR, value:last_name::VARCHAR

FROM NESTED_INGEST_JSON

,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors);

//Assign new column  names to the columns using “AS”

SELECT value:first_name::VARCHAR AS FIRST_NM

, value:last_name::VARCHAR AS LAST_NM

FROM NESTED_INGEST_JSON

,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors);

— Set your worksheet drop lists. DO NOT EDIT THE DORA CODE.

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from (  

     SELECT 'DWW17' as step

      ,( select row_count

        from LIBRARY_CARD_CATALOG.INFORMATION_SCHEMA.TABLES

        where table_name = 'NESTED_INGEST_JSON') as actual

      , 5 as expected

      ,'Check number of rows' as description 

);

//Create a new database to hold the Twitter file

CREATE DATABASE SOCIAL_MEDIA_FLOODGATES

COMMENT = 'There\'s so much data from social media – flood warning';

USE DATABASE SOCIAL_MEDIA_FLOODGATES;

//Create a table in the new database
CREATE TABLE SOCIAL_MEDIA_FLOODGATES.PUBLIC.TWEET_INGEST

(“RAW_STATUS” VARIANT)

COMMENT = 'Bring in tweets, one row per tweet or status entity';


//Create a JSON file format in the new database

CREATE FILE FORMAT SOCIAL_MEDIA_FLOODGATES.PUBLIC.JSON_FILE_FORMAT

TYPE = 'JSON'

COMPRESSION = 'AUTO'

ENABLE_OCTAL = FALSE

ALLOW_DUPLICATE = FALSE

STRIP_OUTER_ARRAY = TRUE

STRIP_NULL_VALUES = FALSE

IGNORE_UTF8_ERRORS = FALSE;

 create stage like_a_window_into_an_s3_bucket

 url = 's3://uni-lab-files';

list @like_a_window_into_an_s3_bucket;

copy into SOCIAL_MEDIA_FLOODGATES.PUBLIC.TWEET_INGEST

from @like_a_window_into_an_s3_bucket

files = ( 'nutrition_tweets.json')

file_format = ( format_name='JSON_FILE_FORMAT' );

//select statements as seen in the video

SELECT RAW_STATUS

FROM TWEET_INGEST;

SELECT RAW_STATUS:entities

FROM TWEET_INGEST;

SELECT RAW_STATUS:entities:hashtags

FROM TWEET_INGEST;

//Explore looking at specific hashtags by adding bracketed numbers

//This query returns just the first hashtag in each tweet

SELECT RAW_STATUS:entities:hashtags[0].text

FROM TWEET_INGEST;

//This version adds a WHERE clause to get rid of any tweet that

//doesn't include any hashtags

SELECT RAW_STATUS:entities:hashtags[0].text

FROM TWEET_INGEST

WHERE RAW_STATUS:entities:hashtags[0].text is not null;

//Perform a simple CAST on the created_at key

//Add an ORDER BY clause to sort by the tweet's creation date

SELECT RAW_STATUS:created_at::DATE

FROM TWEET_INGEST

ORDER BY RAW_STATUS:created_at::DATE;

//Flatten statements that return the whole hashtag entity

SELECT value

FROM TWEET_INGEST

,LATERAL FLATTEN

(input => RAW_STATUS:entities:hashtags);

SELECT value

FROM TWEET_INGEST

,TABLE(FLATTEN(RAW_STATUS:entities:hashtags));

//Flatten statement that restricts the value to just the TEXT of the hashtag

SELECT value:text

FROM TWEET_INGEST

,LATERAL FLATTEN

(input => RAW_STATUS:entities:hashtags);

//Flatten and return just the hashtag text, CAST the text as VARCHAR

SELECT value:text::VARCHAR

FROM TWEET_INGEST

,LATERAL FLATTEN

(input => RAW_STATUS:entities:hashtags);

//Flatten and return just the hashtag text, CAST the text as VARCHAR

// Use the AS command to name the column

SELECT value:text::VARCHAR AS THE_HASHTAG

FROM TWEET_INGEST

,LATERAL FLATTEN

(input => RAW_STATUS:entities:hashtags);

//Add the Tweet ID and User ID to the returned table

SELECT RAW_STATUS:user:id AS USER_ID

,RAW_STATUS:id AS TWEET_ID

,value:text::VARCHAR AS HASHTAG_TEXT

FROM TWEET_INGEST

,LATERAL FLATTEN

(input => RAW_STATUS:entities:hashtags);

— Set your worksheet drop lists. DO NOT EDIT THE DORA CODE.

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

   SELECT 'DWW18' as step

  ,( select row_count

    from SOCIAL_MEDIA_FLOODGATES.INFORMATION_SCHEMA.TABLES

    where table_name = 'TWEET_INGEST') as actual

  , 9 as expected

  ,'Check number of rows' as description 

 );

 create or replace view SOCIAL_MEDIA_FLOODGATES.PUBLIC.HASHTAGS_NORMALIZED as

(SELECT RAW_STATUS:user:id AS USER_ID

,RAW_STATUS:id AS TWEET_ID

,value:text::VARCHAR AS HASHTAG_TEXT

FROM TWEET_INGEST

,LATERAL FLATTEN

(input => RAW_STATUS:entities:hashtags)

);

— Set your worksheet drop lists. DO NOT EDIT THE DORA CODE.

select GRADER(step, (actual = expected), actual, expected, description) as graded_results from

(

  SELECT 'DWW19' as step

  ,( select count(*)

    from SOCIAL_MEDIA_FLOODGATES.INFORMATION_SCHEMA.VIEWS
    where table_name = 'HASHTAGS_NORMALIZED') as actual

  , 1 as expected

  ,'Check number of rows' as description

 );

create sequence SEQ_AUTHOR_UID

 start = 1

 increment = 1

 comment = 'Use this to fill in AUTHOR_UID';

— This SQL file is for the Hands On Lab Guide for the 30-day free Snowflake trial account (Enterprise edition)

— The numbers below correspond to the sections of the Lab Guide in which SQL is to be run in a Snowflake worksheet

/* *********************************************************************************** */

/* *** MODULE 1  ********************************************************************* */

/* *********************************************************************************** */

— 1.2.1 Create a virtual warehouse cluster

use role SYSADMIN;

create or replace warehouse LOAD_WH with

  warehouse_size = 'xlarge'

  auto_suspend = 300

  initially_suspended = true;

use warehouse LOAD_WH;

— 1.2.2 Create the new empty CITIBIKE_LAB database

create or replace database CITIBIKE_LAB;

create or replace schema DEMO;

create or replace schema UTILS;

— 1.2.3  Create an API integration to support creating an external function call to a REST API

use schema UTILS;

use role accountadmin;

— Create an API Integration object

create or replace api integration fetch_http_data

  api_provider = aws_api_gateway

  api_aws_role_arn = 'arn:aws:iam::148887191972:role/ExecuteLambdaFunction'

  enabled = true

  api_allowed_prefixes = ('https://dr14z5kz5d.execute-api.us-east-1.amazonaws.com/prod/fetchhttpdata&#8217;);

grant usage on integration fetch_http_data to role sysadmin;

— 1.2.4

— Now create the external function that uses the API integration object

use role sysadmin;

— create the function

create or replace external function utils.fetch_http_data(v varchar)

    returns variant

    api_integration = fetch_http_data

    as 'https://dr14z5kz5d.execute-api.us-east-1.amazonaws.com/prod/fetchhttpdata&#8217;;

— 1.2.5 Create a few reference tables and populate them with data.

use schema DEMO;

create or replace table GBFS_JSON (

  data     varchar,

  url        varchar,

  payload              variant,

  row_inserted    timestamp_ltz);

— Populate it with raw JSON data through the External Function call

insert into GBFS_JSON

select

  $1 data,

  $2 url,

  utils.fetch_http_data( url ) payload,

  current_timestamp() row_inserted

from

  (values

    ('stations', 'https://gbfs.citibikenyc.com/gbfs/en/station_information.json&#8217;),

    ('regions', 'https://gbfs.citibikenyc.com/gbfs/en/system_regions.json&#8217;));

— Now refine that raw JSON data by extracting out the STATIONS nodes

create or replace table STATION_JSON as

with s as (

  select payload, row_inserted

    from gbfs_json

   where data = 'stations'

     and row_inserted = (select max(row_inserted) from gbfs_json)

  )

select

  value station_v,

  payload:response.last_updated::timestamp last_updated,

  row_inserted

from s,

  lateral flatten (input => payload:response.data.stations) ;

— extract the individual region records

create or replace table REGION_JSON as

with r as (

  select payload, row_inserted

    from gbfs_json

   where data = 'regions'

     and row_inserted = (select max(row_inserted) from gbfs_json)

  )

select

  value region_v,

  payload:response.last_updated::timestamp last_updated,

  row_inserted

from r,

  lateral flatten (input => payload:response.data.regions);

— Lastly, create a view that “flattens” the JSON into a standard table structure

create or replace view STATIONS_VW as

with s as (

  select * from station_json

   where row_inserted = (select max(row_inserted) from station_json)

   ),

     r as (

  select * from region_json

   where row_inserted = (select max(row_inserted) from region_json))

select

  station_v:station_id::number   station_id,

  station_v:name::string         station_name,

  station_v:lat::float           station_lat,

  station_v:lon::float           station_lon,

  station_v:station_type::string station_type,

  station_v:capacity::number     station_capacity,

  station_v:rental_methods       rental_methods,

  region_v:name::string          region_name

from s

  left outer join r

    on station_v:region_id::integer = region_v:region_id::integer ;

select * from CITIBIKE_LAB.DEMO.GBFS_JSON  Limit 10

select * from CITIBIKE_LAB.DEMO.REGION_JSON Limit 10

select * from CITIBIKE_LAB.DEMO.STATION_JSON Limit 10

select * from CITIBIKE_LAB.DEMO.STATIONS_VW –Limit 10

/* *********************************************************************************** */

/* *** MODULE 2  ********************************************************************* */

/* *********************************************************************************** */

— 2.1.1  Set context

use schema demo;

use role SYSADMIN;

create or replace stage CITIBIKE_STAGE

  url = 's3://sfquickstarts/VHOL Snowflake for Data Lake/'

  file_format=(type=parquet);

— 2.1.3 Let's see what data is available:

— Show the list of files in the external stage

list @citibike_stage/Data/2015;

— Let's take a peek inside the files themselves

select $1 from @citibike_stage/Data/2015 limit 100;

— 2.1.4 Create a basic External Table on the Parquet files

—       in the bucket

create or replace file format citibike_parquet_ff

  type = parquet;

create or replace external table TRIPS_BASIC_XT

  location = @citibike_stage/Data

  auto_refresh = false

  file_format=(format_name=citibike_parquet_ff);

— Take a look at the raw data, and use the metadata$filename

— pseudocolumn to see which row lives in which source file.

  select metadata$filename, value

    from TRIPS_BASIC_XT

    LIMIT 100;

— 2.1.5

— Create a new external table on the same set of Parquet

— files, but this time we'll define each column

— separately, and partition the files on the date

— portion of their folder names. We'll use Snowflake's

— infer_schema and GENERATE_COLUMN_DESCRIPTION functionality

— to help us create the table definition

select *

from table(

  infer_schema(

    location=>'@citibike_stage/Data'

    , file_format=>'citibike_parquet_ff'

    )

  );

— Use the INFER_SCHEMA and GENERATE_COLUMN_DESCRIPTION

— functions to build the CREATE EXTERNAL TABLE statement, which

— we will then customize.

SELECT

  $$ CREATE OR REPLACE EXTERNAL TABLE FOO ($$ || (

      SELECT

      GENERATE_COLUMN_DESCRIPTION(ARRAY_AGG(OBJECT_CONSTRUCT(*))

        , 'EXTERNAL_TABLE') ||

        $$) LOCATION = @citibike_stage/Data FILE_FORMAT = my_parquet_format; $$

      FROM

        TABLE (

          INFER_SCHEMA(

            LOCATION => '@citibike_stage/Data',

            FILE_FORMAT => 'citibike_parquet_ff'

          )

        )

    );

— Output of the command above

—

— CREATE OR REPLACE EXTERNAL TABLE FOO (“BIRTH_YEAR” NUMBER(4, 0) AS ($1:BIRTH_YEAR::NUMBER(4, 0)),

— “PROGRAM_ID” NUMBER(4, 0) AS ($1:PROGRAM_ID::NUMBER(4, 0)),

— “TRIPDURATION” NUMBER(9, 0) AS ($1:TRIPDURATION::NUMBER(9, 0)),

— “START_STATION_ID” NUMBER(4, 0) AS ($1:START_STATION_ID::NUMBER(4, 0)),

— “STOPTIME” TIMESTAMP_NTZ AS ($1:STOPTIME::TIMESTAMP_NTZ),

— “END_STATION_ID” NUMBER(4, 0) AS ($1:END_STATION_ID::NUMBER(4, 0)),

— “GENDER” NUMBER(2, 0) AS ($1:GENDER::NUMBER(2, 0)),

— “USERTYPE” TEXT AS ($1:USERTYPE::TEXT),

— “STARTTIME” TIMESTAMP_NTZ AS ($1:STARTTIME::TIMESTAMP_NTZ),

— “BIKEID” NUMBER(9, 0) AS ($1:BIKEID::NUMBER(9, 0)))

— LOCATION=@citibike_stage/Data

— FILE_FORMAT='citibike_parquet_ff';

CREATE OR REPLACE EXTERNAL TABLE TRIPS_BIG_XT (

“BIRTH_YEAR” NUMBER(4, 0) AS ($1:BIRTH_YEAR::NUMBER(4, 0)),

“PROGRAM_ID” NUMBER(4, 0) AS ($1:PROGRAM_ID::NUMBER(4, 0)),

“TRIPDURATION” NUMBER(9, 0) AS ($1:TRIPDURATION::NUMBER(9, 0)),

    STARTDATE    date as

    to_date(split_part(metadata$filename, '/', 3) || '-' || split_part(metadata$filename, '/', 4) || '-01'),

“START_STATION_ID” NUMBER(4, 0) AS ($1:START_STATION_ID::NUMBER(4, 0)),

“STOPTIME” TIMESTAMP_NTZ AS ($1:STOPTIME::TIMESTAMP_NTZ),

“END_STATION_ID” NUMBER(4, 0) AS ($1:END_STATION_ID::NUMBER(4, 0)),

“GENDER” NUMBER(2, 0) AS ($1:GENDER::NUMBER(2, 0)),

“USERTYPE” TEXT AS ($1:USERTYPE::TEXT),

“STARTTIME” TIMESTAMP_NTZ AS ($1:STARTTIME::TIMESTAMP_NTZ),

“BIKEID” NUMBER(9, 0) AS ($1:BIKEID::NUMBER(9, 0))

)

partition by (startdate)

    location = @citibike_stage/Data

    auto_refresh = false

    file_format=(format_name=citibike_parquet_ff);

— 2.1.6  Let's see what the data looks like in the new

—        external table.

select * from trips_big_xt limit 100;

— 2.1.7  Add the partition column into the query for effective pruning

select

  start_station_id,

  count(*) num_trips,

  avg(tripduration) avg_duration

from trips_big_xt

where startdate between to_date('2014-01-01') and to_date('2014-06-30')

group by 1;

— 2.1.8  External tables act like regular tables,

—        so we can join them with other tables

with t as (

  select

     start_station_id,

     end_station_id,

     count(*) num_trips

  from trips_big_xt

  where startdate between to_date('2014-01-01') and to_date('2014-12-30')

    group by 1, 2)

select

   ss.station_name start_station,

   es.station_name end_station,

   num_trips

 from t inner join stations_vw ss on t.start_station_id = ss.station_id

        inner join stations_vw es on t.end_station_id = es.station_id

order by 3 desc;

/* *********************************************************************************** */

/* *** MODULE 3  ********************************************************************* */

/* *********************************************************************************** */

—  3.1.1  Create the materialized view

use role SYSADMIN;

use schema DEMO;

create or replace materialized view TRIPS_MV as

select

  startdate,

  start_station_id,

  end_station_id,

  count(*) num_trips

from trips_big_xt

group by 1, 2, 3;

select

  count(*)                           num_rows,

  sum(num_trips)             num_trips

from trips_mv;

— 3.1.2 Let's re-run our join query, replacing the

—       external table with the new materialized view.

with t as (

  select

    start_station_id,

    end_station_id,

    sum(num_trips) num_trips

  from trips_mv

  where startdate between to_date('2014-01-01') and to_date('2014-12-30')

  group by 1, 2)

select

  ss.station_name start_station,

  es.station_name end_station,

  num_trips

from t

  inner join stations_vw ss

     on t.start_station_id = ss.station_id

  inner join stations_vw es

     on t.end_station_id = es.station_id

order by 3 desc;

/* *********************************************************************************** */

/* *** MODULE 4  ********************************************************************* */

/* *********************************************************************************** */

— 4.1.1 Create an external stage where documents are stored.

create or replace stage documents

 url = 's3://sfquickstarts/VHOL Snowflake for Data Lake/PDF/'

 directory = (enable = true auto_refresh = false);

alter stage documents refresh;

ls @documents;

— 4.1.2 search the Directory Table for files with algorithm in

—       the name

select *

from directory(@documents)

where file_url ilike '%algorithm%pdf';

–4.2.1 Query directory table using scoped URL

select

  build_scoped_file_url(@documents, relative_path) as scoped_file_url

from directory(@documents);

— 4.2.1 Create external stage to import PDFBox from S3

create or replace stage jars_stage

 url = 's3://sfquickstarts/Common JARs/'

 directory = (enable = true auto_refresh = false);

— 4.2.2 Create a java function to parse PDF files

create or replace function read_pdf(file string)

returns string

language java

imports = ('@jars_stage/pdfbox-app-2.0.24.jar')

HANDLER = 'PdfParser.ReadFile'

as

$$

import org.apache.pdfbox.pdmodel.PDDocument;

import org.apache.pdfbox.text.PDFTextStripper;

import org.apache.pdfbox.text.PDFTextStripperByArea;

import java.io.File;

import java.io.FileInputStream;

import java.io.IOException;

import java.io.InputStream;

public class PdfParser {

    public static String ReadFile(InputStream stream) throws IOException {

        try (PDDocument document = PDDocument.load(stream)) {

            document.getClass();

            if (!document.isEncrypted()) {

                PDFTextStripperByArea stripper = new PDFTextStripperByArea();

                stripper.setSortByPosition(true);

                PDFTextStripper tStripper = new PDFTextStripper();

                String pdfFileInText = tStripper.getText(document);

                return pdfFileInText;

            }

        }

        return null;

    }

}

$$;

— 4.2.3 Next, you can query the table and use the udf to parse the contents of

— the PDF as unstructured data.

select

  relative_path,

  file_url,

  read_pdf('@documents/' || relative_path)

from directory(@documents)

limit 5;

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

list @citibike_trips;  —Stage info

–create file format

create or replace file format csv type='csv'

  compression = 'auto' field_delimiter = ',' record_delimiter = '\n'

  skip_header = 0 field_optionally_enclosed_by = '\042' trim_space = false

  error_on_column_count_mismatch = false escape = 'none' escape_unenclosed_field = '\134'

  date_format = 'auto' timestamp_format = 'auto' null_if = (”) comment = 'file format for ingesting data for zero to snowflake';

  –verify file format is created

show file formats in database citibike;

copy into trips from @citibike_trips file_format=csv PATTERN = '.*csv.*' ;

show warehouses;

–verify table is clear

select * from trips limit 20;

select date_trunc('hour', starttime) as “date”,

count(*) as “num trips”,

avg(tripduration)/60 as “avg duration (mins)”,

avg(haversine(start_station_latitude, start_station_longitude, end_station_latitude, end_station_longitude)) as “avg distance (km)”

from trips

group by 1 order by 1;

–cache query-results in 51sec

select date_trunc('hour', starttime) as “date”,

count(*) as “num trips”,

avg(tripduration)/60 as “avg duration (mins)”,

avg(haversine(start_station_latitude, start_station_longitude, end_station_latitude, end_station_longitude)) as “avg distance (km)”

from trips

group by 1 order by 1;

select

monthname(starttime) as “month”,

count(*) as “num trips”

from trips

group by 1 order by 2 desc;

create table trips_dev clone trips; –clone trips to clone_dev

create database weather;

use role sysadmin;

use warehouse compute_wh;

use database weather;

use schema public;

create table json_weather_data (v variant);

–stage your s3 weather data

create stage nyc_weather

url = 's3://snowflake-workshop-lab/zero-weather-nyc';

list @nyc_weather;

–Load datat to Json weather table

copy into json_weather_data

from @nyc_weather

    file_format = (type = json strip_outer_array = true);

select * from json_weather_data limit 10;

// create a view that will put structure onto the semi-structured data

create or replace view json_weather_data_view as

select

    v:obsTime::timestamp as observation_time,

    v:station::string as station_id,

    v:name::string as city_name,

    v:country::string as country,

    v:latitude::float as city_lat,

    v:longitude::float as city_lon,

    v:weatherCondition::string as weather_conditions,

    v:coco::int as weather_conditions_code,

    v:temp::float as temp,

    v:prcp::float as rain,

    v:tsun::float as tsun,

    v:wdir::float as wind_dir,

    v:wspd::float as wind_speed,

    v:dwpt::float as dew_point,

    v:rhum::float as relative_humidity,

    v:pres::float as pressure

from

    json_weather_data

where

    station_id = '72502';

select * from json_weather_data_view

where date_trunc('month',observation_time) = '2018-01-01'

limit 20;

— cross DB join

select weather_conditions as conditions

,count(*) as num_trips

from citibike.public.trips
left outer join json_weather_data_view

on date_trunc('hour', observation_time) = date_trunc('hour', starttime)

where conditions is not null

group by 1 order by 2 desc;

——

–Time travel

drop table json_weather_data;

select * from json_weather_data limit 10;

—Restore table

undrop table json_weather_data;

–verify table is undropped

select * from json_weather_data limit 10;

—-mess data

update trips set start_station_name = 'oops';

select

start_station_name as “station”,

count(*) as “rides”

from trips

group by 1

order by 2 desc

limit 20;

—serach for last update query id to roll back

set query_id =

(select query_id from table(information_schema.query_history_by_session (result_limit=>5))

where query_text like 'update%' order by start_time desc limit 1);

—using timetravel to restore the table back to query id we set

create or replace table trips as

(select * from trips before (statement => '01aa5a5a-0000-f97d-0003-7b5200010146′));

select

start_station_name as “station”,

count(*) as “rides”

from trips

group by 1

order by 2 desc

limit 20;

create role junior_dba;

grant role junior_dba to user Test;
