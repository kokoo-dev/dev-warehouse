Mysql에서 Update 또는 Delete시 WHERE 구문에 서브쿼리를 사용할 경우 에러가 발생합니다.<br/>

> error.sql
~~~sql
UPDATE table_name
set column1 = 'value1'
where no = (
            select no 
            from table_name 
            where column2 = 'value2'
           );
~~~

Oracle이나 다른 DB에서는 문제가 없지만 MySQL에서는 오류가 발생하는 쿼리입니다. <br/>


이는 서브쿼리에 select를 한번 더 감싸준 후 alias를 지정해주면 해결됩니다. <br/>
> success.sql
~~~sql
UPDATE table_name
set column1 = 'value1'
where no = (
            select no
            from (
                  select no 
                  from table_name
                  where column2 = 'value2'
                 ) sub
           );
~~~
