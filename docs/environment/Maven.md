
- 修改maven配置文件

```
<localRepository>D:\Application\Develop\Maven\MavenRepository</localRepository>

<mirror>
 <id>nexus-aliyun</id>
 <mirrorOf>central</mirrorOf>
 <name>Nexus aliyun</name>
 <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```
- 配置环境变量

MAVEN_HOME -> `D:\Application\Develop\Maven\apache-maven-3.5.4`

Path -> `%MAVEN_HOME%\bin`

- 验证

`mvn -vesion`