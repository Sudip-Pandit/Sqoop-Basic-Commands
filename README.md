# Sqoop-Basic-Commands
This repository contains all the basic and advance Sqoop commands

====================SQOOP COMMANDS WITH DUMMY DATASETS=========================
1. Incremental Import
-----------------
create database incredb;
use incredb;

CREATE TABLE custoincre(custid INT,firstname VARCHAR(20),createdt date);


insert into custoincre values(1,'Arun','2015-09-20');
insert into custoincre values(2,'srini','2015-09-21');
insert into custoincre values(3,'vasu','2015-09-22');


----------------------------
Come to edge node and execute
----------------------------

sqoop import \
--connect jdbc:mysql://localhost/incredb \
--username root \
--password <>  \
--table custoincre \
-m 1 \
--target-dir /user/cloudera/datadir

hadoop fs -ls /user/cloudera/datadir
hadoop fs -cat /user/cloudera/datadir/part-m-00000


----------------------------
Come to mysql and add 2 more records
----------------------------

insert into custoincre values(4,'mohamed','2015-09-23');
insert into custoincre values(5,'arun','2015-09-24');

----------------------------
Come to edge node and execute incremental command
----------------------------

sqoop import \
--connect jdbc:mysql://localhost/incredb \
--username root \
--password <> \
--table custoincre \
-m 1 \
--target-dir /user/cloudera/datadir \
--incremental append \
--check-column custid \
--last-value 3


----------------------------
Come to edge node to verify new data
----------------------------

hadoop fs -ls /user/cloudera/datadir
hadoop fs -cat /user/cloudera/datadir/part-m-00001

2) SAVE IN AVRO FILE FORMAT:

Create the datbase and table in mysql

create database dbser;
use dbser;
CREATE TABLE custser(custid INT,firstname VARCHAR(20),createdt date);

insert into custser values(1,'Arun','2015-09-20');
insert into custser values(2,'srini','2015-09-21');
insert into custser values(3,'vasu','2015-09-22');

Edge Node
============
sqoop import \
--connect  jdbc:mysql://localhost/dbser \
--username root \
--password <> \
--table custser \
-m 1 \
--delete-target-dir \
--target-dir /user/cloudera/jobdata_test \
--as-avrodatafile

Check the file:
---------------------
hadoop fs -cat /user/cloudera/jobdata_test/*  

3) Sqoop Job Created

=======================================
Mysql 
=======================================
mysql -uroot -p<>

create database custserdb;
use custserdb;

CREATE TABLE cust_job(custid INT,firstname VARCHAR(20),createdt date);


insert into cust_job values(1,'Arun','2015-09-20');
insert into cust_job values(2,'srini','2015-09-21');
insert into cust_job values(3,'vasu','2015-09-22');

=======================================
Edge node create the job ,validate,execute
=======================================
hadoop dfsadmin -safemode leave

sqoop job \
--create incjob \
-- import \
--connect jdbc:mysql://localhost/custserdb \
--username root \
--password cloudera \
--table cust_job \
-m 1 \
--target-dir /user/cloudera/job_out \
--incremental append \
--check-column custid \
--last-value 0

sqoop job --list

sqoop job --exec incjob      ---> give password <> and give enter

hadoop fs -cat /user/cloudera/job_out/*
cd
cd .sqoop
cat metastore.db.script | grep 'last.value'   --> you will see 3

========================================
Mysql
=======================================

insert into cust_job values(4,'mohamed','2015-09-23');
insert into cust_job values(5,'ravi','2015-09-24');


==============================
execute the job again
==============================
sqoop job --exec incjob  ---> give password cloudera and give enter
cd
cd .sqoop
cat metastore.db.script | grep 'last.value'    --> gets updated to 5

4) Not ask Password while executing the commands

=======================================
Edge node create the job ,validate,execute
=======================================

hadoop dfsadmin -safemode leave

echo -n cloudera>/home/cloudera/pwfile

sqoop job \
--create passjob \
-- import \
--connect jdbc:mysql://localhost/custserdb \
--username root \
--password-file file:///home/cloudera/pwfile \
--table cust_job \
-m 1 \
--target-dir /user/cloudera/job_out_pass \
--incremental append \
--check-column custid \
--last-value 0

sqoop job --list

sqoop job --exec passjob      ---> It will not ask the password

===========================
6) Update tasks commands
===============================

Task 1 ------

Mysql ---

create database custdd;
use custdd;
CREATE TABLE cust_s(custid INT,firstname VARCHAR(20));


insert into cust_s values(1,'Arun');
insert into cust_s values(2,'srini');
insert into cust_s values(3,'vasu');


Create a passfile --- in edge Node

echo -n  cloudera>/home/cloudera/pfile

Push it to hdfs

hadoop fs -put /home/cloudera/pfile /user/cloudera/

Create sqoop job with that hdfs path 

sqoop job \
--create passjobh \
-- import \
--connect jdbc:mysql://localhost/custdd \
--username root \
--password-file hdfs:/user/cloudera/pfile \
--table cust_s \
-m 1 \
--target-dir /user/cloudera/job_out_h \
--incremental append \
--check-column custid \
--last-value 0

Execute the Job

sqoop job --exec passjobh   ->it should not ask the password 


Task 2 ------->

Import the same table as parquet

sqoop import \
--connect jdbc:mysql://localhost/custdd \
--username root \
--password-file hdfs:/user/cloudera/pfile \
--table cust_s \
-m 1 \
--delete-target-dir \
--target-dir /user/cloudera/job_out_parquet \
--as-parquetfile


hadoop fs -cat /user/cloudera/job_out_parquet/*


=======================================
7) 

Mysql 
==============================

mysql -uroot -pcloudera
create database test;
use test;

CREATE TABLE customermoda(custid INT,firstname VARCHAR(20),createdt date);

insert into customermoda values(1,'Arun','2015-09-20');
insert into customermoda values(2,'srini','2015-09-21');
insert into customermoda values(3,'vasu','2015-09-22');
insert into customermoda values(4,'mohamed','2015-09-23');
insert into customermoda values(5,'arun','2015-09-24');

==============================
Open Edge Node
==============================


sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
-password cloudera \
-m 1 \
--table customermoda \
--target-dir /user/cloudera/dataimportmod2

hadoop fs -ls /user/cloudera/dataimportmod2
hadoop fs -cat /user/cloudera/dataimportmod2/*

==============================
Mysql 
==============================

update customermoda set firstname='zeyo' where custid=3;
select * from customermoda;

==============================
Open Edge Node
==============================

sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
-password cloudera \
-m 1 \
--table customermoda \
--target-dir /user/cloudera/dataimportmod \
--incremental lastmodified \
--check-column createdt \
--last-value 2015-09-22 \
--merge-key custid



hadoop fs -ls /user/cloudera/dataimportmod2
hadoop fs -cat /user/cloudera/dataimportmod2/*


===================================================================
8) SQOOP AWS IMPORT
Mysql ---
=========

create database test;
use test;

CREATE TABLE customermod4(custid INT,firstname VARCHAR(20),lastname VARCHAR(20),city varchar(50),age int,createdt date,transactamt int );

insert into customermod4 values(1,'Arun','Kumar','chennai',33,'2015-09-20',100000);
insert into customermod4 values(2,'srini','vasan','chennai',33,'2015-09-21',10000);
insert into customermod4 values(3,'vasu','devan','banglore',39,'2015-09-22',90000);
insert into customermod4 values(4,'mohamed','imran','hyderabad',33,'2015-09-23',1000);
insert into customermod4 values(5,'arun','basker','chennai',23,'2015-09-24',200000);
insert into customermod4 values(6,'arun','basker','chennai',23,'2015-09-24',200000);

=========
Edge Node
=========


sqoop import \
-Dfs.s3a.access.key=AKIAZXAZFFZFREINE5UZ \
-Dfs.s3a.secret.key=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  \
-Dfs.s3a.endpoint=s3.ap-south-1.amazonaws.com  \
--connect jdbc:mysql://localhost/test \
--username root \
--password cloudera \
--table customermod4 \
-m 1 \
--target-dir s3a://clouderazeyo/<urname>_import;

==========================================================

9) ======
Mysql
======

use test;

CREATE TABLE customer_mapper(custid INT,firstname VARCHAR(20),lastname VARCHAR(20),city varchar(50),age int,createdt date,transactamt int );

insert into customer_mapper values(1,'Arun','Kumar','chennai',33,'2015-09-20',100000);
insert into customer_mapper values(2,'srini','vasan','chennai',33,'2015-09-21',10000);
insert into customer_mapper values(3,'vasu','devan','banglore',39,'2015-09-22',90000);
insert into customer_mapper values(4,'mohamed','imran','hyderabad',33,'2015-09-23',1000);
insert into customer_mapper values(5,'arun','basker','chennai',23,'2015-09-24',200000);
insert into customer_mapper values(6,'arun','basker','chennai',23,'2015-09-24',200000);
insert into customer_mapper values(7,'arun','basker','chennai',23,'2015-09-24',200000);
insert into customer_mapper values(8,'arun','basker','chennai',23,'2015-09-24',200000);
insert into customer_mapper values(9,'arun','basker','chennai',23,'2015-09-24',200000);
insert into customer_mapper values(10,'arun','basker','chennai',23,'2015-09-24',200000);


======
Edge Node
======

sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
--password cloudera \
--table customer_mapper \
-m 1  \
--delete-target-dir \
--target-dir /user/cloudera/mapper_data

hadoop fs -ls /user/cloudera/mapper_data --- One part File

========================================
sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
--password cloudera \
--table customer_mapper \
-m 2  \
--delete-target-dir \
--target-dir /user/cloudera/mapper_data  =====> it should fail

=========================================
sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
--password cloudera \
--table customer_mapper \
-m 2 \
--split-by custid  \
--delete-target-dir \
--target-dir /user/cloudera/mapper_data      

hadoop fs -ls /user/cloudera/mapper_data ==========>>>>> Two Part  Files


========================================
sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
--password cloudera \
--table customer_mapper \
--split-by custid  \
--delete-target-dir \
--target-dir /user/cloudera/mapper_data      

hadoop fs -ls /user/cloudera/mapper_data --- Four or Five Part Files


==================

Task 1 ----

======
Mysql
======
create database test;
use test;

CREATE TABLE customer_mapper_avro(custid INT,firstname VARCHAR(20),lastname VARCHAR(20),city varchar(50),age int,createdt date,transactamt int );

insert into customer_mapper_avro values(1,'Arun','Kumar','chennai',33,'2015-09-20',100000);
insert into customer_mapper_avro values(2,'srini','vasan','chennai',33,'2015-09-21',10000);
insert into customer_mapper_avro values(3,'vasu','devan','banglore',39,'2015-09-22',90000);
insert into customer_mapper_avro values(4,'mohamed','imran','hyderabad',33,'2015-09-23',1000);
insert into customer_mapper_avro values(5,'arun','basker','chennai',23,'2015-09-24',200000);

============
Edge Node
============

create a directory /home/cloudera/avrodir
Go inside that directory   -- cd /home/cloudera/avrodir
ls

========================================================
sqoop import \
--connect jdbc:mysql://localhost/test \
--username root \
--password cloudera \
--table customer_mapper_avro \
-m 1  \
--delete-target-dir \
--target-dir /user/cloudera/avrodata \
--as-avrodatafile

hadoop fs -put customer_mapper_avro.avsc /user/cloudera/



