---
title: MySQL存储过程笔记
date: 2018-07-24 14:35:53
tags:
    - mysql
    - database
---

## 使用

>> 注意每行sql前不能留空

## 例子

显示数据库中所有的存储过程
```sql
show procedure status;
```

显示某个存储过程的详细信息(sp为存储过程名称)
```sql
show create procedure sp;
```

最简单的存储过程
```sql
drop procedure if exists sp//
CREATE PROCEDURE sp() select 1 //
--调用存储过程
call sp()//
```
<!-- more -->

带输入参数的存储过程

```sql
drop procedure if exists sp1 //
create procedure sp1(in p int)
    begin
    --声明一个int类型的变量
        declare v1 int;
    --将参数传给声明的变量，此步骤多余，可不要
        set v1 = p;
        insert into test(id) values(v1);
     end
      //
--调用这个存储过程
call sp1(1)//
--去数据库查看调用之后的结果
select * from test//
```

带输出参数的存储过程

```sql
drop procedure if exists sp2 //
create procedure sp2(out p int)
   begin
       select max(id) into p from test;
   end
   //
--调用该存储过程，注意：输出参数必须是一个带@符号的变量
call sp2(@pv)
```

带输入和输出参数的存储过程

```sql
drop procedure if exists sp3 //
create procedure sp3(in p1 int , out p2 int)
   begin
      if p1 = 1 then
   --用@符号加变量名的方式定义一个变量，与declare类似
         set @v = 10;
      else
         set @v = 20;
      end if;
   --语句体内可以执行多条sql，但必须以分号分隔
      insert into test(id) values(@v);
      select max(id) into p2 from test;
    end
    //
--调用该存储过程，注意：输入参数是一个值，而输出参数则必须是一个带@符号的变量
call sp3(1,@ret)//
select @ret//
```

既做输入又做输出参数的存储过程

```sql
drop procedure if exists sp4 //
create procedure sp4(inout p4 int)
   begin
      if p4 = 4 then
         set @pg = 400;
      else
         set @pg = 500;
      end if;
      select @pg;
    end//
call sp4(@pp)//
--这里需要先设置一个已赋值的变量，然后再作为参数传入
set @pp = 4//
call sp4(@pp)//
```

带游标的存储过程

```sql
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `sss`()
    COMMENT 'insert table'
BEGIN
    DECLARE root INT;
    DECLARE zid INT;
    DECLARE done INT DEFAULT FALSE;
    DECLARE rs CURSOR FOR select id,tid from demo;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN rs;
     read_loop: LOOP
        FETCH  NEXT from rs INTO root,zid;
        IF done THEN
            LEAVE read_loop;
        END IF;
        insert into temp(rootid,zid) value(root,zid);
        END LOOP;
    CLOSE rs;
END;;
DELIMITER ;
```
