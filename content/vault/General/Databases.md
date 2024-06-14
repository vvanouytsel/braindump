## Sqlite

- Open database from sqlite file

```bash
$ sqlite3 var/www/dsquare-appliance/tsq.sqlite3
$ sqlite3 /mnt/data/edge-manager/jobs.sqlite3
```

- Show columns of a specified table

```bash
sqlite> pragma table_info(job);

0|uuid|text|1||1
1|name|text|1||0
2|status|text|1||0
3|arguments|text|0||0
4|result|text|0||0
5|created|datetime|1||0
6|updated|datetime|1||0
```

- Show a couple of columns of a table

```bash
sqlite> SELECT uuid,name,status FROM job;
```

- Show uuid and result from a specific UUID

```bash
sqlite> SELECT uuid,result FROM job WHERE uuid="ca47cf76-10d2-4ee3-acba-e6db6964568f";
```

## MongoDB

- Connect to remote host

```console
mongosh --host microcks-mongodb.microcks.svc.cluster.local --port 27017
```

- Switch to a database

```console
# This assumes you are currently in the 'test' database and want to switch to the 'microcks' database
test> use microcks
switched to db microcks
```

- Authenticate to a database

```console
microcks> db.auth("myUser", "mySecretPassword")
{ ok: 1 }
```