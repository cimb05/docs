### Node.js 环境搭建


- 配置环境变量

1. 在安装路径下创建`node_cache`、`node_global`文件夹。然后修改node默认保存路径。
2. 系统变量中添加`NODE_HOME`，值为node安装位置
3. 系统变量的`path`变量中添加`%NODE_HOME%`、`%NODE_HOME%\node_cache`、`%NODE_HOME%\node_global`

- 修改配置路径和镜像源

```
npm config set cache "D:\Software\nodejs\node_cache"
npm config set prefix "D:\Software\nodejs\node_global"
npm config set registry https://registry.npm.taobao.org
```

- 验证

```
npm config ls  
```

```
npm install -g cnpm
```
