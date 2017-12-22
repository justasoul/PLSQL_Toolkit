# Profiler DBMS_HPROF

## Run a profiling run 
  
```PLSQL
DECLARE
-- l_result  BINARY_INTEGER;

lv_bd_dir varchar2(100) := 'PLSHPROF_DIR';
lv_profile_run_comment  PLSQL_PROFILER_RUNS.RUN_COMMENT%TYPE := upper('BN_HPROF_TESTE_001');
ln_profile_runid        PLSQL_PROFILER_RUNS.RUNID%TYPE;
BEGIN

dbms_output.put_line('start');

DBMS_HPROF.start_profiling(
    location=> lv_bd_dir
  , filename=> 'profiler.txt'                  
);


/* Code to be profiled */

DBMS_PROFILER.stop_profiler;

ln_profile_runid := DBMS_HPROF.analyze (
                     location    => lv_bd_dir,
                     filename    => 'profiler.txt',
                     run_comment => lv_profile_run_comment
                   );

dbms_output.put_line('RUNID: ' || ln_profile_runid);
  
END;  
```
  
The profiler can also be run from SQL Developer

## Validate profiling run data
  
```
with dataset as (
  SELECT level nivel
       , substr(SYS_CONNECT_BY_PATH(symbolid, '/'), 2) breadcrumbs 
       , RPAD(' ', level*2, ' ') || fi.owner || '.' || fi.module AS name
       , fi.function
       , pci.subtree_elapsed_time -- microseconds
       , pci.function_elapsed_time -- microseconds
       , round(pci.function_elapsed_time / pci.calls) avg_function_elapsed_time -- microseconds
       , pci.calls
       , FI.symbolid
       , PCI.parentsymid         
  FROM   dbmshp_parent_child_info pci
         JOIN dbmshp_function_info fi ON pci.runid = fi.runid AND pci.childsymid = fi.symbolid
  WHERE  pci.runid = :RUNID  
  CONNECT BY PRIOR childsymid = parentsymid 
  START WITH pci.parentsymid = 1
  order siblings by pci.subtree_elapsed_time desc 
)
Select * 
  from dataset D  
 where D.nivel in (1, 2, 3) 
```
  
**From Oracle-Base**  
  
```
SELECT RPAD(' ', (level-1)*2, ' ') || a.name AS name,
       a.subtree_elapsed_time,
       a.function_elapsed_time,
       a.calls
FROM   (SELECT fi.symbolid,
               pci.parentsymid,
               RTRIM(fi.owner || '.' || fi.module || '.' || NULLIF(fi.function,fi.module), '.') AS name,
               NVL(pci.subtree_elapsed_time, fi.subtree_elapsed_time) AS subtree_elapsed_time,
               NVL(pci.function_elapsed_time, fi.function_elapsed_time) AS function_elapsed_time,
               NVL(pci.calls, fi.calls) AS calls
        FROM   dbmshp_function_info fi
               LEFT JOIN dbmshp_parent_child_info pci ON fi.runid = pci.runid AND fi.symbolid = pci.childsymid
        WHERE  fi.runid = :run_id 
        AND    fi.module != 'DBMS_HPROF') a
CONNECT BY a.parentsymid = PRIOR a.symbolid
START WITH a.parentsymid IS NULL;
```
  
**Mine**  

  
```
with dataset as (
  SELECT a.runid, 
         (Select R.run_comment from dbmshp_runs R where R.runid = a.runid) run_comment, 
         level nivel, 
         RPAD(' ', (level-1)*2, ' ') || a.name AS name,
         a.subtree_elapsed_time,
         a.function_elapsed_time,
         a.calls,
         round(a.function_elapsed_time/a.calls) function_average_time 
  FROM   (SELECT fi.runid, 
                 fi.symbolid,
                 pci.parentsymid,
                 RTRIM(fi.owner || '.' || fi.module || '.' || NULLIF(fi.function,fi.module), '.') AS name,
                 NVL(pci.subtree_elapsed_time, fi.subtree_elapsed_time) AS subtree_elapsed_time,
                 NVL(pci.function_elapsed_time, fi.function_elapsed_time) AS function_elapsed_time,
                 NVL(pci.calls, fi.calls) AS calls
          FROM   dbmshp_function_info fi
                 LEFT JOIN dbmshp_parent_child_info pci ON fi.runid = pci.runid AND fi.symbolid = pci.childsymid
          WHERE  fi.runid = :run_id 
          AND    fi.module != 'DBMS_HPROF'
         ) a
  CONNECT BY a.parentsymid = PRIOR a.symbolid
  
  START WITH a.parentsymid IS NULL
  order siblings by function_elapsed_time desc 
)

Select * 
  from dataset D 
```
  
## Reset profiling data tables 
  
```
begin

  execute immediate 'truncate table dbmshp_parent_child_info';

  -- execute immediate 'truncate table dbmshp_function_info';
  for i in (Select Rowid row_id, k.* from dbmshp_function_info K ) loop 
    delete from dbmshp_function_info where rowid = i.row_id;
  end loop;

  -- execute immediate 'truncate table dbmshp_runs';
  for i in (Select Rowid row_id, k.* from dbmshp_runs K ) loop 
    delete from dbmshp_runs where rowid = i.row_id;
  end loop;

end;
```  