#关于今天省市区查询总结

----------

代码如下：

	SELECT XP.ID,S.CITYNAME,SP.CITYNAME,XP.CITYNAME FROM S_PROVINCES S 
	INNER JOIN S_PROVINCES SP ON S.ID=SP.PARENTID 
	INNER JOIN S_PROVINCES XP ON SP.ID=XP.PARENTID
	WHERE S.ID=440000 
	
总结：



首先我是先查询所有数据，然后我在通过多表连接，查询表中的数据，在得出广东省的id后我再关联区的表，然后通过三表连接，加上id等于广东省的id最后得出广东省的省市区的数据