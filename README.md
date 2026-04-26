# MsgMate 消息推送平台

MsgMate 是一个使用 Go 语言实现的消息推送平台，面向业务系统提供统一的消息发送、模板管理、消息记录查询和异步消费能力。项目通过 HTTP API 接收业务侧请求，将消息写入队列或数据库，再由消费端按照优先级、定时任务和限流规则完成实际推送。

项目适合作为消息中心、通知中心、营销触达平台或后端系统消息模块的基础实现，也可用于学习消息推送系统的架构设计与工程落地。

## 核心能力

- 统一消息发送接口：业务方通过 `/msg/send_msg` 提交消息发送请求。
- 消息模板管理：支持创建、查询、更新、删除消息模板。
- 消息记录查询：可根据消息 ID 查询消息发送记录。
- 多渠道推送：包含短信、邮件、飞书等推送通道的实现入口。
- 异步消费：支持 Kafka 消息队列，也支持使用 MySQL 作为消息队列的降级/替代方案。
- 定时消息：支持按指定时间发送消息。
- 优先级控制：消息可携带优先级，用于消费侧调度。
- 限流与配额：包含全局、来源、用户维度的配额和限流数据模型。
- 本地开发环境：提供 Docker Compose 一键启动 MySQL、Redis、Kafka、Kafka UI。

## 技术栈

- Go
- Gin
- MySQL
- Redis
- Kafka
- Docker Compose
- OpenAPI

## 项目结构

```text
.
├── config/                 # 本地配置文件示例
├── sql/                    # 数据库初始化脚本
├── src/
│   ├── config/             # 配置加载
│   ├── constant/           # 常量定义
│   ├── ctrl/
│   │   ├── consumer/       # 消息消费与定时消息消费
│   │   ├── handler/        # 通用请求处理
│   │   ├── msg/            # 消息与模板 HTTP 接口
│   │   ├── msgpush/        # 短信、邮件、飞书等推送通道
│   │   └── tools/          # 模板替换、限流等工具
│   ├── data/               # 数据访问层
│   ├── initialize/         # 路由注册
│   └── main.go             # 服务入口
├── wrkbench/               # 压测脚本
├── docker-compose.yml      # 本地依赖服务
├── openapi.yml             # API 文档
├── Makefile                # 常用命令
└── README.md
```

## 快速开始

### 1. 启动依赖服务

```bash
docker compose up -d
```

该命令会启动以下服务：

- MySQL：`localhost:3306`
- Redis：`localhost:6379`
- Kafka：`localhost:9092`
- Kafka UI：`http://localhost:8899`

### 2. 初始化数据库

使用 `sql/msgcenter.sql` 初始化消息中心所需的数据表。

### 3. 准备配置文件

项目运行时需要指定配置文件，例如：

```bash
go run ./src/main.go --config=./config/config-test.toml
```

配置文件中包含 MySQL、Redis、Kafka、推送渠道等配置。真实环境中的密钥、邮箱授权码、Webhook 地址等敏感信息不要提交到 Git 仓库，建议通过环境变量或本地私有配置文件管理。

### 4. 启动服务

```bash
make run
```

或直接执行：

```bash
go run ./src/main.go --config=./config/config-test.toml
```

服务默认监听配置文件中的端口，本地开发配置通常为：

```text
http://localhost:8081
```

## API 概览

接口定义见 `openapi.yml`。

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | `/msg/send_msg` | 发送消息 |
| GET | `/msg/get_msg_record` | 查询消息记录 |
| POST | `/msg/create_template` | 创建消息模板 |
| GET | `/msg/get_template` | 查询消息模板 |
| POST | `/msg/update_template` | 更新消息模板 |
| POST | `/msg/del_template` | 删除消息模板 |

## 配置说明

项目支持 Kafka 队列配置，示例结构如下：

```toml
[kafka]
brokers = ["localhost:9092"]

[kafka.topics.msg]
name = "msg"
ack = 0
async = true
offset = 0
group_id = "msg-consumer"
```

如果 Go 服务运行在宿主机，而 Kafka 运行在 Docker 容器中，通常应使用 `localhost:9092` 作为 broker 地址。更多说明见 `config/README.md`。