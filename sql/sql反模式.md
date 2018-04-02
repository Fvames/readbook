## SQL反模式

2. 乱穿马路
- 2.1 存储多值属性（，分隔存储）
- 2.2 查询使用函数instr， 聚合函数
- 2.3 字段长度
- 2.5 交叉表

3. 单纯的树
- 3.1 邻接表 两层结果
- 3.2 枚举路径 不能确保引用完整性，及路径字段的数据长度
- 3.3 嵌套集 查询快， 插入，删

 img/3.1.png

4. 主键ID， 伪主键
- 4.1 联合主键
- 4.2 外键名与主键名一致， 关联查询使用 using
 
5. 约束
- 5.1 外键
 
6. 泛型
- 6.1 单表继承： 所有类型所有列保留，使用一个属性区分类型
- 6.2 实体表继承： 每个子类对应一张表
- 6.3 类表继承： 公共属性表， 其他子类表
 
7. 多态关联
- 7.1 同主键关联
 
8. 多列属性
- 一对多关联表
 
9. 元数据分裂
- 同一张表，根据日期分片
 
10. 取整错误， decimal(18, 2)
 
11. 列限制可输入的数据值， 后期增加允许项 

12. 幽灵文件： 大文本， 图片不存储在数据库内，使用单独的文件管理系统， 导致与数据库不属于同一事物

13. 乱用所引

14. NULL值处理

15. 模棱两可的分组查询
- 15.1 关联子查询： 每一行都需关联子查询
- 15.2 使用衍生表： join关联结果集
- 15.3 使用join
```mysql
SELECT bp1.product_id, bp1.date_reported as latest, b1.bug_id
FROM Bugs b1 JOIN BugsProducts bp1 ON (bp1.bug_id = b1.bug_id)
LEFT OUT JOIN (
    Bugs AS b2 JOIN BugsProducts AS bp2 ON(bp2.bug_id=b2.bug_id)
) ON (
    bp1.product_id=bp2.product_id AND (b1.date_reported<b2.date_reported OR b1.date_reported=b2.date_reported AND b1.bug_id<b2.bug_id)
)
WHERE b2.bug_id IS NULL
``` 

16. 随机选择
- 16.1 从1到最大值随机选择，被选中值可能被删除
```mysql
SELECT b1.*
FROM Bugs AS b1
JOIN (SELECT CEIL(RAND() * (SELECT MAX(bug_id) FROM `Bugs`)) AS rand_id) AS b2
ON (b1.bug_id=b2.rand_id)
```
- 16.2 选择下一个最大值，队列中缝隙不大， 且对每个值要被等概率选中的重要性不高
```mysql
SELECT b1.*
FROM Bugs AS b1
JOIN (SELECT CEIL(RAND() * (SELECT MAX(bug_id) FROM `Bugs`)) AS rand_id) AS b2
WHERE (b1.bug_id>=b2.rand_id)
ORDER BY b1.bug_id
LIMIT 1

```
- 16.3 获取所有的键值，随机选择一个（程序处理， 数据量不大）
- 16.4 偏移量选择随机行
- 16.5 专有解决方案
```oracle
SELECT * FROM (
 SELECT * FROM Bugs SAMPLE(1) ORDER BY dbms_random.value
) WHERE ROWNUM = 1
```

17. 全文搜索（大文本搜索）
18. 意大利面条式查询
追求一条sql语句查询出结果，导致sql语句冗长，分多条sql
- union
- 批量构造sql语句
19. 隐式的列，去除*通配符
20. 明文存储密码， hash加盐（每个用户不同的盐）
21. SQL注入
- 21.1 过滤用户输入的内容
- 21.2 参数化动态查询 ？占位符
- 21.3 给动态输入的值加引号
- 21.4 用户和代码隔离，不同类型选择不同的sql
- 21.5 审查代码
22. 伪键洁癖， 自增主键强制连续
23. 非礼勿视，调试语句，打印出完整的sql语句及参数，再执行排查
24. 外交豁免权， 重视文档
- 24.1
    > 1)清晰定义项目需求， 并且落实为文档 
    > 2)设计并实现一个方案满足需求 
    > 3)验证并测试方案可行
- 24.2 编写文档
    > 实体关系图
    表，列及视图
    关系
    触发器
    存储过程
    sql安全， 访问权限
    数据库基础设施：备份，账户密码，集群
- 24.3 版本控制脚本    
    