# Lấy toàn bộ cột và bảng trong đây 

```
SELECT 
    table_name, 
    column_name
FROM 
    information_schema.columns
WHERE 
    table_schema = 'public'
ORDER BY 
    table_name, 
    ordinal_position;
```