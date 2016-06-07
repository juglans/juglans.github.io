---
layout: post
title:  "bitnami redmine版本由2.3.1升级至3.2.2过程"
date:   2016-06-07 16:30:07 +0800
categories: jekyll update
---
bitnami redmine版本由2.3.1升级至3.2.2过程
=============

## 环境： 

- 操作系统为ubuntu13.**版本，非长期支持版。
- 安装目录：/opt/redmine-2.3.1-0/
- 所有者用户：root

## 安装过程：

1. 备份2.3.1v数据库

~~~
sudo /opt/redmine-2.3.1-0/mysql/bin/mysqldump -u bitnami -p  bitnami_redmine > /home/erick/redmineback/2.3.1backup.sql
~~~

1. 备份2.3.1的配置文件和files目录（上传文件）

~~~
cp /opt/redmine-2.3.1-0/apps/redmine/htdocs/config/database.yml /home/erick/redmineback/
cp /opt/redmine-2.3.1-0/apps/redmine/conf/redmine.conf /home/erick/redmineback/
 cp /opt/redmine-2.3.1-0/apps/phpmyadmin/conf/phpmyadmin.conf /home/erick/redmineback/
cp /opt/redmine-2.3.1-0/apache2/conf/httpd.conf /home/erick/redmineback/
mkdir /home/erick/redmineback/files
cp /opt/redmine-2.3.1-0/apps/redmine/htdocs/files/* /home/erick/redmineback/files
cp /opt/redmine-2.3.1-0/apps/redmine/htdocs/public/favicon.ico /home/erick/redmineback/
~~~

1. 下载并安装最新redmine bitnami3.2.2安装包，目录与之前的区分开

~~~
/opt/redmine-3.2.2-0/
~~~

1. 备份3.2.2的数据库

~~~
sudo /opt/redmine-3.2.2-0/mysql/bin/mysqldump -u bitnami -p  bitnami_redmine > /home/erick/redmineback/3.2.2backup.sql
~~~

1. 删除并创建3.2.2的数据库

~~~
sudo /opt/redmine-3.2.2-0/mysql/bin/mysql -u bitnami -p
DROP DATABASE bitnami_redmine;
CREATE DATABASE bitnami_redmine;
quit  
~~~

1. 用2.3.1的备份数据恢复到3.2.2的数据库中

> 通过phpmyadmin的导入功能实现

1. 迁移3.2.2的数据库

~~~
cd /opt/redmine-3.2.2-0/apps/redmine/htdocs
sudo /opt/redmine-3.2.2-0/ruby/bin/ruby bin/rake db:migrate RAILS_ENV=production
~~~

1. 修改3.2.2的配置文件

设置apache，启用gogs,修改apache参数文件:

~~~
/opt/redmine-3.2.2-0/apache2/conf/httpd.conf
~~~

在最后增加以下配置信息：

> 
~~~
<VirtualHost *:80>
     DocumentRoot  "/opt/redmine-3.2.2-0/apps/redmine/htdocs/public/"
     ServerName qa.yncic.com
     ServerAlias qa.yncic.com
</VirtualHost>
<VirtualHost *:80>
        ServerAdmin webmaster@domain.tld
        ServerName git.yncic.com
        ProxyRequests Off
        <Proxy *>
        Order deny,allow
        Allow from all
        </Proxy>
        ProxyPass / http://localhost:3000/
        ProxyPassReverse / http://localhost:3000/
</VirtualHost>
~~~

1. 拷贝files目录到3.2.2中

~~~
sudo cp -r /opt/redmine-2.3.1-0/apps/redmine/htdocs/files/* /opt/redmine-3.2.2-0/apps/redmine/htdocs/files/
sudo chown -R daemon:daemon /opt/redmine-3.2.2-0/apps/redmine/htdocs/files/*
~~~

1.  拷贝icon图标

~~~
sudo cp /opt/redmine-2.3.1-0/apps/redmine/htdocs/public/favicon.ico /opt/redmine-3.2.2-0/apps/redmine/htdocs/public/favicon.ico
~~~

***

## 参考资料

* https://wiki.bitnami.com/Applications/BitNami_Redmine#How_to_upgrade_Redmine.3f
* https://wiki.bitnami.com/Components/MySQL#How_to_create_a_database_backup.3f
* http://fableking.iteye.com/blog/1395366
* http://blog.csdn.net/shishuo365/article/details/45999053

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

