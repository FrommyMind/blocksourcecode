---
title: Hive 复杂数据类型
date: 2018-05-13 15:08:12
tags:
  - Hive
  - Data Type
---

https://community.hortonworks.com/questions/22262/insert-values-in-array-data-type-hive.html

1 ) 
	
	INSERT INTO table test_array VALUES (1,array('a','b'));
Error: Error while compiling statement: FAILED: SemanticException [Error 10293]: Unable to create temp file for insert values Expression of type TOK_FUNCTION not supported in insert/values (state=42000,code=10293)
2 ) 
	
	INSERT INTO test_array (col2) VALUES (array('a','b')) from dummy limit 1; 
Error: Error while compiling statement: FAILED: ParseException line 1:54 missing EOF at 'from' near ')' (state=42000,code=40000)

<!-- more -->
	create table  test_array_type(
	arr array<string>
	);
	create table dummy(a string); insert into table dummy values ('a');
	INSERT INTO table test_array_type SELECT array('a','b') from dummy;

	SELECT * from test_array_type;
	
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest1.png?raw=true)

	select  arr[1] from test_array_type;
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest2.png?raw=true)


	select explode(arr) from test_array_type;

![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest3.png?raw=true)

	select * from (select explode(arr) from test_array_type) s join (select "AA")t on 1 =1 ;
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest4.png?raw=true)

	create table  test_map_type(
	map1 map<int,string>
	);
	INSERT INTO table test_map_type SELECT map(1,'b',2,'c') from dummy;
	select * from test_map_type;
	
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest5.png?raw=true)

	select map1,map_keys(map1),map_values(map1),map1[2] from test_map_type;
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest6.png?raw=true)



	create table  test_struct_type(
	map1 struct<col1:int,col2:string, col3:int>
	);

	INSERT INTO table test_struct_type SELECT struct(1,'b',2) from dummy;
	select * from test_struct_type;
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest7.png?raw=true)



	create table  test_arr_map_type(
	map1 array<map<int,string>>
	);

	INSERT INTO table test_arr_map_type SELECT array(map(1,'b',2,'c'),map(3,'e')) from dummy;
	select * from test_arr_map_type;
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest8.png?raw=true)



	select map1,map_keys(map1[0]),map_keys(map1[1]),map1[1][3] from test_arr_map_type;
![](https://github.com/FrommyMind/MarkDownPhotos/blob/master/screenshot/Hive/DataTypeTest/DateTypeTest9.png?raw=true)



