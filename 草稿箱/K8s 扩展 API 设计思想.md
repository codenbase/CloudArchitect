本文记录了与 Kubernetes 扩展 API 相关的问题集合

## CRD 还是 AA（API Aggregation）

1. 声明式 API 的环境解耦，摆脱了对底层 K8s 集群的强依赖。
2. 定制化存储，避免业务数据影响主集群的 Etcd 性能。
3. 深度的定制化能力。
4. 多种 API Version 兼容能力。
5. 降低私有化部署和实施的准入门槛与高昂成本。
6. 轻量级享受云原生、声明式。
7. 进可攻（云原生部署）、退可守（裸机部署）。
8. 自带 Etcd，或者用 Kine 桥接到 MySQL。
9. 重写 `RESTStorage` 的存储实现库 & KubeZoo, vCluster, kcp.io。
10. Debezium / Canal 等 CDC 工具和消息队列的事件流。
11. 数据量是否会搞死主干 K8s 集群？
12. CRD 是为了普惠大众，服务于业务场景轻、数据不敏感、追求快速开发的绝大多数业务。
13. AA 是为了构建超级平台架构，如百万车辆影子管理的 IoT 平台，剥离 K8s 基础网络，复用 K8s 控制面思想。

## 云原生 IoT 架构选型

1. **CRD：** 单 K8s 集群 + ETCD 分片 (Overrides) + 外挂 MySQL (CQRS 读写分离)。通过 `--etcd-servers-overrides` 指定一个高性能 NVMe 独立 ETCD 集群，就能完美承接高吞吐量的写入。
2. **AA（Aggregated API）：** 解决了“计算隔离”，需要开发一个独立的控制面（即 **Standalone APIServer**），不仅解决了读压力（List/Watch 导致的内存 WatchCache 暴涨、序列化导致的 CPU 飙升），也解决了写压力中的计算部分（海量高频写请求带来的 APIServer 鉴权、Mutating/Validating Webhook 校验的 CPU 开销）。
3. **独立 ETCD 集群：** 实现“存储隔离”，解决了磁盘 I/O 瓶颈和 bbolt 引擎的 8GB 容量天花板。
4. 千万级演进路线：CRD  ->  AA （Standalone APIServer）
5. 亿级演进路线：多集群联邦（集群分片）
