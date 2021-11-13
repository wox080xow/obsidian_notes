Source倒出資料
```
su - postgres
pg_dumpall --file=/tmp/pg_dumpall.sql
```
Destination倒入資料
```
su - postgres
psql -f /tmp/pgdumpall.sql
```
