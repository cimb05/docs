# F5部署

##	一、环境准备

### 1.	部署说明

- 首先确定好需要部署的服务器，通常部署在linux服务器上。服务器包括`数据库服务器`和`应用服务器`，`应用服务器`根据不同客户给的机器的数据差异，也会有不同的部署情况。**所以务必明确每一台机器部署的应用，配置好应用与服务器之间的IP端口信息**。

- 数据库服务器通常客户会帮我们安装好，我们需要知道数据库的`tns地址信息`。

### 2.	部署顺序

	1.创建用户
	2.安装jdk
	3.安装Redis
	4.安装zk
	5.安装微服务模块
	6.安装nginx
	7.Tomcat部署
	8.防火墙操作
	9.申请授权，安装授权证书

- 如果客户只提供了一台服务器来部署，就将上面的部署步骤全部部署到该服务器，如果有两台，可以将redis、zk、微服务模块部署在一台服务器，将nginx、tomcat部署在另一台服务器上，或者根据要求部署。

### 3. 连接环境
- 将客户提供的服务器配置到我们的linux连接工具上(如Xshell)，登录环境
- 如果所有应用程序部署在一个服务器上，直接进行下面的部署
- 如果有多个服务器，先连接部署redis、zk、微服务的服务器，然后开始下面的部署步骤。

### 4. 微服务模块准备
- 首先应该明确项目中需要用到哪些微服务模块。

- 其中对应关系如下：
```
ams-client-group:		网关
ams-server-aml-group:		反洗钱
ams-server-assessment-group:	考核
ams-server-common-group:	公共基础
ams-server-customer-group:	客户
ams-server-learn-group:		学习
ams-server-report-group:	报表
ams-server-service-group:	服务
ams-server-staff-group		人员
```
- 最基础的微服务模块包括`网关`、`公共基础`、`人员`。
### 5.  微服务配置
- 进入每个微服务的`config`目录中,每个微服务都有`bootstrap.yml`和`application-dev.yml`配置文件。
- `application-dev.yml`也可能是`application-prod.yml`，根据具体情况而定
- 打开`bootstrap.yml`文件，修改`address`为当前的服务器ip（有其它涉及到ip的配置也需要进行修改）。
```
  registry:
    protocol: zk
    address: (当前的服务器ip):2181
    username: amscli
    password: apexsoft
```
- 打开`application-dev.yml`文件，将其中的`所有url`的`ip：端口/数据库服务名`改成实际的数据库地址。
```
    mainDs:
      type: ORACLE
      url: jdbc:oracle:thin:@218.66.59.169:3701/qhcrm
      username: livebos
      password: abs
...
```
- **确保每一个微服务都配置完成且正确。**



##	二、执行部署

### 1.	创建用户

```
useradd ftq
```

```
passwd  ftq
```

### 2. 安装jdk
- 有的客户装好系统之后，已经自带jdk环境，如果自带的jdk版本高于1.8，即可跳过此步骤，使用系统自带的jdk。
```
rpm -qa | grep java
```

- 执行出来没有jdk或者jdk版本小于1.8，则重装一个1.8版本以上的jdk。
- 创建目录
```
/home/ftq/java
```
- 上传压缩文件到该目录
```
tar -zxvf /home/ftq/java/jdk-8u221-linux-x64.tar.gz
```
- 配置环境变量
```
vi /etc/profile
```
- 增加以下内容
```
JAVA_HOME=/home/ftq/java/jdk1.8.0_221
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```
- 刷新环境变量
```
source /etc/profile
```
- 验证安装情况
```
java -version
```

### 3.	安装redis

- 创建redis目录

```
mkdir /home/ftq/redis
```

- 上传redis安装程序`redis-4.0.2.tar.gz`到创建的目录下。

```
tar -zxvf redis-4.0.2.tar.gz
```
- 进入解压后的目录：

```
make install
```

- 如果报错可以尝试：`make MALLOC=libc`

- 修改配置文件 `redis.conf`的以下四项配置
- redis.conf 修改之前的配置项：

```
port	6379				    	(保持不变)
bind 	127.0.0.1		 	 	(修改成当前服务器的ip)	
protected-mode yes				(原来是yes，修改成no)
# requirepass foobared				(取消次配置的注释并修改foobared为redisForApexsoft)
```
- 修改之后的配置项
```
port	6379
bind 	(修改成当前服务器的ip)		 	 		
protected-mode no
requirepass redisForApexsoft
```

- 在redis的根目录启动redis  (停止命令：`redis-cli shutdown`)

```
nohup src/redis-server redis.conf >/dev/null 2>&1 &
```

- 查看Redis状态

```
ps -ef|grep redis
```

- 进行客户端校验

```
src/redis-cli -h （当前服务器的ip）-p 6379 -a redisForApexsoft
```

### 4. 安装zk

- 创建zk目录，上传zk安装程序

```
mkdir /home/ftq/zk
```
- 解压上传的程序
```
tar -zxvf zookeeper-3.4.13.tar.gz
```

- 复制配置文件
```
cd zookeeper-3.4.13/conf
```

```
cp zoo_sample.cfg zoo.cfg
```

- 创建zk数据文件夹，修改`zoo.cfg`的`dataDir`路径

```
mkdir -p /home/ftq/zk/zookeeper-3.4.13/zkdata/dev
```
```
dataDir=/home/ftq/zk/zookeeper-3.4.13/zkdata/dev
```

- 进入zk的bin目录，启动zk
```
./zkServer.sh start ../conf/zoo.cfg
```
- 查看启动状态
```
./zkServer.sh status
```

- 登陆zk。

```
./zkCli.sh -server (当前服务器的ip):2181
```

```
addauth digest amscli:apexsoft
```

- ~~微服务模块启动以后，可以查看注册进zk的微服务~~(下一步微服务运行以后再执行)。

```
ls /ams/futures
```

### 5. 安装微服务模块

- 创建ams运行目录，已有该目录则无需创建

```
mkdir /user/local/bin
```
- 上传ams文件到该目录，并赋予执行权限
```
chmod -R 777 /usr/local/bin/ams
```

- 创建微服务目录，拷贝需要的模块，如client、common-group、staff模块到该目录。

```
mkdir /home/ftq/ams
```

- 再次确保微服务config目录下的配置文件得到正确配置。

- 进入各个微服务模块，执行启动命令
```
ams start
```
- 此时可以执行zk最后一个步骤，进入zk查看微服务是否注册成功。


### 6. 安装nginx

- **根据服务器情况，如果只有一台服务器来部署则继续进行，否则切换到部署nginx的服务器上，执行一下创建用户的操作。**

- 上传 `pcre-8.38.tar.gz`、`zlib-1.2.7.tar.gz`、`nginx-1.16.1.tar.gz`到`/home/ftq/nginx`目录，依次解压。

```
tar -zxvf pcre-8.38.tar.gz
```

```
tar -zxvf zlib-1.2.7.tar.gz
```

```
tar -zxvf nginx-1.16.1.tar.gz
```
- 解压完以后，依次进入各个目录下，执行安装。(即分别进入各个目录下执行下列命令)

```
./configure
```

```
make&&make install
```

- 验证是否安装完成

验证pcre是否安装完成

```
pcre-config --version
```

验证zlib是否安装完成，执行下面命令，cat下面命令的执行结果

```
find /usr/ -name zlib.pc
```

验证nginx是否安装完成

```
/usr/local/nginx/sbin/nginx -v
```

- 修改配置文件，将`nginx.conf`文件和`ext`目录复制到nginx的`conf`目录下进行替换
- 打开`dev`目录下的`futures_dev.conf`文件，进行修改ip地址为当前服务器地址，注意配置文件下面有个这样的配置信息：
```
    location ^~ / {
        autoindex off;
        root /opt/application/static/futures_umi; #address of futures_umi
        client_max_body_size 1G;
    }
```
- `root`路径下的地址就是我们前端项目的地址，我们把我们的前端项目文件`futures_umi`复制到该目录，然后nginx启动即可访问到前端项目。

（5）复制前段项目到指定目录下。

### 7. Tomcat部署

- 创建目录，复制Tomcat文件到复制的目录，并进行解压
```
mkdir /home/ftq/tomcat
```
-原tomcat目录为`LiveBOS_Tomcat_Kirin_YDQH`，我们这边把后缀去掉，改成`LiveBOS_Tomcat_Kirin`。

（2）进入conf目录，修改server.xml文件，将数据库地址**批量**该为实际数据库地址，防止遗漏。
```
jdbc:oracle:thin:@172.30.101.56:1521:orcl
```
（3）进入`/home/ftq/tomcat/LiveBOS_Tomcat_Kirin/LiveBOS/LiveBOSCore/WEB-INF/classes`目录，修改`system.properties`文件

```
redis.port=6379
redis.host=(部署redis的服务器ip)

ams.registry.protocol=zk
ams.registry.address=(部署zk的服务器ip):2181
```
- 这边要根据实际情况配置文件路径
```
公司的路径为/data/ftq，我们部署的路径为/home/ftq，所以进行如下批量替换
/data/ftq  ->  /home/ftq

因为我们改了tomcat文件的文件夹名称，所以进行如下批量替换
/LiveBOS_Tomcat_Kirin_YDQH -> /tomcat/LiveBOS_Tomcat_Kirin
```
- 注释掉以下这行代码
```
lb.session.namespace=futures
```

### 8. 防火墙操作

开启防火墙

```
systemctl start firewalld
```

开放指定端口

```
firewall-cmd --zone=public --add-port=8011/tcp --permanent
```

```
firewall-cmd --zone=public --add-port=8001/tcp --permanent
```

重启防火墙

```
firewall-cmd --reload
```

### 9. 申请授权，安装授权证书
