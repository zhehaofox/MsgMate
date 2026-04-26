 # 消息中心Kafka配置指南

## 配置结构说明

新版本的Kafka配置采用更灵活的结构化方式，支持通过配置文件动态添加不同的队列。

### 主要配置项

```toml
[kafka]
brokers = ["地址1:端口", "地址2:端口"]  # Kafka代理服务器地址列表

# 队列配置示例
[kafka.topics.队列名称]
name = "Topic名称"      # Kafka Topic名称
ack = 0                # 确认机制：0=不等待确认，1=等待leader确认，-1=等待所有副本确认
async = true           # 是否异步发送
offset = 0             # 消费者偏移量：0=从头开始消费，-1=从最新消息开始消费
group_id = "组ID"      # 消费者组ID，可选
```

## 本地开发配置

当您在本地机器上直接运行Go程序而Kafka在Docker容器中运行时，需要特别注意以下几点：

### 1. 配置brokers地址

由于Go程序不在Docker网络中，无法通过容器名称解析地址，因此需要使用主机的IP地址或localhost：

```toml
[kafka]
brokers = ["localhost:9092"]  # 使用本地地址和映射的端口
```

### 2. 确保端口映射正确

在docker-compose.yml中，确保Kafka容器的端口已正确映射到主机：

```yaml
kafka:
  ports:
    - "9092:9092"  # 主机端口:容器端口
```

### 3. 配置Kafka监听地址

在docker-compose.yml中，确保Kafka配置了正确的监听地址：

```yaml
kafka:
  environment:
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,OUTSIDE://localhost:9092
    KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,OUTSIDE:PLAINTEXT
    KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,OUTSIDE://0.0.0.0:9093
    KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
```


## 故障排除

如果遇到连接问题，可以尝试以下解决方案：

1. **检查端口映射**：确认docker-compose.yml中的端口映射正确
2. **使用IP地址**：使用本机实际IP地址替代localhost
3. **修改hosts文件**：在Windows的hosts文件中添加映射`127.0.0.1 msgcenter_kafka`
4. **检查防火墙**：确保防火墙未阻止Kafka端口