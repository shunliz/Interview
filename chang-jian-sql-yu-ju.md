1. Oracle 分页查询

```
SELECT a2.* FROM (SELECT a1.*, ROWNUM rn FROM (SELECT * FROM emp ORDER BY sal) a1 WHERE ROWNUM=10) a2 WHERE rn=6；
```
2. 用结果集创建新表





