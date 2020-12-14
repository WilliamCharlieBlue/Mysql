# 01. Mysql 8 部署 (CentOS 8)
## 1.1 安装mysql
### 查看CentOS的内核版本号
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\c8697f5180884be0a3c47e27ec2db62b\clipboard.png)
### 根据内核版本号，选择相应的RPM包
- RPM包对应地址：https://dev.mysql.com/downloads/repo/yum/
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\c8697f5180884be0a3c47e27ec2db62b\clipboard.png)
### 直接复制对应链接
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\c8697f5180884be0a3c47e27ec2db62b\clipboard.png)
### wget 下载rpm包
```shell
wget https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\691a50fef3e64b9195a15914a7561994\clipboard.png)
### yum 安装rpm包
```shell
yum -y install mysql80-community-release-el8-1.noarch.rpm
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\27737caa30c542deb55334e0529934a8\clipboard.png)
### yum 安装mysql
```shell
# yum -y install mysql-community-server 
# 询问了度娘，CentOS8默认源安装的就是mysql8
# 不需要上述的wget和yum，直接使用下面的yum语句
yum -y install mysql-server
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\f33a8a94bfd84fb8a4f18df5f1f7c2e5\clipboard.png)
###  安装成功后，启动mySQL
```shell
systemctl start  mysqld.service
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\f33a8a94bfd84fb8a4f18df5f1f7c2e5\clipboard.png)
### 查看mySQL运行状态
```shell
systemctl status mysqld.service
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\9477122727e34bbba4ff52fc265a6a7c\clipboard.png)

## 1.2 终止和重启
### 终止mysql服务
```shell
systemctl stop mysqld.service
```
### 重启mysql
```shell
service mysqld restart
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\7ece98b3c2e946ef8671580364353f2f\clipboard.png)

## 1.3 密码与权限
### CentOS8，从默认源安装mysql8没有密码，可以直接mysql进入使用
```shell
mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
mysql> quit
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\ac55c91040b549ee86b8f8932830e6ea\微信图片编辑_20201214221701.jpg)
### 通过密码访问mysql
```shell
mysql -uroot -p123456
```
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\ec1d5edcc97c4ec6a8f79baabbda469c\clipboard.png)
### 设置密码等级失败了
- 暂时还没找到原因
![img](C:\Users\Matrix\AppData\Local\YNote\data\heshihoshi@163.com\c3f3b68cc8e349ef885df46db4e5456b\clipboard.png)