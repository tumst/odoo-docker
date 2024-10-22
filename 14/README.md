
# Copy to Server
```
pscp -r .\modules\comsec_meeting admin@172.19.4.46:/home/admin/odoo-14.0/custom-addons
pscp -r .\modules\jasper_rpt admin@172.19.4.46:/home/admin/odoo-14.0/custom-addons
```

# run odoo
```
python odoo-bin -c ./odoo-dev.conf --log-sql --log-level debug --log-request --log-response

-- for Linux: Ubuntu
python3 odoo-bin -c ./odoo.conf --dev reload,xml
```

# Find and kill process in Ubuntu
```
--- find port
$sudo lsof -i -P -n | grep 8069
$sudo ss -pln | grep postgresql

--- kill process by port
$fuser -k 8069/tcp


The PostgreSQL utility pg_lsclusters shows information about the configuration and status of all clusters, including the port number.

$ pg_lsclusters

```

# nginx conf
```
sudo ln -s sites-available/comsec.conf sites-enabled/comsec.conf        -- symlink  !!! error file not found !!!


nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo systemctl start nginx

```


# psql command.
```
-- switch user
$sudo -i -u postgres  // switch to postgres user
# sudo psql "dbname=comsec host=localhost port=5432 user=comsec sslmode=verify-full"
# sudo psql postgresql://postgres@127.0.0.1:5432/postgres?sslmode=require
# sudo psql postgresql://postgres@127.0.0.1:5432/postgres?sslmode=verify-full
# psql -U postgres -h 127.0.0.1 postgres
# psql "sslmode=verify-full dbname=postgres user=postgres host=localhost port=5432"

######
##  สำหรับ odoo server ให้ cert อยู่ที่ /opt/odoo/.postgresql/
##  สำหรับ command psql ให้ cert อยู่ที่ ~/.postgresql/

# psql -U odoo -h 172.22.33.21 RIL_Test ### password in odoo-server.conf # [OK]
# psql -h 172.22.33.21 -U odoo "dbname=RIL_Test sslmode=require"
# psql -h 172.22.33.21 -U odoo "dbname=RIL_Test sslmode=verify-ca"
# psql -h rilodoodb01.bjc.co.th -U odoo "dbname=RIL_Test sslmode=verify-full" 
# psql -h 172.22.33.21 -U odoo "dbname=RIL_Test sslmode=verify-ca sslrootcert=/home/riladmin/.postgresql/root.crt" # ระบุ certs path

psql "host=172.22.33.21 port=5432 dbname=RIL_Test user=odoo sslmode=verify-ca sslrootcert=/home/riladmin/.postgresql/root.crt sslcert=/home/riladmin/.postgresql/postgresql.crt sslkey=/home/riladmin/.postgresql/postgresql.key" # ระบุ certs path


# x  sudo psql postgresql://odoo@172.22.33.21:5432/RIL_Test?sslmode=verify-full
# x  sudo psql "dbname=RIL_Test host=127.0.0.1 port=5432 user=odoo sslmode=verify-full"


-- access psql command
$psql

-- list all databases
$\l

-- switch connect to a database
\c dbname

-- list all tables
$\dt

-- Describe a table
$\d table_name

```

# SQL command ...
```
select name, setting, short_desc  from pg_settings where name like '%ssl%';


:-- information about your RDS for PostgreSQL DB instance's SSL usage by process, client, and application by using the following query --:
SELECT datname as "Database name", usename as "User name", ssl, client_addr, application_name, backend_type, state
   FROM pg_stat_ssl
   JOIN pg_stat_activity
   ON pg_stat_ssl.pid = pg_stat_activity.pid
   ORDER BY ssl;
```

# Backup PGSQL
```
admin@COMSECAPP:~$ pg_dump -U postgres -d comsec -h localhost -p 5432 -n public --format=c --encoding=UTF-8 --verbose -f /home/admin/dbbackup/comsec_230719_1426.backup

-- schema only
pg_dump -U postgres -h localhost -p 5432 -s comsec > comsec_schemas_230710_1125.sql             [x]

-- data only
pg_dump -U postgres -h localhost -p 5432 -a comsec > comsec_data_230710_1125.sql                [x]

-- both schema & data
pg_dump -U postgres -h localhost -p 5432 comsec > comsec_full_230710_1125.sql                   [x]

pg_dump -U postgres -h localhost -p 5432 -d comsec > comsec_full_230710_1125.sql                [x]

-- both schema & data with `--no-owner` # it's restore pass
pg_dump -U postgres -h localhost -p 5432 --no-owner -d comsec > comsec_full_230719_1609.sql     [ok]

```

# Backup filestore with .zip
```
cd /home/admin/.local/share/Odoo/filestore/
zip -r comsec.zip comsec/
```

# Restore PGSQL
```

psql -U postgres -h localhost -p 5432 -d comsec < comsec_full_230719_1349.pgsql


pg_restore -U postgres -h localhost -p 5432 -d comsec_test comsec_230719_1426.backup


```


# Copy File from Linux to Windows
```
-- backup file --
pscp admin@172.19.4.46:/home/admin/dbbackup/comsec_* D:/BJC/Comsec/Backup/

-- storage file --
pscp admin@172.19.4.46:/home/admin/.local/share/Odoo/filestore/comsec.zip D:/BJC/Comsec/Backup/comsec.zip

```

# สร้าง Postgres user
```
$sudo -i -u postgres
$psql
postgres=# \du                  -- show users
postgres=# CREATE USER comsec;
CREATE ROLE                     -- result
postgres=# GRANT ALL PRIVILEGES ON DATABASE comsec TO comsec;
GRANT                           -- result
postgres=# ALTER USER comsec WITH PASSWORD 'nji9mko0';   -- change new password of user comsec
ALTER ROLE                      -- result
postgres=# ALTER USER comsec WITH SUPERUSER;   -- set role super user
ALTER ROLE
postgres=# ALTER USER comsec WITH CREATEDB;
ALTER ROLE
postgres=# ALTER USER comsec WITH CREATEROLE;
ALTER ROLE
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 comsec    | Superuser, Create role, Create DB                          | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}


```

# Step to Backup and Restore
```
- Goto Odoo web http://172.19.4.46/web/database/manager
- เลืือกฐานข้อมูลที่ต้องการ Backup กดปุ่ม Backup ระบบจะให้ใส่ ข้อมูล      
    - Master password:  xxxx-xxxx-xxxx
    - เลือก Database Name: comsec
    - Backup Format: zip (includes filestore)
    - กดปุ่ม Backup รอสักครู่ระบบจะทำการ Backup Database และ Filestore
- ทำการแตกไฟล์ comsec-YYYY-MM-DD_HH-mm-ss.zip ภายในประกอบไปด้วย
    - file: manifest.json -- ข้อมูลของ Odoo และ Module ที่ทำการติดตั้ง
    - file: dump.sql -- script file sql
    - folder: filestore/ -- folder ของ ไฟล์ต่างๆ ที import เข้ามาด้วย Odoo
- copy dump.sql ไฟล์ ไปยังเครื่องที่ต้องการจะทำการ Restore/กู้คืน
- ทำการกู้คืน แบบ manual
    - ทำการสร้างฐานข้อมูล เช่น comsec_xxxx
        -    admin@compname:~$ sudo -i -u postgres                                  -- เปลี่ยน user
        - postgres@compname:~$ psql                                                 -- เข้า psql command
        -           postgres=# CREATE DATABASE comsec_xxx;
        -           postgres=# ALTER DATABASE comsec_2 OWNER TO comsec;             -- change owner from postgres to comsec
        -           postgres=# \c comsec_xxx                                        -- เข้าใช้ฐานข้อมูล comsec_xxx
        
    - ใช้คำสั่ง เพื่อ restore database
             admin@compname:~$psql -U postgres -h localhost -p 5432 -d comsec_xxxx < dump.sql
    - ย้ายไฟล์ attachments backup = filestore/ 
        $ cp -R /dbbackup/comsec_xx-/filestore/* ./local/share/Odoo/filestore/comsec_xx/
    
```

# Postgres configuration
```
--- /etc/postgres/12/main/postgresql.conf

listen_addresses = '*' 


--- /etc/postgres/12/main/pg_hba.conf
# Database administrative login by Unix domain socket
local   all             postgres                                md5

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5

hostssl all             all              0.0.0.0/0              cert  clientcert=verify-full
```