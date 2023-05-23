#### 临时函数

创建临时函数

```sql
-- in 后边为入参
-- out 后边为返参数,多个就写多个
CREATE OR REPLACE FUNCTION pg_temp.tmp_calculate_gender_id_func(in id_no text, out gender_id int) as
$$
DECLARE
    gender_id_str text;
BEGIN
    gender_id := 0;
    IF length(id_no) != 18 and length(id_no) != 15 then
        return;
    END IF;
    IF length(id_no) = 18 THEN
        gender_id_str := substr(right(id_no, 2), 1, 1);
    ELSEIF length(id_no) = 15 THEN
        gender_id_str := right(id_no, 1);
    END IF;
    IF gender_id_str not in ('0', '1', '2', '3', '4', '5', '6', '7', '8', '9') then
        return;
    END IF;
    gender_id := to_number(gender_id_str, '9');
    gender_id = gender_id % 2;
    IF gender_id = 0 then
        gender_id = 2;
    END IF;
    return;
end;

$$
    LANGUAGE plpgsql;
```

调用临时函数


```sql
SELECT pg_temp.tmp_calculate_gender_id_func('35078119640307722');
```

