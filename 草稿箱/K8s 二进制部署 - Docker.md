### 1. 创建工作区

```bash
# 创建主目录和三个子目录
mkdir -p ~/k8s-configs/{templates,pki,nodes}

# 在 nodes 下，为每个节点创建专属的、模仿容器内部结构的“配置包”
for node in control-plane worker-01 worker-02; do
  mkdir -p ~/k8s-configs/nodes/${node}/{kubernetes/pki,services,entrypoint.d,profile.d,containerd}
done
```

### 2.  CA 证书

```bash
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```

```bash
cat > ca-csr.json <<EOF
{
  "CN": "kubernetes-ca",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "kubernetes",
      "OU": "System"
    }
  ]
}
EOF
```

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

### 3. Admin 证书

```bash
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF
```

```bash
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

```bash
# 在本地电脑上执行
# 将 CA 私钥和 admin 证书只拷贝给 control-plane 节点
cp ~/k8s-configs/pki/{ca-key.pem,admin.pem,admin-key.pem} ~/k8s-configs/nodes/control-plane/kubernetes/pki/

# 将 CA 根证书分发给所有三个节点
for node in control-plane worker-01 worker-02; do
  echo ">>> Distributing ca.pem to ${node}..."
  cp ~/k8s-configs/pki/ca.pem ~/k8s-configs/nodes/${node}/kubernetes/pki/
done
```
