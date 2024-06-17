---
date: 2024-04-16
---
# 简介
https://www.bilibili.com/video/BV1zy4y1L7HF/?spm_id_from=333.337.search-card.all.click&vd_source=37dcb67dc2fcca470ec53b1511e0412f
Canal是阿里巴巴旗下的一款开源项目，纯Java开发。基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了MySQL（也支持mariaDB）
![[Pasted image 20240416195501.png|825]]
debezium：1.x是单机版本；2.x有了分布式版本；
## 异构数据
结构不同的数据
# Kylin-V10 环境准备
## 安装docker
## 开启ip_forward为Docker运行做准备，创建Docker网桥flink-network
```sh
cat>>/etc/sysctl.conf << -'EOF‘
net.ipv4.ip_forward=1
vm.max_map_count=655360
EOF
sysctl -p

docker network create flink-network
```
## 创建SQL，并Docker构建MySQL8实例
注：不用docker安装的方式也行，这是为了迅速测试。默认，docker在mysql8实例上开启了binlog，所以docker要拉取mysql8，24/06/04拉取后查看版本为：8.0.27
```bash
[root@kylin ~]# mkdir -p /var/flinkcdc/mysql-script
[root@kylin ~]# cat /var/flinkcdc/mysql-script/init.sql
create database mydb;
use mydb;
CREATE TABLE products (
  id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description VARCHAR(512)
);
ALTER TABLE products AUTO_INCREMENT = 101;

INSERT INTO products
VALUES (default,"scooter","Small 2-wheel scooter"),
(default,"car battery","12V car battery"),
(default,"12-pack drill bits","12-pack of drill bits with sizes ranging from #40 to #3"),
(default,"hammer","12oz carpenter's hammer"),
(default,"hammer","14oz carpenter's hammer"),
(default,"hammer","16oz carpenter's hammer"),
(default,"rocks","box of assorted rocks"),
(default,"jacket","water resistent black wind breaker"),
(default,"spare tire","24 inch spare tire");

CREATE TABLE orders (
  order_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
  order_date DATETIME NOT NULL,
  customer_name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 5) NOT NULL,
  product_id INTEGER NOT NULL,
  order_status BOOLEAN NOT NULL -- Whether order has been placed
) AUTO_INCREMENT = 10001;

INSERT INTO orders
VALUES (default, '2020-07-30 10:08:22', 'Jark', 50.50, 102, false),
(default, '2020-07-30 10:11:09', 'Sally', 15.00, 105, false),
(default, '2020-07-30 12:00:30', 'Edward', 25.25, 106, false);

[root@kylin ~]# docker run -p 3306:3306 --network flink-network --name mysql-source -v /var/flinkcdc/mysql-script:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=cf -d mysql:8

[root@kylin ~]# docker exec -it mysql-source mysql -uroot -pcf
mysql> show variables like '%time_zone%';
#  输出中time_zone的Value为+08:00才行；如果是SYSTEM不行
mysql> set time_zone='+8:00';
#  当前生效
mysql> set persist time_zone='+8:00';
#  更换会话后time_zone调整仍然有效，exit后再次登录可看到变化

mysql> show databases;
mysql> use mydb;
mysql> show tables;
+----------------+
| Tables_in_mydb |
+----------------+
| orders         |
| products       |
+----------------+
2 rows in set (0.01 sec)

mysql> select * from orders;
......
3 rows in set (0.00 sec)

mysql> select * from products;
......
9 rows in set (0.00 sec)
```













