# Introduction

Deploy lnmp(Linux, Nginx, MySQL, PHP7) using docker. [Read More...](http://www.jianshu.com/p/fcd0e542a6b2)

## How To Use?

#### Build Image

```shell
docker build --tag addcn/mysql -f mysql/Dockerfile .
docker build --tag addcn/php7 -f php7/Dockerfile .
docker build --tag addcn/nginx -f nginx/Dockerfile .
```

#### Run Container

```shell
sudo docker run --name mysql -p 3306:3306 -v /srv/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -itd addcn/mysql
sudo docker run --name php7 -p 9000:9000 -v /srv/webroot:/usr/local/nginx/html --link mysql:mysql -itd addcn/php7
sudo docker run --name nginx -p 80:80 -v /srv/webroot:/usr/local/nginx/html -v /srv/config/nginx:/usr/local/nginx/conf/servers -v /srv/logs/nginx:/usr/local/nginx/logs --link php7:php7 -itd addcn/nginx
```
#### 其它参数
```shell
--restart=[no-container|on-failure-container|always]
no - container：不重启
on-failure - container:退出状态非0时重启
always:始终重启
```
注意事项：
1.用docker自带的--link把多个容器链接在一起，有重启或升级的问题，比如很多容器都依赖于 db 这个容器，然后db容器重启了，重启时docker分配的ip会变，导致其他依赖于db的容器都要重启。
2.--link 链接的容器还有启动顺序的问题， 需要先启动db容器再启动其他依赖于db的容器， 这样导致 --link和--restart=always 不能一起用， 如果一起用会发现宿主机重启了， docker容器并没有全部重启，

#### Test PHP & MySQL

> vi /var/www/html/test.php

```php
<?php
//date
echo date("Y-m-d H:i:s")."<br />\n";

//mysql
try {
    $conn = new PDO('mysql:host=mysql;port=3306;dbname=mysql;charset=utf8', 'root', '123456');
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}
//$conn->exec('set names utf8');
$sql = "SELECT * FROM `user` WHERE 1";
$result = $conn->query($sql);
while($rows = $result->fetch(PDO::FETCH_ASSOC)) {
    echo $rows['Host'] . ' ' . $rows['User']."<br />\n";
}

//phpinfo
phpinfo();
?>
```

http://192.168.8.36/test.php


![docker-lnmp][1]

  [1]: docs/docker-lnmp.png
