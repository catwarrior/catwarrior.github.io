## Read `The Docker Book`
`Docker image to Docker container` just like `Source code to Application`

Docker image
bootfs -> rootfs -> ufs1 -> ufs2 ...

## Database 
$ docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

import: mysql -hmysql 6d < Dump20160515.sql
export: mysqldump -hmysql -uroot -proot 6d > /opt/Dump20160608.sql

## A simple way to backup and restore the mysql data.

 - prepare
``` bash
docker volume create --name amysql-volume
docker run --name amysql -v amysql-volume:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6
```

 - access to mysql
``` bash
docker run -it --link amysql:mysql --rm mysql:5.6 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
```
- test remove the container, create another container, the data still there
``` bash
docker run --name amysqlbackup -v amysql-volume:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6
docker run -it --link amysqlbackup:mysql --rm mysql:5.6 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
```

- backup the data volumes to the host current folder.
``` bash
docker run --rm -v $(pwd):/backup -v amysql-volume:/dbdata debian:jessie tar cvf /backup/backup.tar /dbdata
```
- restore the data volumes
``` bash
docker run --rm -v $(pwd):/backup -v amysql-volume:/dbdata debian:jessie bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

## Fadfun docker
fad_backend
   nginx + code.
   
   v: /fad/web/data
      /fad/log
      
mysql
   /var/lib/mysql

data-volumns:
   /fad/web/data
   /fad/log
   /var/lib/mysql
   
fad_data
fad_app
fad_mysql
fad_php

### Step1. create data container. `fad_data`
```
docker run --name=fad_data -p 3306:3306 -v /var/lib/mysql -v /fad/web/data -v /fad/log -ti -d busybox echo ALL-Fad-Data
```
### Step2. create mysql container. `fad_mysql`
```
docker run --name=fad_mysql --volumes-from fad_data -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6
```
### Step3. create php container. `fad_php`
```
docker run --name=fad_php -p 9000:9000 --link fad_mysql:mysql -d php:5.6-fpm
```

### Step4. create app container.(www root in it.)  `fad`
```
probably a custom image(based on nginx), with code in it.
```
for dev
```
docker run --name=fad -d -p 80:80 --link fad_php:phpfpm --volumes-from fad_data -v "$PWD":/usr/share/nginx/html nginx:stable
```



### Import database dump.
```
docker run -it --link fad_mysql:mysql --rm -v "$PWD":/db mysql:5.6 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD" 6d < /db/Dump20160515.sql'
```

### Update settting in database to adopt new environment.

http://learn.elgg.org/en/1.12/admin/backup-restore.html#creating-a-usable-backup-automatically
``` sql

UPDATE `elgg_datalists` SET `value` = "/usr/share/nginx/html/" WHERE `name` = "path";

UPDATE `elgg_datalists` SET `value` = "/fad/web/data/" WHERE `name` = "dataroot";

UPDATE `elgg_sites_entity` SET `url` = "http://mynewdomain.com";

UPDATE elgg_metastrings set string = '/fad/web/data/' WHERE id = (SELECT value_id from elgg_metadata where name_id = (SELECT * FROM (SELECT id FROM elgg_metastrings WHERE string = 'filestore::dir_root') as ms2) LIMIT 1);

```
