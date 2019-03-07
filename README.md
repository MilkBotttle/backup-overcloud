# Backup Overcloud control
# Version
Pike
## MySQL backup
1. Login any controller or databae node
2. Get DB root password
```
sudo -i
mkdir -p /root/mysql_backup/
MYSQLDBPASS=$(hiera -c /etc/puppet/hiera.yaml mysql::server::root_password)
```
3. Backup Openstack data DB
```
mysql -uroot -p$MYSQLDBPASS -e "select distinct table_schema from information_schema.tables where engine='innodb' and table_schema != 'mysql';" \
      -s -N | xargs mysqldump -uroot -p$MYSQLDBPASS --single-transaction --databases > /root/mysql_backup/openstack_databases-`date +%F`-`date +%T`.sql
```
4. Backup Openstack user DB
```
mysql -uroot -p$MYSQLDBPASS -e "SELECT CONCAT('\"SHOW GRANTS FOR ''',user,'''@''',host,''';\"') FROM mysql.user where (length(user) > 0 and user NOT LIKE 'root')" \
      -s -N | xargs -n1 mysql -uroot -p$MYSQLDBPASS -s -N -e | sed 's/$/;/' > /root/mysql_backup/openstack_databases_grants-`date +%F`-`date +%T`.sql
```
## Redis backup
1. Login any controller or redis ndoe
2. Get redis ip, password
```
REDISIP=$(hiera -c /etc/puppet/hiera.yaml redis_vip)
REDISPASS=$(hiera -c /etc/puppet/hiera.yaml tripleo::haproxy::redis_password)
```
3. Check connetcion
```
redis-cli -a $REDISPASS -h $REDISIP ping
```
4. Backup redis database
```
redis-cli -a $REDISPASS -h $REDISIP bgsave
```
Backup save in `/var/lib/redis`

## Backup filesystem
```
mkdir -p /root/filesystem_backup
tar --xattrs --ignore-failed-read \
    -zcvf /root/filesystem_backup/fs_backup-`date '+%Y-%m-%d-%H-%M-%S'`.tar.gz \
    /etc/nova \
    /var/log/nova \
    /var/lib/nova \
    --exclude /var/lib/nova/instances \
    /etc/glance \
    /var/log/glance \
    /var/lib/glance \
    /etc/keystone \
    /var/log/keystone \
    /var/lib/keystone \
    /etc/httpd \
    /etc/cinder \
    /var/log/cinder \
    /var/lib/cinder \
    /etc/heat \
    /var/log/heat \
    /var/lib/heat \
    /var/lib/heat-config \
    /var/lib/heat-cfntools \
    /etc/rabbitmq \
    /var/log/rabbitmq \
    /var/lib/rabbitmq \
    /etc/neutron \
    /var/log/neutron \
    /var/lib/neutron \
    /etc/corosync \
    /etc/haproxy \
    /etc/logrotate.d/haproxy \
    /var/lib/haproxy \
    /etc/openvswitch \
    /var/log/openvswitch \
    /var/lib/openvswitch \
    /etc/ceilometer \
    /var/lib/redis \
    /etc/sysconfig/memcached \
    /etc/gnocchi \
    /var/log/gnocchi \
    /etc/aodh \
    /var/log/aodh \
    /etc/panko \
    /var/log/panko \
    /etc/ceilometer \
    /var/log/ceilometer
```
# backup-overcloud
