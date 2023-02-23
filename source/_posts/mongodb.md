## mongodb 安装与使用

### 安装

> cd /usr/local
>
> sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.0.9.tgz
> 
> sudo tar -zxvf mongodb-osx-ssl-x86_64-4.0.9.tgz
> 
> sudo mv mongodb-osx-x86_64-4.0.9/ mongodb

### 配置环境变量

> export PATH=/usr/local/mongodb/bin:$PATH

### 配置数据的存放位置

```
sudo mkdir -p /usr/local/var/mongodb
sudo mkdir -p /usr/local/var/log/mongodb
sudo chown runoob /usr/local/var/mongodb
sudo chown runoob /usr/local/var/log/mongodb
```

### 运行

```
// 在后台运行
mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork

// 在前台运行
mongod --config /usr/local/etc/mongod.conf
```

### 查看是否已启动

```
ps aux | grep -v grep | grep mongod
```
此时，可以使用 npm install mongodb 来操作 mongo 数据库了。

### 结束运行

```
mongo
db.adminCommand({ "shutdown" : 1 })
```

### 查数据库数据

[下载 robo](https://robomongo.org/download)

### 查看 API

https://docs.mongodb.com/manual/reference/method/db.collection.find/

