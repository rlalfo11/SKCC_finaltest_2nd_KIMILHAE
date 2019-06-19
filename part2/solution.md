# Part2 - Big Data Knowledge Test

## Problem 1

	select a.id, a.type, a.status, a.amount, (a.amount-b.avg_amt) as difference
	from account a
	join (select type , avg(amount) as avg_amt
		  from problem1.account
		  group by type
		  ) b
	on a.type = b.type
	where a.status='Active'


## Problem 2

	create external table solution
	(
		id int,
		fname string,
		lname string,
		address string,
		city string,
		state string,
		zip string,
		birthday string,
		hireday string
	)
		STORED AS PARQUET
		LOCATION '/user/training/problem2/data/employee'

	select * from solution limit 10;

## Problem 3

	create table solution as
	select c.id, c.fname, c.lname, c.hphone from customer c
	join
	account a
	on (c.id = a.custid)
	where a.amount < 0;


## Problem 4



## Problem 5

	select c.fname, c.lname, c.city, c.state
	from customer c
	where c.city = 'Palo Alto'
	and state = 'CA'
	union all
	select e.fname, e.lname, e.city, e.state
	from employee e
	where e.city = 'Palo Alto'
	and state = 'CA'


## problem 6

	create table `problem6.solution`(
		  `id` int,
		  `fname` string,
		  `lname` string,
		  `address` string,
		  `city` string,
		  `state` string,
		  `zip` string,
		  `birthday` string
	)

	insert into table problem6.solution
	select  `id` ,
		  `fname` ,
		  `lname` ,
		  `address` ,
		  `city` ,
		  `state` ,
		  `zip` ,
		 substring( `birthday`, 0 ,5)  from problem6.employee

	select * from solution;


## problem 7

	select concat(fname, ' ' , lname) as name FROM employee
	where city = 'Seattle'
	order by name
	limit 10


## problem 8

	create table `solution` (
	  `id` int NOT NULL,
	  `fname` varchar(255) default NULL,
	  `lname` varchar(255) default NULL,
	  `address` varchar(255) default NULL,
	  `city` varchar(255),
	  `state` varchar(50) default NULL,
	  `zip` varchar(10) default NULL,
	  `birthday` varchar(255),
	  PRIMARY KEY (`id`)
	);
	
	sqoop export --connect jdbc:mysql://localhost/problem8 --username training --password training --export-dir /user/training/problem8/data/customer --table solution


## problem 9

	create table `solution`(
		  `id` string,
		  `fname` string,
		  `lname` string,
		  `address` string,
		  `city` string,
		  `state` string,
		  `zip` string,
		  `birthday` string
		  )

	insert into table solution
	select  concat('A', id) ,
		  fname ,
		  lname ,
		  address ,
		  city,
		  state ,
		  zip ,
		  birthday from customer;

	select * from solution limit 10;

## problem 10

	create view problem10.solution AS
	select a.id,
		   a.fname,
		   a.lname,
		   a.city,
		   a.state,
		   b.charge,
		   substr(b.tstamp,1,10) as billdata
	from customer a
	join billing b on a.id = b.id;

## problem 11

11-a.
	select brand, name, COUNT(p.prod_id) AS sold
	from products p
	join order_details d
	  on (p.prod_id = d.prod_id)
	where p.brand = 'Dualcore'
	group by brand, name, p.prod_id
	order by sold desc
	limit 3;

