# Oracle安装

### 1.	安装相关环境软件包

```
yum install -y binutils* compat-libcap1* compat-libstdc++* gcc* glibc* ksh* libgcc* libstdc++* libaio* make* sysstat* elfutils-libelf-devel* unixODBC*

```

### 2.	修改内核参数

- 修改

```
vim /etc/sysctl.conf
```

- 添加以下配置

```
io-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 1073741824
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

- 使内核改变立即生效

```
/sbin/sysctl -p
```

### 3.	修改用户的限制文件

- 修改配置

```
vim /etc/security/limits.conf
```

- 添加以下配置

```
oracle           soft    nproc           2047
oracle           hard    nproc           16384
oracle           soft    nofile          1024
oracle           hard    nofile          65536
oracle           soft    stack           10240
```

### 4.	修改login文件

- 修改

```
vim /etc/pam.d/login
```

- 添加以下配置

```
session  required   /lib64/security/pam_limits.so
session  required   pam_limits.so
```


### 5.	修改profile文件

- 修改

```
vim /etc/profile
```

- 添加如下配置

```
if [ $USER = "oracle" ]; then
  if [ $SHELL = "/bin/ksh" ]; then
   ulimit -p 16384
   ulimit -n 65536
  else
   ulimit -u 16384 -n 65536
  fi
fi

```

### 6.	用户、权限

- 添加组和用户

```
groupadd dba
groupadd oinstall
useradd -d /home/oracle -m -g oinstall -G dba -p 123 oracle
```

- 添加相关目录
```
mkdir -p /u01/app/oracle/product/11.2.0
mkdir /u01/app/oracle/oradata
mkdir /u01/app/oracle/inventory
mkdir /u01/app/oracle/fast_recovery_area
```

- 添加权限

```
chown -R oracle:oinstall /u01/app/oracle
chmod -R 775 /u01/app/oracle
```


### 7.	上传oracle软件包到 /tmp目录下，解压

```
unzip linux.x64_11gR2_database_1of2.zip && unzip linux.x64_11gR2_database_2of2.zip
```

### 8.	设置oracle用户环境变量

- 切换到Oracle用户

```
su - oracle 
```

- 修改

```
vim .bash_profile
```

- 添加如下配置

```
ORACLE_BASE=/u01/app/oracle
ORACLE_HOME=$ORACLE_BASE/product/11.2.0
ORACLE_SID=orcl
PATH=$PATH:$ORACLE_HOME/bin
export ORACLE_BASE ORACLE_HOME ORACLE_SID PATH
```

- 使修改立即生效

```
source .bash_profile
```


### 9.	备份db_install.rsp

```
su root
```
```
cd /tmp/database/response/
```
```
cp db_install.rsp db_install.rsp.bak
```



### 10.	编辑静默安装响应文件

- 编辑
```
vim /tmp/database/response/db_install.rsp
```

- 配置如下

```
oracle.install.option=INSTALL_DB_SWONLY

ORACLE_HOSTNAME=woitumi-197

UNIX_GROUP_NAME=oinstall

INVENTORY_LOCATION=/u01/app/oracle/inventory

SELECTED_LANGUAGES=en,zh_CN

ORACLE_HOME=/u01/app/oracle/product/11.2.0

ORACLE_BASE=/u01/app/oracle

oracle.install.db.InstallEdition=EE

oracle.install.db.DBA_GROUP=dba

oracle.install.db.OPER_GROUP=dba

DECLINE_SECURITY_UPDATES=true    //385
```

### 11.	Oracle安装


```
su - oracle
```
```
cd /tmp/database
```
```
./runInstaller -silent -ignorePrereq -ignoreSysPrereqs -responseFile /tmp/database/response/db_install.rsp
```
(等待......)
```
su root
```
```
sh /u01/app/oracle/inventory/orainstRoot.sh
```
```
sh /u01/app/oracle/product/11.2.0/root.sh
```


### 12.	配置监听

```
su - oracle
```
```
netca -silent -responseFile /tmp/database/response/netca.rsp
```
如果报错，执行:
```
export DISPLAY=localhost:0.0
```

--检查是否已经启动
```
netstat -tnulp | grep 1521

```



### 13.	修改配置文件

```
su - root
```
```
vim /tmp/database/response/dbca.rsp
```
```
GDBNAME = "orcl"
SID = "orcl"
SYSPASSWORD = "oracle"
SYSTEMPASSWORD = "oracle"
SYSMANPASSWORD = "oracle"
DBSNMPPASSWORD = "oracle"
DATAFILEDESTINATION =/u01/app/oracle/oradata        /357
RECOVERYAREADESTINATION=/u01/app/oracle/fast_recovery_area        /367
CHARACTERSET = "ZHS16GBK"        /415
TOTALMEMORY = "1638"        /540
```

//重载配置文件
```
su - oracle 
```
```
dbca -silent -responseFile /tmp/database/response/dbca.rsp
```



### 14.	检验安装结果

```
ps -ef | grep ora_ | grep -v grep
```
```
lsnrctl status
```
```
sqlplus / as sysdba
```





### 15.	修改文件主机地址

```
cd /u01/app/oracle/product/11.2.0/network/admin
```
修改listener.ora和tnsnames.ora中的主机地址，改成本机内网ip，不知道内网ip可以查询`/etc/hosts`文件

```
vim listener.ora
```
```
SID_LIST_LISTENER =  
(SID_LIST =  
  (SID_DESC =  
  (GLOBAL_DBNAME = orcl)
  (SID_NAME = orcl)
  )
)
```



### 16.	需要重启服务器需要进行的操作

```
su - oracle
```
```
cd $ORACLE_HOME/bin
```
```
lsnrctl start
```
```
sqlplus / as sysdba
```
```
startup
```



### 17.	导入dmp



```
1.	查询系统路径，上传dump文件到DATA_PUMP_DIR路径
select * from dba_directories;

2.	创建表空间
CREATE tablespace LIVEBOS datafile  '/u01/app/oracle/oradata/orcl11g/LIVEBOS.dbf' SIZE 100M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;

3.	创建用户并授权
-- Create the user
create user LIVEBOS
  identified by "abs"
  default tablespace LIVEBOS
  --temporary tablespace LIVEBOS_TMP
  profile DEFAULT
  password expire
  quota unlimited on livebos
  quota 5m on system;
-- Grant/Revoke role privileges
grant connect to LIVEBOS;
-- Grant/Revoke system privileges
grant alter any procedure to LIVEBOS;
grant alter any sequence to LIVEBOS;
grant alter any table to LIVEBOS;
grant analyze any to LIVEBOS;
grant comment any table to LIVEBOS;
grant create any job to LIVEBOS;
grant create any procedure to LIVEBOS;
grant create any sequence to LIVEBOS;
grant create any type to LIVEBOS;
grant create table to LIVEBOS;
grant create view to LIVEBOS;
grant debug connect session to LIVEBOS;
grant delete any table to LIVEBOS;
grant drop any procedure to LIVEBOS;
grant drop any sequence to LIVEBOS;
grant execute any procedure to LIVEBOS;
grant insert any table to LIVEBOS;
grant select any sequence to LIVEBOS;
grant select any table to LIVEBOS;
grant unlimited tablespace to LIVEBOS;
grant update any table to LIVEBOS;

4.	执行dmp命令
impdp livebos/abs directory=DATA_PUMP_DIR dumpfile=zhqh_livebos_2021-07-30.dmp full=y  table_exists_action=replace log=zhqh_livebos_2021-07-30.log

输入用户名和密码
username:	sys as sysdba
password:	oracle(配置的实际密码)

```