- 在 https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases 下载apoc扩展包JAR文件
- 在 https://github.com/neo4j-contrib/neo4j-graph-algorithms/releases 下载algo扩展包JAR文件
- 将jar包放到Neo4j安装目录下plugins文件夹中
- 在配置文件里添加

```
dbms.security.procedures.unrestricted=algo.*
dbms.security.procedures.unrestricted=apoc.*
```
- 重新启动Neo4j服务

```
neo4j stop
neo4j start
```