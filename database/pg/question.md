### null 不能进行比较

这些比较的结果都为null

```sql
select 1 !=null, 1 = null, null = null, null != null, 1 in (null), 1 not in (null)
```

null比较方法

```
select null is null
```

