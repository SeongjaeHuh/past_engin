1번

```
select a.id as id, a.type as type, a.status as status, a.amount as amount, a.amount-b.average as difference
from account a join (select avg(amount) as average, type from account where status = 'Closed' group by type) b
on a.type = b.type
where a.status = 'Active'
;
```


2번

```
create database problem2;

create external table solution
(id INT, fname STRING, lname STRING, address STRING, city STRING, state STRING,
zip STRING, birthday STRING, hireday STRING)
row format SERDE 'parquet.hive.serde.ParquetHiveSerDe'
STORED AS
INPUTFORMAT "parquet.hive.DeprecatedParquetInputFormat"
OUTPUTFORMAT "parquet.hive.DeprecatedParquetOutputFormat"
LOCATION '/user/training/problem2/data/employee';
```

3번
```
create table solution as
select b.id as id , b.fname as fname, b.lname as lname, regexp_replace(b.hphone,'[ )(-]','') as hphone
from account a join customer b
on a.custid=b.id
where a.amount >= 0
;
```

4번


```
create external table employee1
(
cust_id int,
lname string,
fname string,
address string,
city string,
state string,
zip_cd string
)
row format delimited
fields terminated by '\t'
location '/user/training/problem4/data/employee1/.'
```
```
create external table employee2
(
cust_id int,
number int,
lname string,
fname string,
address string,
city string,
state string,
zip_cd string
)
row format delimited
fields terminated by ','
location '/user/training/problem4/data/employee2/.'
```

```
create table solution as
select res.cust_id, res.fname, res.lname, res.address, res.city, res.state, res.zip_cd
from
(
select cust_id, fname, lname, address, city, state, substr(zip_cd,0,5) zip_cd
from employee1
where state = 'CA'

union all

select cust_id, initcap(fname) fname, initcap(lname) lname, address, city, state, zip_cd
from employee2
where state = 'CA'
) res
;

```

```

insert OVERWRITE DIRECTORY '/user/training/problem4/solution/'
row format delimited
FIELDS TERMINATED BY '\t'
SELECT * FROM solution
;
```


5번

```
select concat_ws(',', cus.fname, cus.lname, cus.zip)
from customer cus
where city='Palo Alto'
and state = 'CA'

union all

select concat_ws(',', emp.fname, emp.lname, emp.zip)
from employee emp
where city='Palo Alto'
and state = 'CA'
;
```

```
hive --database problem5 -f solution.sql
```


6번


```

create table solution as
select id,fname,lname,address,city,state,zip,substr(birthday,7,4) birthyear
from employee
;
```




7번
```
select concat_ws(',',res.lname,res.fname)
from
(
select lname,fname
from employee
where city = 'Seattle'
order by lname, fname
) res
;
```
```
hive --database problem7 -f solution.sql
```

8번

```
sqoop export
--connect jdbc:mysql://localhost/problem8
--username cloudera
--password cloudera
--table solution
--fields-terminated-by '\t'
--export-dir /user/training/problem8/data/customer/.
```

9번

```
# concat 사용하니까 id의 type이 자동으로 string이 되었음.
create table solution as
select concat('A',id) id, fname, lname, address, city, state, zip from customer
;
```


10번

```
create view solution as
select c.id id, c.fname fname, c.lname lname, c.state state, c.zip zip, b.charge charge, substr(b.tstamp,0,10) billdate
from billing b, customer c
where b.id = c.id
;
```

11번

```
select o.prod_id, count(*) cnt from order_details o, products p
where o.prod_id = p.prod_id
and p.brand = 'Dualcore'
group by o.prod_id
order by cnt desc
limit 3;
```

```
select to_date(o.order_date) date, sum(p.price) revenue, sum(p.price-p.cost) profit
from orders o, order_details d, products p
where o.order_id = d.order_id
and d.prod_id = p.prod_id
and p.brand = 'Dualcore'
group by to_date(o.order_date);
```

```
select o.order_id order_id, sum(p.price) revenue
from orders o, order_details d, products p
where o.order_id = d.order_id
and d.prod_id = p.prod_id
group by o.order_id
order by revenue desc
limit 10;
```
