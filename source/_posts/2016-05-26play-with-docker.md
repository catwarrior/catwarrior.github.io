## Read `The Docker Book`
`Docker image to Docker container` just like `Source code to Application`

Docker image
bootfs -> rootfs -> ufs1 -> ufs2 ...

## Database 
$ docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

import: mysql -hmysql 6d < Dump20160515.sql
export: mysqldump -hmysql -uroot -proot 6d > /opt/Dump20160608.sql
