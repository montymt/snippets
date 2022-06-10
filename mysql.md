<!--ts-->
<!--te-->
# MySQL

## 编译安装

版本：mysql-5.7.34，centos-7.8.2003，2G内存，4核CPU

CentOS默认会安装mariadb-libs，需要卸载

`rpm -qa | grep mariadb-libs`

`rpm -e mariadb mariadb-libs --nodeps`

源码下载：`wget https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-5.7.34.tar.gz`

安装依赖：`yum -y install make gcc-c++ cmake openssl-devel ncurses-devel`

编译：

```
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DWITH_EMBEDDED_SERVER=1 -DENABLED_LOCAL_INFILE=1 \
-DWITH_BOOST=/root/mysql-5.7.34/boost

make -j 4
# 报c++: internal compiler error 因为开了4个线程，导致内存不足
make
make install
groupadd mysql
useradd -s /sbin/nologin -M -g mysql mysql
# 配置文件：/etc/my.cnf
mkdir /usr/local/mysql/var
chown mysql.mysql /usr/local/mysql/var
/usr/local/mysql/bin/mysqld --initialize-insecure --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var --user=mysql
chgrp -R mysql /usr/local/mysql/.
\cp support-files/mysql.server /etc/init.d/mysql
cat > /etc/ld.so.conf.d/mysql.conf<<EOF
    /usr/local/mysql/lib
    /usr/local/lib
EOF
ldconfig
ln -sf /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -sf /usr/local/mysql/include/mysql /usr/include/mysql
/etc/init.d/mysql start


ln -sf /usr/local/mysql/bin/mysql /usr/bin/mysql
ln -sf /usr/local/mysql/bin/mysqldump /usr/bin/mysqldump
ln -sf /usr/local/mysql/bin/myisamchk /usr/bin/myisamchk
ln -sf /usr/local/mysql/bin/mysqld_safe /usr/bin/mysqld_safe
ln -sf /usr/local/mysql/bin/mysqlcheck /usr/bin/mysqlcheck

#UPDATE mysql.user SET authentication_string=PASSWORD('root') WHERE User='root';
#flush privileges;
```


## 二进制安装

这种方式比较稳定，不需要编译，小内存也可以

```
rpm -e mariadb-libs --nodeps
wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.35-linux-glibc2.12-x86_64.tar.gz
yum -y install libaio numactl
groupadd mysql
useradd -M -g mysql -s /sbin/nologin mysql
tar xf mysql-5.7.35-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.35-linux-glibc2.12-x86_64 /usr/local
cd /usr/local && ln -sfv mysql-5.7.35-linux-glibc2.12-x86_64 mysql
bin/mysqld --initialize-insecure --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var --user=mysql
cp my.cnf /etc/my.cnf
cp support-files/mysql.server /etc/init.d/mysql
/etc/init.d/mysql start
ln -sf /usr/local/mysql/bin/mysql /usr/bin/mysql
#alter user root@localhost identified by 'root'
#flush privileges;
```


## 增加innodb索引长度

```
set global innodb_large_prefix = 1;
set global innodb_file_format = BARRACUDA;
```

## 数据定义DDL

`create table xx(id int unsigned primary key auto_increment)engine=innodb default-charset=utf8mb4`

`alter table student add address varchar(191) not null default '' comment 'address'`

`alter table student modify address varchar(150) not null default ''`

`alter table student change sex gender enmu('male', 'female') not null default 'male'`

`alter table a set default 1`

`alter table a drop default`

`ALTER TABLE user AUTO_INCREMENT=100`

`drop index xxx on student`

`alter table student drop index xxx`

## 数据操纵DML

`insert into student (name, age) values ('a', 1), ('b', 2)`

`update student set age=4 where name='b'`

`delete from student where age > 35`

## 数据查询DQL

`select a,b from c left join d on c.e=d.f where g=h group by i having j > 0 order by k limit z`

子查询，常用于SQL优化

### where子查询

`select cat_id,good_id,good_name from goods where good_id in(select max(good_id) from goods group by cat_id)`

### from子查询

分组并找出每组最大id：

`select * from (select cat_id,good_id,good_name from goods order by cat_id asc, good_id desc) as tep group by cat_id`

挂科两门以上的平均分：

`select name ,avg(score) as average from stu where name in(select name from (select name,count(*) as gk from stu where score<60 group by name having gk>=2)as tmp)
group by name`

### in子查询

`select * from department where did in(SELECT did from employee where age=20)`

### exists子查询

`select * from department where EXISTS (SELECT did from employee where age>21)`

### join子查询

`select * from a left join (select cid from e where f > 0) as b on b.cid=a.cid group by a.fid`

### GROUP_CONCAT

GROUP_CONCAT将多条数据合并为一条

`SELECT GROUP_CONCAT(mid) FROM (SELECT MIN(id) as mid FROM user WHERE state=0 GROUP BY age,gender) user_t`

## 数据控制DCL

`GRANT ALL PRIVILEGES ON *.* TO 'bloguser'@'%' IDENTIFIED BY 'mypassword'`

`flush privileges`

`revoke all privileges on *.* from bloguser@%`
