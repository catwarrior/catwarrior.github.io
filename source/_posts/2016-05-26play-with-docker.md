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

- backup the database folder the host current folder.
``` bash
docker run --rm -v $(pwd):/backup -v amysql-volume:/dbdata debian:jessie tar cvf /backup/backup.tar /dbdata
```
