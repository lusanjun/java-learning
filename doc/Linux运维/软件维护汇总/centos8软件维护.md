## centos8软件维护

### 1. JDK

1. 下载：https://www.oracle.com/java/technologies/downloads/  选择版本，如 jdk-8u321-linux-x64.tar.gz

2. 上传至服务器目录，如 /home

3. 解压缩

   ```shell
   tar -xvf jdk-8u321-linux-x64.tar.gz
   ```

4. 配置环境变量

   ```shell
   vim /etc/profile
   ```

5. 末尾添加以下内容，JAVA_HOME 目录以实际为准。

   ```
   JAVA_HOME=/home/jdk1.8.0_321
   CLASSPATH=$JAVA_HOME/lib/
   PATH=$PATH:$JAVA_HOME/bin
   export PATH JAVA_HOME CLASSPATH
   ```

6. 使配置生效

   ```shell
   source /etc/profile
   ```

7. 验证安装结果

   ```shell
   java -version
   ```

8. 出现以下字样，即安装成功

   ```shell
   java version "1.8.0_321"
   Java(TM) SE Runtime Environment (build 1.8.0_321-b07)
   Java HotSpot(TM) 64-Bit Server VM (build 25.321-b07, mixed mode)
   ```

### 2. Maven

1. 下载：https://maven.apache.org/download.cgi 选择合适的版本，如 apache-maven-3.8.5-bin.tar.gz

2. 上传至服务器目录，如 /home

3. 解压缩

   ```shell
   tar -xvf apache-maven-3.8.5-bin.tar.gz
   ```

4. 配置环境变量

   ```shell
   vim /etc/profile
   ```

5. 末尾添加以下内容，MAVEN_HOME 目录以实际为准。

   ```
   export MAVEN_HOME=/home/apache-maven-3.8.5
   export PATH=$PATH:$MAVEN_HOME/bin
   ```

6. 使配置生效

   ```shell
   source /etc/profile
   ```

7. 验证安装结果

   ```shell
   mvn -v
   ```

8. 出现以下字样，即安装成功

   ```shell
   Apache Maven 3.8.5 (3599d3414f046de2324203b78ddcf9b5e4388aa0)
   Maven home: /home/apache-maven-3.8.5
   Java version: 1.8.0_321, vendor: Oracle Corporation, runtime: /home/jdk1.8.0_321/jre
   ```

### 3. MySQL

#### 3.1 yum安装

1. 添加 MySQL yum Repository

   ```shell
   wget https://dev.mysql.com/get/mysql80-community-release-el8-3.noarch.rpm
   ```

   或者手动下载，上传至服务器目录

2. repo 安装

   ```shell
   rpm -ivh mysql80-community-release-el8-3.noarch.rpm 
   ```

   可以查看安装结果

   ```shell
   yum repolist enabled | grep "mysql.*-community.*"
   ```

   显示如下

   ```shell
   mysql-connectors-community MySQL Connectors Community
   mysql-tools-community      MySQL Tools Community
   mysql80-community          MySQL 8.0 Community Server
   ```

3. 安装 mysql

   ```shell
    yum install mysql-community-server
   ```

   如果出现 No match for argument: mysql-community-server错误，

   或 Error: Unable to find a match: mysql-community-server ，执行

   ```shell
   yum module disable mysql
   ```

   再次安装即可

4. 启动 mysql

   ```shell
   systemctl start mysqld
   ```

   查看 mysql 服务状态

   ```shell
   systemctl status mysqld
   ```

5. 登录 mysql

   服务安装完成会生成一个 root 用户和临时密码，显示密码

   ```shell
   grep 'temporary password' /var/log/mysqld.log
   ```

   如果没有显示，查看 /var/log/mysql/mysqld.log 日志，可能版本不同 root 用户并没有创建密码。

   登录 mysql，输入密码。没有密码的，直接回车跳过。

   ```shell
   mysql -uroot -p
   ```

6. 更改 root 用户密码，

   ```shell
   alter user 'root'@'localhost' identified by 'MyPwd@123456';
   ```

   简单密码修改若不成功，是因为实施的默认密码策略 validate_password 要求密码至少包含一个大写字母，一个小写字母，一位数字和一个特殊字符，并且密码总长度至少为8个字符。

   可以更改密码策略，修改密码的复杂程度，0为最低

   ```shell
   set global validate_password.policy=0;
   ```

   修改密码的长度，默认为8

   ```shell
   set global validate_password.length=6;
   ```

   修改完毕，可设置简单密码，不推荐简单密码，仅供测试方便使用！

7. 允许远程访问

   ```shell
   use mysql;
   update user set host='%' where user='root';
   flush privileges;
   ```

8. Navicat 连接

   如果报错：2059-Authentication pugin ‘caching_sha2_password’ cannot be loaded

   原因是8.0+版本的加密方式为caching_sha2_passsword，与5版本的mysql_native_password方式不同。解决方法，更新用户的加密方式：

   ```shell
   alter user 'root'@'%' identified with mysql_native_password by '123456';
   flush privileges;
   ```

9. 卸载 mysql

   停止服务

   ```shell
   systemctl stop mysqld
   ```

   查看已安装MySQL情况

   ```shell
   rpm -qa | grep -i mysql
   ```

   卸载以上内容

   ```shell
   rpm -ev ...
   ```

   查找mysql相关文件，删除文件，手动删除 my.cnf

   ```shell
   find -name mysql
   rm -rf ...
   rm -rf /etc/my.cnf
   ```

### 4. Redis

1. 下载：https://redis.io/download/ 选择合适的版本，如 redis-6.2.6，上传至服务器目录，如 /home

   或直接在服务器下载

   ```shell
   wget https://download.redis.io/releases/redis-6.2.6.tar.gz
   ```

2. 解压缩

   ```shell
   tar -xvf redis-6.2.6.tar.gz
   ```

3. 如果系统的 C 语言依赖没有安装，先安装依赖

   ```shell
   yum install -y gcc-c++
   ```

   升级 gcc

   ```shell
   yum install -y centos-release-scl scl-utils-build
   yum install -y devtoolset-9-toolchain
   scl enable devtoolset-9 bash
   ```

4. 进入目录，编译

   ```shell
   cd redis-6.2.6
   make
   ```

5. 指定目录安装，如安装在 /usr/local/redis

   ```shell
   mkdir -p /usr/local/redis
   make PREFIX=/usr/local/redis install
   ```

6. 启动，进入 bin 目录启动

   ```shell
   cd /usr/local/redis/bin
   ```

7. 后台启动

   复制 redis 配置文件 redis.conf ，到 bin 目录。

   ```shell
   cp /home/redis-6.2.6/redis.conf /usr/local/redis/bin
   ```

   修改 redis.conf 文件，把 daemonize 修改为 yes

8. 方便以后 RDM 连接，redis.conf 可配置为

   ```
   bind 0.0.0.0 -::1
   protected-mode no
   port 6379
   # 后台启动
   daemonize yes
   requirepass MyPwd@123456 #密码
   ```

9. 以配置文件启动

   ```shell
   ./redis-server redis.conf
   ```

10. 进入 redis

    ```shell
     ./redis-cli
    ```

11. 密码登录

    ```shell
    127.0.0.1> auth MyPwd@123456
    ```

12. 配置服务器启动

    系统服务目录里创建 redis.service 文件

    ```shell
    vim /etc/systemd/system/redis.service
    ```

    写入以下内容

    ```shell
    [Unit]
    Description=redis-server
    After=network.target
    
    [Service]
    Type=forking
    ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/bin/redis.conf
    PrivateTmp=true
    
    [Install]
    WantedBy=multi-user.target
    ```

    重载系统服务

    ```shell
    systemctl daemon-reload
    ```

    可以使用系统服务开启关闭 redis

    ```shell
    systemctl start redis.service
    systemctl stop redis.service
    systemctl restart redis.service
    ```

    可以服务器开机自启动

    ```shell
    systemctl enable redis.service
    ```

### 5. MongoDB

#### 5.1 yum安装

1. 创建源文件

   ```shell
   vim /etc/yum.repos.d/mongodb-org-6.0.repo
   ```

   源文件内容

   ```
   [mongodb-org-6.0]
   name=MongoDB Repository
   baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/6.0/x86_64/
   gpgcheck=1
   enabled=1
   gpgkey=https://www.mongodb.org/static/pgp/server-6.0.asc
   ```

2. 安装：

   ```shell
   sudo yum install -y mongodb-org
   ```

3. 启动

   ```shell
   sudo systemctl start mongod
   ```

   如果遇到以下错误

   ```shell
   Failed to start mongod.service: Unit mongod.service not found.
   ```

   则执行以下命令，再启动

   ```shell
   sudo systemctl daemon-reload
   ```

4. 查看服务状态

   ```shell
   sudo systemctl status mongod
   ```

   看到`Active: active (running)`则启动成功

   如果启动失败，可以执行以下命令，再重新启动

   ```shell
   sudo rm -rf /tmp/mongodb-27017.sock
   ```

5. 关闭服务

   ```shell
   sudo systemctl stop mongod
   ```

6. 重启服务

   ```shell
   sudo systemctl restart mongod
   ```

7. 登录

   ```shell
   mongosh
   ```

8. 创建用户名密码

   ```shell
   use admin
   db.createUser({
     user: 'root',    // 用户名（自定义）
     pwd: 'abc123',    // 密码（自定义）
     roles:[{
       role: 'root',   // 选择角色属性，这里选择"超级账号"
       db: 'admin'     // 指定数据库
     }]
   })
   ```

9. 修改`mongod.conf`开启安全认证

   ```shell
   vim /etc/mongod.conf
   
   #允许远程访问
   net:
     port: 27017
     bindIp: 0.0.0.0
   security:
     authorization: enabled
   ```

10. 重启服务，使用用户密码登录

    ```shell
    mongosh  --port 27017 -u "admin" -p "abc123" --authenticationDatabase "admin"
    ```

11. 卸载：

    - 先关闭服务

      ```shell
      sudo service mongod stop
      ```

    - 清除服务包

      ```shell
      sudo yum erase $(rpm -qa | grep mongodb-org)
      ```

    - 清除数据目录

      ```shell
      sudo rm -r /var/log/mongodb
      sudo rm -r /var/lib/mongo
      ```

#### 5.2 压缩包安装

1. 下载压缩包：https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel80-6.0.9.tgz，上传至`/usr/local/`

   或在目录下执行

   ```shell
   wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel80-6.0.9.tgz
   ```

2. 解压缩

   ```shell
   tar -zxvf mongodb-linux-x86_64-rhel80-6.0.9.tgz mongodb-linux-x86_64-rhel80-6.0.9/
   mv mongodb-linux-x86_64-rhel80-6.0.9 mongodb
   vim /etc/profile
   ```

3. 配置环境变量

   ```
   export MONGODB_HOME=/usr/local/mongodb
   export PATH=$PATH:$MONGODB_HOME/bin
   ```

4. 刷新配置

   ```shell
   source /etc/profile
   ```

5. 验证成功

   ```shell
   mongod --version
   ```

6. 创建数据目录和日志目录

   ```shell
   mkdir -p /usr/local/mongodb/data
   mkdir /usr/local/mongodb/logs
   ```

7. 新建配置文件 mongo.conf

   ```
   dbpath=/usr/local/mongodb/data
   logpath=/usr/local/mongodb/logs/mongodb.log
   logappend=true
   bind_ip=0.0.0.0
   auth=false
   port=27017
   fork=true
   journal=false
   ```

8. 安装 mongo shell

   ```shell
   wget wget https://downloads.mongodb.com/compass/mongosh-1.10.5-linux-x64.tgz
   tar -zxvf mongosh-1.10.5-linux-x64.tgz 
   cp mongosh-1.10.5-linux-x64.tgz/bin/mongosh /usr/local/mongodb/bin
   ```

9. 启动 mongodb

   ```shell
   mongod --config mongo.conf
   ```

10. 进入 mongodb

    ```shell
    # /usr/local/mongodb/bin
    mongosh
    
    test> show dbs
    admin   40.00 KiB
    config  60.00 KiB
    local   72.00 KiB
    test> 
    ```

    

