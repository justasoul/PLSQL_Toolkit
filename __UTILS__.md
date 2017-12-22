# UTILS 


## Determine row/line checksum  
  
```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```
  
```plsql
Create or replace function bn_get_row_cheksum (pv_table_name varchar2, pv_rowid rowid) return varchar2 is
  pragma autonomous_transaction;
-- See: https://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:15843631345170
	lv_res varchar2(100);
begin
  lv_res := owa_opt_lock.checksum(user,pv_table_name, pv_rowid);
	rollback;
	
	return lv_res; 
end bn_get_row_cheksum;   
```
  
```PLSQL
Create or replace function bn_get_row_cheksum (pv_table_name varchar2, pv_rowid rowid) return varchar2 is
  pragma autonomous_transaction;
-- See: https://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:15843631345170
	lv_res varchar2(100);
begin
  lv_res := owa_opt_lock.checksum(user,pv_table_name, pv_rowid);
	rollback;
	
	return lv_res; 
end bn_get_row_cheksum;   
```  

``` plsql
Create or replace function bn_get_row_cheksum (pv_table_name varchar2, pv_rowid rowid) return varchar2 is
  pragma autonomous_transaction;
-- See: https://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:15843631345170
	lv_res varchar2(100);
begin
  lv_res := owa_opt_lock.checksum(user,pv_table_name, pv_rowid);
	rollback;
	
	return lv_res; 
end bn_get_row_cheksum;   
```
  
``` PLSQL
Create or replace function bn_get_row_cheksum (pv_table_name varchar2, pv_rowid rowid) return varchar2 is
  pragma autonomous_transaction;
-- See: https://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:15843631345170
	lv_res varchar2(100);
begin
  lv_res := owa_opt_lock.checksum(user,pv_table_name, pv_rowid);
	rollback;
	
	return lv_res; 
end bn_get_row_cheksum;   
```    