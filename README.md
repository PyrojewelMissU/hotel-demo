# 酒店管理与搜索系统

基于 Spring Boot + Elasticsearch + RabbitMQ 的微服务架构酒店管理系统，实现了酒店数据的 CRUD 操作和高性能搜索功能。

## 项目简介

本项目是一个演示 Elasticsearch 与关系型数据库数据同步的微服务系统，通过 RabbitMQ 实现事件驱动的数据一致性。

### 核心功能

- 🏨 **酒店管理**：完整的酒店信息增删改查
- 🔍 **智能搜索**：基于 Elasticsearch 的全文搜索、多条件过滤
- 📍 **地理位置**：支持按距离排序的地理位置搜索
- 💡 **自动补全**：酒店名称、商圈的自动补全建议
- 🎯 **广告推广**：支持广告酒店的权重提升
- 📊 **聚合统计**：动态筛选条件（品牌、城市、星级）

### 技术架构

```
┌─────────────────┐         ┌─────────────────┐
│  hotel-admin    │         │  hotel-search   │
│   (端口 8099)    │         │   (端口 8089)    │
│                 │         │                 │
│  MySQL CRUD     │         │  ES 搜索服务     │
└────────┬────────┘         └────────┬────────┘
         │                           │
         │    RabbitMQ 消息队列       │
         └───────────┬───────────────┘
                     │
         ┌───────────▼───────────┐
         │  数据同步 (异步)        │
         │  MySQL → Elasticsearch │
         └───────────────────────┘
```

## 技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Spring Boot | 2.7.17 | 核心框架 |
| Java | 11+ | 开发语言 |
| MySQL | 8.0.33 | 关系型数据库 |
| Elasticsearch | 7.17.6 | 搜索引擎 |
| RabbitMQ | 最新稳定版 | 消息队列 |
| MyBatis Plus | 3.5.4 | ORM 框架 |
| Hutool | 5.8.12 | 工具库 |
| Lombok | 1.18.30 | 简化 Java 代码 |

## 项目结构

```
hotel-es/
├── hotel-admin/              # 酒店管理服务
│   ├── src/main/java/
│   │   └── com/amazecode/hotel/
│   │       ├── web/          # REST 控制器
│   │       ├── service/      # 业务逻辑
│   │       ├── mapper/       # 数据访问
│   │       └── pojo/         # 实体类
│   └── src/main/resources/
│       ├── application.yaml  # 配置文件
│       └── static/           # 前端页面
│
├── hotel-search/             # 酒店搜索服务
│   ├── src/main/java/
│   │   └── com/amazecode/hotel/
│   │       ├── controller/   # REST 控制器
│   │       ├── service/      # ES 搜索逻辑
│   │       ├── mq/           # MQ 消息监听
│   │       ├── config/       # MQ 配置
│   │       └── pojo/         # 实体类、文档模型
│   └── src/main/resources/
│       ├── application.yaml  # 配置文件
│       └── static/           # 前端页面
│
└── pom.xml                   # 父 POM 文件
```

## 环境要求

- **JDK**: 11 或更高版本
- **Maven**: 3.6+
- **MySQL**: 8.0+
- **Elasticsearch**: 7.17.6
- **RabbitMQ**: 3.x
- **操作系统**: Windows / Linux / macOS

## 中间件部署

### 方案一：Docker 部署（推荐）

#### 1. 部署 MySQL

```bash
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e MYSQL_DATABASE=itheima \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0.33
```

#### 2. 部署 Elasticsearch

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -v es-data:/usr/share/elasticsearch/data \
  elasticsearch:7.17.6
```

验证安装：
```bash
curl http://localhost:9200
```

#### 3. 部署 Kibana（可选，用于可视化管理 ES）

```bash
docker run -d \
  --name kibana \
  -p 5601:5601 \
  -e "ELASTICSEARCH_HOSTS=http://192.168.100.129:9200" \
  kibana:7.17.6
```

访问：`http://localhost:5601`

#### 4. 部署 RabbitMQ

```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3-management
```

访问管理界面：`http://localhost:15672`
- 用户名：`admin`
- 密码：`admin123`

### 方案二：Linux 虚拟机部署

如果你使用 Linux 虚拟机（如本项目配置的 `192.168.100.129`），请确保：

1. **安装 Docker 和 Docker Compose**：
```bash
# 安装 Docker
curl -fsSL https://get.docker.com | bash -s docker

# 启动 Docker
sudo systemctl start docker
sudo systemctl enable docker
```

2. **创建 docker-compose.yml**：

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0.33
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: itheima
    volumes:
      - mysql-data:/var/lib/mysql

  elasticsearch:
    image: elasticsearch:7.17.6
    container_name: elasticsearch
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    volumes:
      - es-data:/usr/share/elasticsearch/data

  kibana:
    image: kibana:7.17.6
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123

volumes:
  mysql-data:
  es-data:
```

3. **启动所有服务**：
```bash
docker-compose up -d
```

4. **检查服务状态**：
```bash
docker-compose ps
```

### 方案三：手动安装

请参考各中间件官方文档进行安装：
- [MySQL 安装文档](https://dev.mysql.com/doc/)
- [Elasticsearch 安装文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/install-elasticsearch.html)
- [RabbitMQ 安装文档](https://www.rabbitmq.com/download.html)

## 数据库初始化

### 1. 创建数据库

```sql
CREATE DATABASE IF NOT EXISTS itheima CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE itheima;
```

### 2. 创建酒店表

```sql
CREATE TABLE `tb_hotel` (
  `id` bigint NOT NULL COMMENT '酒店id',
  `name` varchar(255) NOT NULL COMMENT '酒店名称',
  `address` varchar(255) NOT NULL COMMENT '酒店地址',
  `price` int NOT NULL COMMENT '酒店价格',
  `score` int NOT NULL COMMENT '酒店评分',
  `brand` varchar(32) NOT NULL COMMENT '酒店品牌',
  `city` varchar(32) NOT NULL COMMENT '所在城市',
  `star_name` varchar(16) DEFAULT NULL COMMENT '酒店星级',
  `business` varchar(255) DEFAULT NULL COMMENT '商圈',
  `latitude` varchar(32) NOT NULL COMMENT '纬度',
  `longitude` varchar(32) NOT NULL COMMENT '经度',
  `pic` varchar(255) DEFAULT NULL COMMENT '酒店图片',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='酒店表';
```

### 3. 导入测试数据（可选）

如果你有测试数据的 SQL 文件，执行：
```bash
mysql -u root -p itheima < hotel_data.sql
```

## Elasticsearch 索引初始化

### 1. 创建 hotel 索引

使用 Kibana 的 Dev Tools 或 curl 命令：

```json
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": { "type": "long" },
      "name": {
        "type": "text",
        "analyzer": "ik_max_word",
        "copy_to": "all"
      },
      "address": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "price": { "type": "integer" },
      "score": { "type": "integer" },
      "brand": {
        "type": "keyword",
        "copy_to": "all"
      },
      "city": { "type": "keyword" },
      "starName": { "type": "keyword" },
      "business": {
        "type": "keyword",
        "copy_to": "all"
      },
      "location": { "type": "geo_point" },
      "pic": { "type": "keyword", "index": false },
      "all": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "suggestion": {
        "type": "completion",
        "analyzer": "ik_max_word"
      },
      "isAD": { "type": "boolean" }
    }
  }
}
```

### 2. 或运行项目中的测试类

项目中已包含索引创建的测试类，启动 `hotel-search` 服务后运行：
```bash
mvn test -Dtest=HotelIndexTest#createHotelIndex
```

## 配置文件修改

### hotel-admin 配置

编辑 `hotel-admin/src/main/resources/application.yaml`：

```yaml
server:
  port: 8099

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/itheima?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8
    username: root
    password: 123456  # 修改为你的 MySQL 密码
    driver-class-name: com.mysql.cj.jdbc.Driver

  rabbitmq:
    host: 192.168.100.129  # 修改为你的 RabbitMQ 地址
    port: 5672
    username: admin
    password: admin123
    virtual-host: /

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
  type-aliases-package: com.amazecode.hotel.pojo
```

### hotel-search 配置

编辑 `hotel-search/src/main/resources/application.yaml`：

```yaml
server:
  port: 8089

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/itheima?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8
    username: root
    password: 123456  # 修改为你的 MySQL 密码
    driver-class-name: com.mysql.cj.jdbc.Driver

  rabbitmq:
    host: 192.168.100.129  # 修改为你的 RabbitMQ 地址
    port: 5672
    username: admin
    password: admin123
    virtual-host: /

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
  type-aliases-package: com.amazecode.hotel.pojo
```

同时修改 `HotelSearchApplication.java` 中的 Elasticsearch 连接地址：

```java
@Bean
public RestHighLevelClient client() {
    return new RestHighLevelClient(RestClient.builder(
            HttpHost.create("http://192.168.100.129:9200")  // 修改为你的 ES 地址
    ));
}
```

## 项目启动

### 1. 编译项目

在项目根目录执行：

```bash
mvn clean install
```

### 2. 启动 hotel-admin 服务

**方式一：使用 Maven**
```bash
cd hotel-admin
mvn spring-boot:run
```

**方式二：直接运行主类**
```bash
# IDEA 中直接运行 HotelAdminApplication.java
```

**方式三：运行 JAR 包**
```bash
cd hotel-admin/target
java -jar hotel-admin-1.0.0.jar
```

启动成功后，访问：
- 管理界面：http://localhost:8099
- API 文档：http://localhost:8099/hotel/list

### 3. 启动 hotel-search 服务

**方式一：使用 Maven**
```bash
cd hotel-search
mvn spring-boot:run
```

**方式二：直接运行主类**
```bash
# IDEA 中直接运行 HotelSearchApplication.java
```

**方式三：运行 JAR 包**
```bash
cd hotel-search/target
java -jar hotel-search-1.0.0.jar
```

启动成功后，访问：
- 搜索界面：http://localhost:8089
- API 文档：http://localhost:8089/hotel/list

### 4. 验证服务

#### 验证 hotel-admin
```bash
curl http://localhost:8099/hotel/list?page=1&size=5
```

#### 验证 hotel-search
```bash
curl -X POST http://localhost:8089/hotel/list \
  -H "Content-Type: application/json" \
  -d '{"key":"", "page":1, "size":5}'
```

## 功能测试

### 1. 酒店管理（hotel-admin）

**查询酒店列表**
- 访问：http://localhost:8099
- 支持分页查询

**新增酒店**
```bash
curl -X POST http://localhost:8099/hotel \
  -H "Content-Type: application/json" \
  -d '{
    "name": "测试酒店",
    "address": "北京市朝阳区",
    "price": 500,
    "score": 45,
    "brand": "如家",
    "city": "北京",
    "starName": "三星级",
    "business": "国贸",
    "latitude": "39.904989",
    "longitude": "116.405285",
    "pic": "https://example.com/pic.jpg"
  }'
```

**修改酒店**
```bash
curl -X PUT http://localhost:8099/hotel \
  -H "Content-Type: application/json" \
  -d '{
    "id": 60223,
    "name": "上海希尔顿酒店",
    "price": 3097,
    ...
  }'
```

**删除酒店**
```bash
curl -X DELETE http://localhost:8099/hotel/60223
```

### 2. 酒店搜索（hotel-search）

**全文搜索**
```bash
curl -X POST http://localhost:8089/hotel/list \
  -H "Content-Type: application/json" \
  -d '{
    "key": "希尔顿",
    "page": 1,
    "size": 10
  }'
```

**条件过滤**
```bash
curl -X POST http://localhost:8089/hotel/list \
  -H "Content-Type: application/json" \
  -d '{
    "key": "",
    "city": "上海",
    "brand": "希尔顿",
    "starName": "五星级",
    "minPrice": 1000,
    "maxPrice": 5000,
    "page": 1,
    "size": 10
  }'
```

**地理位置搜索**
```bash
curl -X POST http://localhost:8089/hotel/list \
  -H "Content-Type: application/json" \
  -d '{
    "location": "31.21,121.5",
    "page": 1,
    "size": 10
  }'
```

**自动补全**
```bash
curl http://localhost:8089/hotel/suggestion?key=希
```

**获取筛选条件**
```bash
curl -X POST http://localhost:8089/hotel/filters \
  -H "Content-Type: application/json" \
  -d '{"key": ""}'
```

### 3. 数据同步测试

1. 在 `localhost:8099` 修改一个酒店的信息
2. 观察控制台日志，应该看到：
   - hotel-admin 发送 MQ 消息
   - hotel-search 接收 MQ 消息
   - hotel-search 同步数据到 ES
3. 在 `localhost:8089` 搜索该酒店，确认数据已更新

## 常见问题

### 1. 连接超时

**问题**：无法连接到 MySQL / Elasticsearch / RabbitMQ

**解决**：
- 检查中间件服务是否启动：`docker ps`
- 检查防火墙是否开放端口
- Linux 虚拟机需要检查网络配置：`ping 192.168.100.129`

### 2. RabbitMQ 消息未消费

**问题**：修改酒店数据后，ES 中数据未同步

**解决**：
- 检查 RabbitMQ 管理界面：http://192.168.100.129:15672
- 确认队列 `hotel.insert.queue` 和 `hotel.delete.queue` 已创建
- 确认交换机 `hotel.topic` 已绑定队列
- 查看 hotel-search 服务日志是否有异常

### 3. Elasticsearch 查询失败

**问题**：搜索时报错 `No mapping found for [default]`

**解决**：
- 前端不要传递 `sortBy: "default"` 参数
- 或者在后端过滤无效的排序字段

### 4. 中文乱码

**问题**：搜索中文时无结果或乱码

**解决**：
- 确认 Elasticsearch 安装了 IK 分词器插件
- 安装命令：
  ```bash
  docker exec -it elasticsearch bash
  ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.17.6/elasticsearch-analysis-ik-7.17.6.zip
  exit
  docker restart elasticsearch
  ```

### 5. 内存不足

**问题**：Elasticsearch 启动失败或服务器卡顿

**解决**：
- 调整 ES 的 JVM 参数：`-e "ES_JAVA_OPTS=-Xms256m -Xmx256m"`
- 增加虚拟机内存配置

## 监控与管理

### RabbitMQ 管理界面

访问：http://192.168.100.129:15672
- 查看队列消息数量
- 监控消息发送和消费速率
- 管理交换机和队列绑定

### Kibana 可视化

访问：http://192.168.100.129:5601
- Dev Tools：执行 ES 查询和管理操作
- Discover：查看索引数据
- Monitoring：监控 ES 集群状态

### 常用 Elasticsearch 命令

```bash
# 查看所有索引
curl http://192.168.100.129:9200/_cat/indices?v

# 查看 hotel 索引的 mapping
curl http://192.168.100.129:9200/hotel/_mapping

# 查看某个文档
curl http://192.168.100.129:9200/hotel/_doc/60223

# 搜索全部文档
curl http://192.168.100.129:9200/hotel/_search

# 删除索引
curl -X DELETE http://192.168.100.129:9200/hotel
```

## 开发建议

1. **日志级别**：开发环境可设置为 `debug`，生产环境改为 `info`
2. **连接池**：生产环境需要配置数据库和 ES 连接池参数
3. **异常处理**：建议添加全局异常处理器
4. **性能优化**：
   - ES 查询使用缓存
   - MySQL 添加索引
   - RabbitMQ 消息持久化
5. **安全性**：
   - 修改中间件默认密码
   - 配置文件敏感信息使用环境变量
   - 开启 HTTPS

## 项目扩展

- [ ] 添加 Redis 缓存层
- [ ] 实现分布式事务
- [ ] 接入 Nacos 配置中心
- [ ] 添加 Spring Cloud Gateway 网关
- [ ] 集成 Sentinel 限流降级
- [ ] 添加分布式链路追踪（SkyWalking）

## 许可证

MIT License

## 联系方式

如有问题，请提交 Issue 或联系项目维护者。

---

**祝你使用愉快！** 🎉
