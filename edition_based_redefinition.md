# Edition-Based Redefinition

## Links   
 * [Oracle-Base](http://oracle-base.com/articles/11g/edition-based-redefinition-11gr2.php)
 * 
  
## Enable Edition-Based Redefinition to a schema 
  
```
-- As SYS, run. CAUTION: Irreversible 
alter user <schema> enable editions;
```
  
## Create a new version 
  
```
-- New version and associate to a schema
create edition <edition_name> as child of ora$base;
grant use on edition <edition_name> to <schema>;
```
  
## Associate current session with a named edition 
```
alter session set edition = <edition_name>;
```
