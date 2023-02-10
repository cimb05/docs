# 虚拟机配置


## NAT网络设置
1. 查看虚拟机里面的NAT设置

```
首先我们点开虚拟机的编辑 -> 虚拟网络编辑器，切换到VMnet8，点击
NAT设置，里面有子网IP，子网掩码，网关IP
```



2. Windows设置本机的VMnet8的IPv4协议。

```
此时我们需要到安装虚拟机的windows上修改VMnet8的IPv4协议。
IP：同网关任意地址。（暂时不清楚用途）这里注意不能是网关ip，否则下面的默认网关数据就会丢失
子网掩码：NAT设置中的子网掩码
默认网关：NAT设置中的网关IP

DNS服务器选择常用的即可，如：
8.8.8.8
114.114.114.114

```



3. 配置虚拟机里的Linux系统联网

```
执行 su root		--切换到root模式
vim /etc/sysconfig/network-scripts/ifcfg-ens33			--编辑虚拟机网卡配置
	BOOTPROTO=static
	ONBOOT=yes
	IPADDR=192.168.174.199		--根据网关IP，自己选一个合适的作为IP
	NETMASK=255.255.255.0		--子网掩码
	GATEWAY=192.168.174.2		--网关IP
	DNS1=8.8.8.8				--DNS1
	DNS2=6.6.6.6				--DNS2
	
IPADDR=192.168.174.199
NETMASK=255.255.255.0
GATEWAY=192.168.174.2
DNS1=8.8.8.8
DNS2=6.6.6.6
	
service network restart	
```



4. 配置虚拟机里的Windows联网

```
更改适配器配置的IPv4协议
IP：同网关任意地址。
子网掩码：NAT设置中的子网掩码
默认网关：NAT设置中的网关IP

DNS服务器选择常用的即可，如：
8.8.8.8
114.114.114.114
```


## 安装esd格式系统的方法

 - 问题描述：
1、虚拟机无法安装esd格式的系统
2、esd转换工具只能转换官方esd文件，无法转换第三方esd文件

 - 解决方案：
1、将第三方esd转换成iso格式系统

 - 解决步骤：
1、准备第三方esd文件，准备官方iso镜像，准备UltraISO工具
2、使用UltraISO工具打开官方镜像，把esd重命名成install.wim/install.esd，替换原来的install.wim
3、保存成新的ISO镜像，解决完成。