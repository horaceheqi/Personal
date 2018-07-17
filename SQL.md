SQL注意事项
---
- 给已有表添加分区
```
alter talbe tableName add if not exists partition (stat_month='${month1}')
```
- hive 语句用 group by 不用 distinct 加快执行效率
