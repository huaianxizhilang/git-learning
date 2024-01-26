1 sql

```sql
CREATE TABLE `testTree`  (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '编号', 
    `parent_id` bigint(20) NULL COMMENT '父编号', 
    `service_name` varchar(255) NOT NULL COMMENT '服务', 
    `code_location` varchar(255) NULL COMMENT '代码仓', 
    `path` varchar(255) NULL COMMENT '路径',
    `branch` varchar(1024) NULL COMMENT '分支', 
    `Extendedone` varchar(255) NULL COMMENT '扩展字段1',
    `Extendedtwo` varchar(255) NULL COMMENT '扩展字段2',
    `Extendedthree` varchar(255) NULL COMMENT '扩展字段3',
    `Extendedfour` varchar(255) NULL COMMENT '扩展字段4',
    `Extendedfive` varchar(255) NULL COMMENT '扩展字段5',
    `Extendedsix` varchar(255) NULL COMMENT '扩展字段6',
	 PRIMARY KEY (`id`)
)
```



2 新增接口 

2.1 新增接口  填写完git地址后二次确认 clone远端代码到本地目录 根据 私钥地址、git地址 生成本地git目录

2.2 删除接口 删除本地目录

2.3 修改接口  删除本地目录，重新chone 全部

2.4 获取远程分支接口 

2.5 pull 具体远程分支到本地接口 