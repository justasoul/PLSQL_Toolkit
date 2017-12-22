# DBMS_PROFILER

## Run a profiler run

  
```
  DECLARE
    l_result  BINARY_INTEGER;

    lv_profile_run_comment  PLSQL_PROFILER_RUNS.RUN_COMMENT%TYPE := upper('bn_test_run_002');
    ln_profile_runid        PLSQL_PROFILER_RUNS.RUNID%TYPE;
  BEGIN
    l_result := DBMS_PROFILER.start_profiler(run_comment => lv_profile_run_comment);
    
    /* Stuff to profile */

    l_result := DBMS_PROFILER.stop_profiler;

    
    Select P.runid
      into ln_profile_runid   
      from PLSQL_PROFILER_RUNS P
     where P.run_comment = lv_profile_run_comment 
    ;

    dbms_output.put_line('RUNID: ' || ln_profile_runid);
      
  END;  
```

## Profiling data tables  
  
 * PLSQL_PROFILER_RUNS  
 * PLSQL_PROFILER_UNITS
 * PLSQL_PROFILER_DATA

## Check a runs data 
  
```
with dataset as (
  Select U.unit_type, U.unit_name, D.line#, D.total_occur      
       , round(d.total_time / 1000000)                   total_time_msec
       , round(d.min_time / 1000000)                     min_time_msec
       , round(d.max_time / 1000000)                     max_time_msec
       , case when D.total_occur > 0 
              then round((d.total_time / D.total_occur) / 1000000)
              else -1 
          end avg_time_msec  
       -- , U.*, ' :: ', D.* 
    from PLSQL_PROFILER_UNITS U 
       , PLSQL_PROFILER_DATA  D
   where U.runid       = D.runid
     and U.unit_number = D.unit_number
     and U.runid = :runid 

union all 

Select '[TOTAL]' unit_type, '[TOTAL]' unit_name, -9999 line#, 1 total_occur
     , round( run_total_time / 1000000 )  total_time_msec
     , round( run_total_time / 1000000 )  min_time_msec
     , round( run_total_time / 1000000 )  max_time_msec
     , round( run_total_time / 1000000 )  avg_time_msec  
  from plsql_profiler_runs 
 where runid = :runid 

)
Select D.*
--      , (Select to_char( 
--                   DBMS_XMLGEN.CONVERT(EXTRACT(xmltype('<?xml version="1.0"?><document>'||XMLAGG(XMLTYPE('<V>'|| DBMS_XMLGEN.CONVERT(US.text)|| '</V>') order by US.line asc /*pode-se por um order by AQUI*/ ).getclobval()||'</document>'), '/document/V/text()') .getclobval(),1) 
--                ) AS data_value   
--           from user_source US
--           where US.type = D.unit_type
--             and US.name = D.unit_name
--             and US.line between D.line# and (D.line# + 20) 
--        ) snippet  
  from dataset D 
 order by D.total_time_msec desc  
```