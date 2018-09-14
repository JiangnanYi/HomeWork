# 总结分割方法

----------
创建一个省市县级表

1、我先查询的是 S_PROVINCES 表的省市县，通过关联，还有表自身的递归我查询除了所有的省市县

2、然后再查询 S_PROVINCES 表的省和市，得出一些只有市没有区的数据
3、通过 union 的连接，我将两个查询对接在一起，然后添加到 lagou_city 表中，得到一份完整的地区表

    CREATE TABLE lagou_city as 
    SELECT XP.ID as c_id,S.CITYNAME as c_province, SP.CITYNAME as c_city,XP.CITYNAME as c_district FROM S_PROVINCES S 
    	INNER JOIN S_PROVINCES SP ON S.ID=SP.PARENTID 
    	INNER JOIN S_PROVINCES XP ON SP.ID=XP.PARENTID
       where SP.depth=2
    union
    SELECT s.ID as c_id,S.CITYNAME as c_province, SP.CITYNAME as c_city,null FROM S_PROVINCES S 
    	INNER JOIN S_PROVINCES SP ON S.ID=SP.PARENTID 
       where SP.depth=2
创建一张公司表

1、我首先查询 lagou_position 表中的字段有多少个属于公司的信息。

2、然后通过分离，将查询到的公司的字段添加到 lagou_company 表中，生成一张公司表

3、把原来的 lagou_position_pk 的表中的公司字段给删除掉

    select * from lagou_position;
    create table lagou_company as
      select distinct l.company_id as company_id,
      l.company_short_name as short_name,
      l.company_full_name  as full_name,
      l.company_size   as size,
      l.financestage
      from lagou_position_pk l;

创建一张工作表

1、在创建 lagou_position 表的时候我们要把里面的城市和市区替换成 lagou_city 表的 c_id

2、所以我们先查询 lagou_position_pk 的数据中把 district 为空的字段找出来，然后查询 city 与 c_city 的字段是否相等，最后 district 为空的字段有哪些，
然后在和 lagou_city 表中的字段对比

3、对比后得出的的答案就是 lagou_position_pk 和 lagou_city 的 id 相符合的结果，然后插入到 lagou_position 表中

4、用 union all 连接 lagou_position_pk 的数据把 district 不为空的字段找出来，然后然后查询 city 与 c_city 的字段是否相等，最后查找 district 不为空的字段，
然后在和 lagou_city 表中的字段对比

5、对比后得出的的答案就是 lagou_position_pk 和 lagou_city 的 id 相符合的结果，然后插入到 lagou_position 表中

6、最后的出来的结果就是将多个表的 id 连接在一起的出答案

    create table lagou_position as
    select pid, c_id as city, company_id as company, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at
    from (select * from lagou_position_pk where district is null) p
      join lagou_city c on c.c_city like concat(p.city, '%') and c.c_district is null;
    
    insert into lagou_position
    select pid, c_id as city, company_id as company, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at
    from (select * from lagou_position_pk where district is not null) p
      join lagou_city c on c.c_city like concat(p.city, '%') and c.c_district like concat(p.district, '%');


