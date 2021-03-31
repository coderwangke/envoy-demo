# envoy-demo

使用envoy暴露集群中的gRPC服务，并支持实现负载均衡。

# 整体架构
![image](https://user-images.githubusercontent.com/42019725/113084217-269eb780-9210-11eb-9f24-19aba5130698.png)

Envoy proxy 主要工作：

- 终止 SSL/TLS 连接。
- 通过查询内部集群 DNS 服务发现运行 gRPC 服务的 pod。
- 将流量路由到 gRPC 服务 pod 并进行其负载平衡。
- 对 gRPC 服务进行运行状况检查。

# 部署步骤

## 创建Grpc服务

```
$ kubectl apply -f helloworld-grpc-server.yaml
```

## 创建envoy服务

```
$ kubectl apply -f envoy-deploy.yaml
```

补充说明：

1 envoy的静态配置有两个模块：static_resources 和 admin （用于配置管理服务器）

2 static_resources 模块包含了listener（用于配置客户端如何连接）和cluster （用于实现负载均衡）的定义

3 filter用于配置envoy如何管理流量，这里使用的是envoy内置的http_connection_manager filter

4 envoy支持给cluster配置health check，用于检查endpoint对象是否是健康的，支持HTTP, gRPC, and Redis 协议

## 测试

```
grpcurl -d '{"name": "abc"}' -plaintext -proto helloworld.proto -v <External-IP>:80 helloworld.Greeter/SayHello
```
