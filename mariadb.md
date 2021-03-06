[documentation](https://mariadb.com/kb/en/)

### execute docker container ( utf8 ):
```
docker pull mariadb

docker run --name mysql-container --volume /my/local/folder/data:/var/lib/mysql --publish 3306:3306 --env MYSQL_ROOT_PASSWORD=root --detach mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

### execute docker container, create DB with specific name, execute all scripts in folder:
```
docker pull mariadb

docker run --name mysql-container --volume /my/local/folder/data:/var/lib/mysql --volume /my/path/to/sql:/docker-entrypoint-initdb.d --publish 3306:3306 --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE={databasename} --detach mariadb --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

```

### connect to mysql shell tool:
```
mysql --user=root --password=root
```
```
docker exec -it mysql-container  /usr/bin/mysql  --user=root --password=root
```

### mycli connect
```
mycli mysql://user_name:passw@mysql_host:mysql_port/schema
```
#### .myclirc settings for vertical output
```properties
auto_vertical_output = True
```

#### mycli execute script, mycli 
```
mycli mysql://user_name:passw@mysql_host:mysql_port/schema --execute "select * from dual"
mycli mysql://user_name:passw@mysql_host:mysql_port/schema < some-sql.txt
```
```
mycli mysql://user_name:passw@mysql_host:mysql_port/schema
help;
source some-sql.txt;
```


### import db export db, archive, backup, restore
```sh
# prerequisites
## try just to connect to db
mysql --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --database=masterdb --password=my_passw
## docker mysql client 
docker run -it mariadb mysql --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --database=masterdb --password=my_passw

# backup ( pay attention to 'database' key )
mysqldump --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --password=my_passw masterdb > backup.sql
mysqldump --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --password=my_passw --databases masterdb > backup.sql
mysqldump --extended-insert --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --password=my_passw --databases masterdb | sed 's$),($),\n($g' > backup.sql
mysqldump --extended-insert=FALSE --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --password=my_passw --databases masterdb > backup.sql

# backup only selected tables 
mysqldump --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 --password=my_passw masterdb table_1 table_2 > backup.sql

# restore
mysql -u mysql_user -p DATABASE < backup.sql
```

#### backup issue: during backup strange message appears: "Enter password:" even with password in command line
```sh
vim .my.cnf
```
```properties
[mysqldump]
user=mysqluser
password=secret
```
```sh
# !!! without password !!!
mysqldump --host=mysql-dev-eu.a.db.ondigitalocean.com --user=admin --port=3060 masterdb table_1 table_2 > backup.sql
```

### execute sql file with mysqltool
* inside mysql 
```
source /path/to/file.sql
```
* shell command
```
mysql -h hostname -u user database < path/to/test.sql
```
* via docker
```
docker run -it --volume $(pwd):/work mariadb mysql --host=eu-do-user.ondigitalocean.com --user=admin --port=25060 --database=master_db --password=pass
# docker exec --volume $(pwd):/work -it mariadb
source /work/zip-code-us.sql
```

### show databases and switch to one of them:
```
show databases;
use {databasename};
```

### user <-> role
![user role relationship](https://i.postimg.cc/bv0dRDrg/mysql-user-role.png)

### print all tables
```sql
show tables;
```

### print all tables and all columns
```sql
select table_name, column_name, data_type from information_schema.columns
 where TABLE_NAME like 'some_prefix%'
order by TABLE_NAME, ORDINAL_POSITION
```

### print all columns in table, show table structure
```sql
describe table_name;
show columns from table_name;
select * from information_schema.columns where TABLE_NAME='listings_dir' and COLUMN_NAME like '%PRODUCT%';
```
```sh
mycli mysql://user:passw@host:port/schema --execute "select table_name, column_name, data_type  from information_schema.columns where TABLE_NAME like 'hlm%' order by TABLE_NAME, ORDINAL_POSITION;" | awk -F '\t' '{print $1","$2","$3}' > columns
```

### add column 
```sql
-- pay attention to quotas around names
ALTER TABLE `some_table` ADD `json_source` varchar(32) NOT NULL DEFAULT '';
-- don't use 'ALTER COLUMN'
ALTER TABLE `some_table` MODIFY `json_source` varchar(32) NULL;
```


### version
SELECT VERSION();

### example of spring config
* MariaDB
```
ds.setMaximumPoolSize(20);
ds.setDriverClassName("org.mariadb.jdbc.Driver");
ds.setJdbcUrl("jdbc:mariadb://localhost:3306/db");
ds.addDataSourceProperty("user", "root");
ds.addDataSourceProperty("password", "myPassword");
ds.setAutoCommit(false);
jdbc.dialect:
  org.hibernate.dialect.MariaDBDialect
  org.hibernate.dialect.MariaDB53Dialect
```

* MySQL
```
jdbc.driver: com.mysql.jdbc.Driver
jdbc.dialect: org.hibernate.dialect.MySQL57InnoDBDialect
jdbc:mysql://localhost:3306/bpmnui?serverTimezone=Europe/Brussels
```

### maven dependency
* MySQL
```
ds.setDriverClassName("com.mysql.jdbc.Driver");
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.6</version>
</dependency>
```

* MariaDB
```
ds.setDriverClassName("org.mariadb.jdbc.Driver");
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <version>2.2.4</version>
</dependency>
```

### create database:
```
DROP DATABASE IF EXISTS {databasename};
CREATE DATABASE {databasename}
  CHARACTER SET = 'utf8'
  COLLATE = 'utf8_general_ci';
```

### create table, autoincrement
```
create table IF NOT EXISTS `hlm_auth_ext`(
  `auth_ext_id` bigint NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `uuid` varchar(64) NOT NULL,
)  ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

### show ddl, ddl for table, ddl table
```
SHOW CREATE TABLE yourTableName;
```

### subquery returns more than one row
```
select 
    u.name_f, 
    u.name_l, 
    (select GROUP_CONCAT(pp.title, '')
    from hlm_practices pp where user_id=100
    )
from hlm_user u 
where u.user_id = 100;
```

### case search, case sensitive, case insensitive
```sql
-- case insensitive search
AND q.summary LIKE '%my_criteria%'

-- case sensitive ( if at least one operand is binary string )
AND q.summary LIKE BINARY '%my_criteria%'
```

### date diff, compare date, datetime substraction
```sql
----- return another date with shifting by interval
-- (now() - interval 1 day)
----- return amount of days between two dates
-- datediff(now(), datetime_field_in_db )
```
