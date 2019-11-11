# docker相关操作
## docker registry
```
mkdir /opt/registry-var/auth/ -p
yum install httpd-tools -y
htpasswd  -Bbn oldboy 123456  >> /opt/registry-var/auth/htpasswd

docker run -d -p 5000:5000 --restart=always --name registry -v /opt/registry-var/auth/:/auth/ -v /opt/myregistry:/var/lib/registry -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd registry

```



vim /etc/docker/daemon.json
```
{
  "registry-mirrors": ["https://gqk8w9va.mirror.aliyuncs.com"],
  "graph":"/opt/mydocker",
  "insecure-registries":["192.168.0.11:5000"]
}
```


```
{
  "bip": "172.17.0.1/16",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://fz5yth0r.mirror.aliyuncs.com"],
  "insecure-registries": ["hub.docker.fenqi.d.xiaonei.com","registry.fenqi.d.xiaonei.com"],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

准备镜像
192.168.0.11:5000/wuxingge/pause-amd64:3.0




## docker hub
[https://hub.docker.com](https://hub.docker.com)

```
docker tag 192.168.0.11:5000/wuxingge/kubernetes-dashboard:v1.10.1 wuxingge/mydocker:kubernetes-dashboard-v1.10.1
```


```
docker login 
docker push wuxingge/mydocker:kubernetes-dashboard-v1.10.1
```

```
docker pull wuxingge/mydocker:kubernetes-dashboard-v1.10.1
```


## aliyun镜像仓库
```
docker tag registry.cn-beijing.aliyuncs.com/minminmsn/kubernetes-dashboard:v1.10.1 registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard:v1.10.1

docker login --username=dong1226032602 registry.cn-hangzhou.aliyuncs.com

docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard:v1.10.1 
```






# 二进制包下载
Client Binaries
https://dl.k8s.io/v1.13.1/kubernetes-client-linux-amd64.tar.gz

Server Binaries
https://dl.k8s.io/v1.13.1/kubernetes-server-linux-amd64.tar.gz

Node Binaries
https://dl.k8s.io/v1.13.1/kubernetes-node-linux-amd64.tar.gz

etcd
https://github.com/etcd-io/etcd/releases/download/v3.3.10/etcd-v3.3.10-linux-amd64.tar.gz

# flannel
https://github.com/coreos/flannel/releases/download/v0.10.0/flannel-v0.10.0-linux-amd64.tar.gz


下载镜像
```
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.12.2
docker pull mirrorgooglecontainers/kube-controller-amd64:v1.12.2
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.12.2
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.13.1
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.14.2
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.12.2
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.12.2
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd-amd64:3.2.24
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.5.1
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.4.1
docker pull mirrorgooglecontainers/coredns:1.2.2
docker pull mirrorgooglecontainers/coredns-amd64:1.2.2
docker pull coredns/coredns:1.2.2
```



# 参考文档
[https://github.com/minminmsn/k8s1.13](https://github.com/minminmsn/k8s1.13)
[https://v1-13.docs.kubernetes.io/zh/docs/](https://v1-13.docs.kubernetes.io/zh/docs/)




---

**二进制部署自此开始**
# 
角色划分
k8s-master1          192.168.0.11          k8s-master          etcd、kube-apiserver、kube-controller-manager、kube-scheduler
k8s-node1            192.168.0.12          k8s-node            etcd、kubelet、docker、kube_proxy
k8s-node2            192.168.0.13          k8s-node            etcd、kubelet、docker、kube_proxy

hosts
```
192.168.0.11  k8s-master1
192.168.0.12  k8s-node1
192.168.0.13  k8s-node2
```
# 




Master部署
# 4.1 下载软件
```
wget https://dl.k8s.io/v1.13.1/kubernetes-server-linux-amd64.tar.gz
wget https://dl.k8s.io/v1.13.1/kubernetes-client-linux-amd64.tar.gz
wget https://github.com/etcd-io/etcd/releases/download/v3.3.10/etcd-v3.3.10-linux-amd64.tar.gz
wget https://github.com/coreos/flannel/releases/download/v0.10.0/flannel-v0.10.0-linux-amd64.tar.gz
```
# 


4.2 cfssl安装
```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64

chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64

cp cfssl_linux-amd64 /usr/local/bin/cfssl
cp cfssljson_linux-amd64 /usr/local/bin/cfssljson
cp cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```
# 




4.3 创建etcd证书
```
mkdir /k8s/etcd/{bin,cfg,ssl} -p
mkdir /k8s/kubernetes/{bin,cfg,ssl} -p
cd /k8s/etcd/ssl/
```

## 1）etcd ca配置
```

cat << EOF | tee ca-config.json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "etcd": {
         "expiry": "876000h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF
```
## 
2）etcd ca证书
```

cat << EOF | tee ca-csr.json
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing"
        }
    ]
}
EOF
```
## 
3）etcd server证书
```

cat << EOF | tee server-csr.json
{
    "CN": "etcd",
    "hosts": [
    "192.168.0.11",
    "192.168.0.12",
    "192.168.0.13"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing"
        }
    ]
}
EOF
```
## 
4）生成etcd ca证书和私钥 初始化ca
```

[root@elasticsearch01 ssl]# ls
ca-config.json  ca-csr.json  server-csr.json

[root@elasticsearch01 ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
2018/12/26 16:13:54 [INFO] generating a new CA key and certificate from CSR
2018/12/26 16:13:54 [INFO] generate received request
2018/12/26 16:13:54 [INFO] received CSR
2018/12/26 16:13:54 [INFO] generating key: rsa-2048
2018/12/26 16:13:54 [INFO] encoded CSR
2018/12/26 16:13:54 [INFO] signed certificate with serial number 144752911121073185391033754516204538929473929443

[root@elasticsearch01 ssl]# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  server-csr.json

生成server证书

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd server-csr.json | cfssljson -bare server

[root@elasticsearch01 ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd server-csr.json | cfssljson -bare server
2018/12/26 16:18:53 [INFO] generate received request
2018/12/26 16:18:53 [INFO] received CSR
2018/12/26 16:18:53 [INFO] generating key: rsa-2048
2018/12/26 16:18:54 [INFO] encoded CSR
2018/12/26 16:18:54 [INFO] signed certificate with serial number 388122587040599986639159163167557684970159030057
2018/12/26 16:18:54 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for websites. 
For more information see the Baseline Requirements for the Issuance and Management of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

[root@elasticsearch01 ssl]# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  server.csr  server-csr.json  server-key.pem  server.pem
```
# 


4.4 etcd安装(所有节点)
## 
1）解压缩
```

tar xf etcd-v3.3.10-linux-amd64.tar.gz
cd etcd-v3.3.10-linux-amd64/
cp etcd etcdctl /k8s/etcd/bin/
```
## 

2）配置etcd主文件
```
**etcd01:**
vim /k8s/etcd/cfg/etcd.conf
#[Member]
ETCD_NAME="etcd01"
ETCD_DATA_DIR="/data1/etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.0.11:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.0.11:2379"
 
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.0.11:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.0.11:2379"
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.0.11:2380,etcd02=https://192.168.0.12:2380,etcd03=https://192.168.0.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"

#[Security]
ETCD_CERT_FILE="/k8s/etcd/ssl/server.pem"
ETCD_KEY_FILE="/k8s/etcd/ssl/server-key.pem"
ETCD_TRUSTED_CA_FILE="/k8s/etcd/ssl/ca.pem"
ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/k8s/etcd/ssl/server.pem"
ETCD_PEER_KEY_FILE="/k8s/etcd/ssl/server-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/k8s/etcd/ssl/ca.pem"
ETCD_PEER_CLIENT_CERT_AUTH="true"


**etcd02:**
vim /k8s/etcd/cfg/etcd.conf
#[Member]
ETCD_NAME="etcd02"
ETCD_DATA_DIR="/data1/etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.0.12:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.0.12:2379"
 
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.0.12:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.0.12:2379"
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.0.11:2380,etcd02=https://192.168.0.12:2380,etcd03=https://192.168.0.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"

#[Security]
ETCD_CERT_FILE="/k8s/etcd/ssl/server.pem"
ETCD_KEY_FILE="/k8s/etcd/ssl/server-key.pem"
ETCD_TRUSTED_CA_FILE="/k8s/etcd/ssl/ca.pem"
ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/k8s/etcd/ssl/server.pem"
ETCD_PEER_KEY_FILE="/k8s/etcd/ssl/server-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/k8s/etcd/ssl/ca.pem"
ETCD_PEER_CLIENT_CERT_AUTH="true"



**etcd03:**
vim /k8s/etcd/cfg/etcd.conf 
#[Member]
ETCD_NAME="etcd03"
ETCD_DATA_DIR="/data1/etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.0.13:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.0.13:2379"
 
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.0.13:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.0.13:2379"
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.0.11:2380,etcd02=https://192.168.0.12:2380,etcd03=https://192.168.0.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"

#[Security]
ETCD_CERT_FILE="/k8s/etcd/ssl/server.pem"
ETCD_KEY_FILE="/k8s/etcd/ssl/server-key.pem"
ETCD_TRUSTED_CA_FILE="/k8s/etcd/ssl/ca.pem"
ETCD_CLIENT_CERT_AUTH="true"
ETCD_PEER_CERT_FILE="/k8s/etcd/ssl/server.pem"
ETCD_PEER_KEY_FILE="/k8s/etcd/ssl/server-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/k8s/etcd/ssl/ca.pem"
ETCD_PEER_CLIENT_CERT_AUTH="true"
```
## 



3）配置etcd启动文件
```

mkdir /data1/etcd -p

vim /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/data1/etcd/
EnvironmentFile=-/k8s/etcd/cfg/etcd.conf
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /k8s/etcd/bin/etcd --name=\"${ETCD_NAME}\" --data-dir=\"${ETCD_DATA_DIR}\" --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\" --listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" --advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" --initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" --initial-cluster=\"${ETCD_INITIAL_CLUSTER}\" --initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\" --cert-file=\"${ETCD_CERT_FILE}\" --key-file=\"${ETCD_KEY_FILE}\" --trusted-ca-file=\"${ETCD_TRUSTED_CA_FILE}\" --client-cert-auth=\"${ETCD_CLIENT_CERT_AUTH}\" --peer-cert-file=\"${ETCD_PEER_CERT_FILE}\" --peer-key-file=\"${ETCD_PEER_KEY_FILE}\" --peer-trusted-ca-file=\"${ETCD_PEER_TRUSTED_CA_FILE}\" --peer-client-cert-auth=\"${ETCD_PEER_CLIENT_CERT_AUTH}\""
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
## 

4）启动 注意启动前etcd02、etcd03同样配置下
```
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
```
## 


5）服务检查
```

/k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://192.168.0.11:2379,https://192.168.0.12:2379,https://192.168.0.13:2379" cluster-health
```
# 
member c21df2258ce015e6 is healthy: got healthy result from https://192.168.0.13:2379
member d427109ed3caf9c3 is healthy: got healthy result from https://192.168.0.11:2379
member ec8c40660d3c1192 is healthy: got healthy result from https://192.168.0.12:2379
cluster is healthy




4.5 生成kubernets证书与私钥
## 1）制作kubernetes ca证书
```

cd /k8s/kubernetes/ssl
```

```
cat << EOF | tee ca-config.json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "kubernetes": {
         "expiry": "876000h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF

cat << EOF | tee ca-csr.json
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```


```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -

[root@elasticsearch01 ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
2018/12/27 09:47:08 [INFO] generating a new CA key and certificate from CSR
2018/12/27 09:47:08 [INFO] generate received request
2018/12/27 09:47:08 [INFO] received CSR
2018/12/27 09:47:08 [INFO] generating key: rsa-2048
2018/12/27 09:47:08 [INFO] encoded CSR
2018/12/27 09:47:08 [INFO] signed certificate with serial number 156611735285008649323551446985295933852737436614

[root@elasticsearch01 ssl]# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```
## 



2）制作apiserver证书
```

vim server-csr.json
{
    "CN": "kubernetes",
    "hosts": [
      "10.254.0.1",
      "127.0.0.1",
      "192.168.0.11",
            "192.168.0.12",
            "192.168.0.13",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}



[root@elasticsearch01 ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
2018/12/27 09:51:56 [INFO] generate received request
2018/12/27 09:51:56 [INFO] received CSR
2018/12/27 09:51:56 [INFO] generating key: rsa-2048
2018/12/27 09:51:56 [INFO] encoded CSR
2018/12/27 09:51:56 [INFO] signed certificate with serial number 399376216731194654868387199081648887334508501005
2018/12/27 09:51:56 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

[root@elasticsearch01 ssl]# ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  server.csr  server-csr.json  server-key.pem  server.pem
```
## 



3）制作kube-proxy证书
```

cat << EOF | tee kube-proxy-csr.json
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Beijing",
      "ST": "Beijing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```



```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy

[root@elasticsearch01 ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
2018/12/27 09:52:40 [INFO] generate received request
2018/12/27 09:52:40 [INFO] received CSR
2018/12/27 09:52:40 [INFO] generating key: rsa-2048
2018/12/27 09:52:40 [INFO] encoded CSR
2018/12/27 09:52:40 [INFO] signed certificate with serial number 633932731787505365511506755558794469389165123417
2018/12/27 09:52:40 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

[root@elasticsearch01 ssl]# ls
ca-config.json  ca-csr.json  ca.pem          kube-proxy-csr.json  kube-proxy.pem  server-csr.json  server.pem
ca.csr          ca-key.pem   kube-proxy.csr  kube-proxy-key.pem   server.csr      server-key.pem
```
# 



4.6部署kubernetes server

## kubernetes master 节点运行如下组件： kube-apiserver kube-scheduler kube-controller-manager kube-scheduler 和 kube-controller-manager 可以以集群模式运行，通过 leader 选举产生一个工作进程，其它进程处于阻塞模式，master三节点高可用模式下可用

1）解压缩文件
```
tar xf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin/
cp kube-scheduler kube-apiserver kube-controller-manager kubectl /k8s/kubernetes/bin/
```
## 

2）部署kube-apiserver组件 创建TLS Bootstrapping Token
```
[root@elasticsearch01 bin]# head -c 16 /dev/urandom | od -An -t x | tr -d ' '
57cf77e5364ec551d908c983e2c05878


vim /k8s/kubernetes/cfg/token.csv
57cf77e5364ec551d908c983e2c05878,kubelet-bootstrap,10001,"system:kubelet-bootstrap"
```
## 



创建Apiserver配置文件
[https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/](https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

```

vim /k8s/kubernetes/cfg/kube-apiserver
KUBE_APISERVER_OPTS="--logtostderr=true \
--v=4 \
--etcd-servers=https://192.168.0.11:2379,https://192.168.0.12:2379,https://192.168.0.13:2379 \
--bind-address=192.168.0.11 \
--secure-port=6443 \
--advertise-address=192.168.0.11 \
--allow-privileged=true \
--service-cluster-ip-range=10.254.0.0/16 \
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \
--authorization-mode=RBAC,Node \
--enable-bootstrap-token-auth \
--token-auth-file=/k8s/kubernetes/cfg/token.csv \
--service-node-port-range=30000-50000 \
--tls-cert-file=/k8s/kubernetes/ssl/server.pem  \
--tls-private-key-file=/k8s/kubernetes/ssl/server-key.pem \
--client-ca-file=/k8s/kubernetes/ssl/ca.pem \
--service-account-key-file=/k8s/kubernetes/ssl/ca-key.pem \
--etcd-cafile=/k8s/etcd/ssl/ca.pem \
--etcd-certfile=/k8s/etcd/ssl/server.pem \
--etcd-keyfile=/k8s/etcd/ssl/server-key.pem"
```
## 




创建apiserver systemd文件
```

vim /usr/lib/systemd/system/kube-apiserver.service 

[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
 
[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-apiserver
ExecStart=/k8s/kubernetes/bin/kube-apiserver $KUBE_APISERVER_OPTS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
```
## 



启动服务
```
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver

[root@elasticsearch01 bin]# systemctl status kube-apiserver
● kube-apiserver.service - Kubernetes API Server
   Loaded: loaded (/usr/lib/systemd/system/kube-apiserver.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-12-27 14:41:22 CST; 20s ago
     Docs: https://github.com/kubernetes/kubernetes
 Main PID: 22060 (kube-apiserver)
   CGroup: /system.slice/kube-apiserver.service
           └─22060 /k8s/kubernetes/bin/kube-apiserver --logtostderr=true --v=4 --etcd-servers=https://192.168.0.11:2379,https://10.2....
```

           
[root@elasticsearch01 bin]# ps -ef |grep kube-apiserver
root     22060     1  5 14:41 ?        00:00:14 /k8s/kubernetes/bin/kube-apiserver --logtostderr=true --v=4 --etcd-servers=https://192.168.0.11:2379,https://192.168.0.12:2379,https://192.168.0.13:2379 --bind-address=192.168.0.11 --secure-port=6443 --advertise-address=192.168.0.11 --allow-privileged=true --service-cluster-ip-range=10.254.0.0/16 --enable-admission-plugins=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota,NodeRestriction --authorization-mode=RBAC,Node --enable-bootstrap-token-auth --token-auth-file=/k8s/kubernetes/cfg/token.csv --service-node-port-range=30000-50000 --tls-cert-file=/k8s/kubernetes/ssl/server.pem --tls-private-key-file=/k8s/kubernetes/ssl/server-key.pem --client-ca-file=/k8s/kubernetes/ssl/ca.pem --service-account-key-file=/k8s/kubernetes/ssl/ca-key.pem --etcd-cafile=/k8s/etcd/ssl/ca.pem --etcd-certfile=/k8s/etcd/ssl/server.pem --etcd-keyfile=/k8s/etcd/ssl/server-key.pem

## 
[root@elasticsearch01 bin]# netstat -tulpn |grep kube-apiserve
tcp        0      0 192.168.0.11:6443          0.0.0.0:*               LISTEN      22060/kube-apiserve 
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      22060/kube-apiserve 






3）部署kube-scheduler组件 创建kube-scheduler配置文件
[https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/](https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

```

vim  /k8s/kubernetes/cfg/kube-scheduler 
KUBE_SCHEDULER_OPTS="--logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect"
```
## 
参数备注： --address：在 127.0.0.1:10251 端口接收 http /metrics 请求；kube-scheduler 目前还不支持接收 https 请求； --kubeconfig：指定 kubeconfig 文件路径，kube-scheduler 使用它连接和验证 kube-apiserver； --leader-elect=true：集群运行模式，启用选举功能；被选为 leader 的节点负责处理工作，其它节点为阻塞状态；




创建kube-scheduler systemd文件
```

vim /usr/lib/systemd/system/kube-scheduler.service 
 
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
 
[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-scheduler
ExecStart=/k8s/kubernetes/bin/kube-scheduler $KUBE_SCHEDULER_OPTS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
```
## 



启动服务
```

systemctl daemon-reload
systemctl enable kube-scheduler.service 
systemctl start kube-scheduler.service
```
## 
[root@elasticsearch01 bin]# systemctl status kube-scheduler.service
● kube-scheduler.service - Kubernetes Scheduler
   Loaded: loaded (/usr/lib/systemd/system/kube-scheduler.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-12-27 15:16:51 CST; 17s ago
     Docs: https://github.com/kubernetes/kubernetes
 Main PID: 29026 (kube-scheduler)
   CGroup: /system.slice/kube-scheduler.service
           └─29026 /k8s/kubernetes/bin/kube-scheduler --logtostderr=true --v=4 --master=127.0.0.1:8080 --leader-elect



           
           
           
4）部署kube-controller-manager组件 创建kube-controller-manager配置文件
[https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/](https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)


```
vim /k8s/kubernetes/cfg/kube-controller-manager
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=true \
--v=4 \
--master=127.0.0.1:8080 \
--leader-elect=true \
--address=127.0.0.1 \
--service-cluster-ip-range=10.254.0.0/16 \
--cluster-name=kubernetes \
--cluster-signing-cert-file=/k8s/kubernetes/ssl/ca.pem \
--cluster-signing-key-file=/k8s/kubernetes/ssl/ca-key.pem  \
--root-ca-file=/k8s/kubernetes/ssl/ca.pem \
--service-account-private-key-file=/k8s/kubernetes/ssl/ca-key.pem"
```
## 


创建kube-controller-manager systemd文件
```

vim /usr/lib/systemd/system/kube-controller-manager.service 
 
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
 
[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-controller-manager
ExecStart=/k8s/kubernetes/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
```
## 



启动服务
```

systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
```

# [root@elasticsearch01 bin]# systemctl status kube-controller-manager
● kube-controller-manager.service - Kubernetes Controller Manager
   Loaded: loaded (/usr/lib/systemd/system/kube-controller-manager.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-12-27 15:19:19 CST; 11s ago
     Docs: https://github.com/kubernetes/kubernetes
 Main PID: 29510 (kube-controller)
   CGroup: /system.slice/kube-controller-manager.service
           └─29510 /k8s/kubernetes/bin/kube-controller-manager --logtostderr=true --v=4 --master=127.0.0.1:8080 --l




           
4.7 验证kubeserver服务
```
设置环境变量

vim /etc/profile.d/kubernetes.sh
export PATH=/k8s/kubernetes/bin:$PATH
```
source /etc/profile

查看master服务状态
kubectl get cs,nodes

```
[root@elasticsearch01 bin]# kubectl get cs,nodes
NAME                                 STATUS    MESSAGE             ERROR
componentstatus/controller-manager   Healthy   ok                  
componentstatus/scheduler            Healthy   ok                  
componentstatus/etcd-0               Healthy   {"health":"true"}   
componentstatus/etcd-1               Healthy   {"health":"true"}   
componentstatus/etcd-2               Healthy   {"health":"true"}   
```

---
# 


Node部署
# kubernetes work 节点运行如下组件：
docker
kubelet
kube-proxy
flannel
系统环境
CentOS Linux release 7.4.1708 (Core)
Docker版本
Server Version: 18.09.0
Cgroup Driver: cgroupfs  



5.1 Docker环境安装
```
yum install  yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce
systemctl start docker && systemctl enable docker
```




```
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT
```



# 5.2 部署kubelet组件
## kublet 运行在每个 worker 节点上，接收 kube-apiserver 发送的请求，管理 Pod 容器，执行交互式命令，如exec、run、logs 等; kublet 启动时自动向 kube-apiserver 注册节点信息，内置的 cadvisor 统计和监控节点的资源使用情况; 为确保安全，只开启接收 https 请求的安全端口，对请求进行认证和授权，拒绝未授权的访问(如apiserver、heapster)   


1）安装二进制文件
```
wget https://dl.k8s.io/v1.13.1/kubernetes-node-linux-amd64.tar.gz
tar xf kubernetes-node-linux-amd64.tar.gz
cd kubernetes/node/bin/
cp kube-proxy kubelet kubectl /k8s/kubernetes/bin/     
```
##            
           

2）复制相关证书到node节点
```

[root@elasticsearch01 ssl]# scp *.pem 192.168.0.12:$PWD
root@192.168.0.12's password: 
ca-key.pem                                                                                         100% 1679   914.6KB/s   00:00    
ca.pem                                                                                             100% 1359     1.0MB/s   00:00    
kube-proxy-key.pem                                                                                 100% 1675     1.2MB/s   00:00    
kube-proxy.pem                                                                                     100% 1403     1.1MB/s   00:00    
server-key.pem                                                                                     100% 1679   809.1KB/s   00:00    
server.pem     
```
## 



3）创建kubelet bootstrap kubeconfig文件 通过脚本实现
```

vim /k8s/kubernetes/cfg/environment.sh
#!/bin/bash
#创建kubelet bootstrapping kubeconfig 
BOOTSTRAP_TOKEN=57cf77e5364ec551d908c983e2c05878
KUBE_APISERVER="https://192.168.0.11:6443"
#设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/k8s/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig
 
#设置客户端认证参数
kubectl config set-credentials kubelet-bootstrap \
  --token=${BOOTSTRAP_TOKEN} \
  --kubeconfig=bootstrap.kubeconfig
 
# 设置上下文参数
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=bootstrap.kubeconfig
 
# 设置默认上下文
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
 
#----------------------
 
# 创建kube-proxy kubeconfig文件
 
kubectl config set-cluster kubernetes \
  --certificate-authority=/k8s/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig
 
kubectl config set-credentials kube-proxy \
  --client-certificate=/k8s/kubernetes/ssl/kube-proxy.pem \
  --client-key=/k8s/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
 
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
 
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig




执行脚本

[root@elasticsearch02 cfg]# sh environment.sh 
Cluster "kubernetes" set.
User "kubelet-bootstrap" set.
Context "default" created.
Switched to context "default".
Cluster "kubernetes" set.
User "kube-proxy" set.
Context "default" created.
Switched to context "default".
```

```
[root@elasticsearch02 cfg]# ls
bootstrap.kubeconfig  environment.sh  kube-proxy.kubeconfig
```
## 


4）创建kubelet参数配置模板文件
```

192.168.0.12:
vim /k8s/kubernetes/cfg/kubelet.config
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 192.168.0.12
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS: ["10.254.0.10"]
clusterDomain: cluster.local.
failSwapOn: false
authentication:
  anonymous:
    enabled: true
    
    

192.168.0.13:
vim /k8s/kubernetes/cfg/kubelet.config
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 192.168.0.13
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS: ["10.254.0.10"]
clusterDomain: cluster.local.
failSwapOn: false
authentication:
  anonymous:
    enabled: true    
```
## 




5）创建kubelet配置文件
[https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet/](https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)

```

192.168.0.12
vim /k8s/kubernetes/cfg/kubelet
KUBELET_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=192.168.0.12 \
--kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig \
--bootstrap-kubeconfig=/k8s/kubernetes/cfg/bootstrap.kubeconfig \
--config=/k8s/kubernetes/cfg/kubelet.config \
--cert-dir=/k8s/kubernetes/ssl \
--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"


备用
--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"



192.168.0.13
vim /k8s/kubernetes/cfg/kubelet
KUBELET_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=192.168.0.13 \
--kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig \
--bootstrap-kubeconfig=/k8s/kubernetes/cfg/bootstrap.kubeconfig \
--config=/k8s/kubernetes/cfg/kubelet.config \
--cert-dir=/k8s/kubernetes/ssl \
--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0"
```
## 




6）创建kubelet systemd文件
```

vim /usr/lib/systemd/system/kubelet.service 
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service
 
[Service]
EnvironmentFile=/k8s/kubernetes/cfg/kubelet
ExecStart=/k8s/kubernetes/bin/kubelet $KUBELET_OPTS
Restart=on-failure
KillMode=process
 
[Install]
WantedBy=multi-user.target
```
## 



7）将kubelet-bootstrap用户绑定到系统集群角色
```

kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```
## 
注意这个默认连接localhost:8080端口，可以在master上操作


[root@elasticsearch01 ssl]# kubectl create clusterrolebinding kubelet-bootstrap \
>   --clusterrole=system:node-bootstrapper \
>   --user=kubelet-bootstrap
clusterrolebinding.rbac.authorization.k8s.io/kubelet-bootstrap created




8）启动服务 
```
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
```
## 
[root@elasticsearch02 cfg]# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-12-27 17:34:30 CST; 18s ago
 Main PID: 24676 (kubelet)
   Memory: 88.6M
   CGroup: /system.slice/kubelet.service
           └─24676 /k8s/kubernetes/bin/kubelet --logtostderr=true --v=4 --hostname-override=192.168.0.11 --kubeconfig=/k8s/kuber



9）Master接受kubelet CSR请求 可以手动或自动 approve CSR 请求。推荐使用自动的方式，因为从 v1.8 版本开始，可以自动轮转approve csr 后生成的证书，如下是手动 approve CSR请求操作方法 查看CSR列表
```

[root@elasticsearch01 ssl]# kubectl get csr
NAME                                                   AGE   REQUESTOR           CONDITION
node-csr-gUwF0auDpLFC0lpBEGzsTs3dJbLC7OicX8wzKAlLQnI   49s   kubelet-bootstrap   Pending




接受node
[root@elasticsearch01 ssl]# kubectl certificate approve node-csr-gUwF0auDpLFC0lpBEGzsTs3dJbLC7OicX8wzKAlLQnI

certificatesigningrequest.certificates.k8s.io/node-csr-ij3py9j-yi-eoa8sOHMDs7VeTQtMv0N3Efj3ByZLMdc approved





再查看CSR
[root@elasticsearch01 ssl]# kubectl get csr
NAME                                                   AGE     REQUESTOR           CONDITION
node-csr-ij3py9j-yi-eoa8sOHMDs7VeTQtMv0N3Efj3ByZLMdc   5m13s   kubelet-bootstrap   Approved,Issued
```
# 



5.3部署kube-proxy组件
## kube-proxy 运行在所有 node节点上，它监听 apiserver 中 service 和 Endpoint 的变化情况，创建路由规则来进行服务负载均衡 
1）创建 kube-proxy 配置文件
[https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/](https://v1-13.docs.kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)


```

192.168.0.12:
vim /k8s/kubernetes/cfg/kube-proxy
KUBE_PROXY_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=192.168.0.12 \
--cluster-cidr=10.254.0.0/16 \
--kubeconfig=/k8s/kubernetes/cfg/kube-proxy.kubeconfig"



192.168.0.13:
vim /k8s/kubernetes/cfg/kube-proxy
KUBE_PROXY_OPTS="--logtostderr=true \
--v=4 \
--hostname-override=192.168.0.13 \
--cluster-cidr=10.254.0.0/16 \
--kubeconfig=/k8s/kubernetes/cfg/kube-proxy.kubeconfig"
```
## 



2）创建kube-proxy systemd文件
```

vim /usr/lib/systemd/system/kube-proxy.service 
 
[Unit]
Description=Kubernetes Proxy
After=network.target
 
[Service]
EnvironmentFile=-/k8s/kubernetes/cfg/kube-proxy
ExecStart=/k8s/kubernetes/bin/kube-proxy $KUBE_PROXY_OPTS
Restart=on-failure
 
[Install]
WantedBy=multi-user.target
```
## 



=========================
=========================

test:
[Unit]
Description=Kubernetes Proxy
After=network.target

[Service]
WorkingDirectory=/var/lib/kube-proxy
ExecStart=/usr/local/bin/kube-proxy \
  --bind-address=192.168.205.10 \
  --v=4 \
  --proxy-mode=ipvs \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
Restart=on-failure

[Install]
WantedBy=multi-user.target

========================
========================




3）启动服务 
```
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
```
## 
[root@elasticsearch02 cfg]# systemctl status  kube-proxy
● kube-proxy.service - Kubernetes Proxy
   Loaded: loaded (/usr/lib/systemd/system/kube-proxy.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-12-27 18:31:42 CST; 11s ago
 Main PID: 5376 (kube-proxy)
   Memory: 40.9M
   CGroup: /system.slice/kube-proxy.service
           ‣ 5376 /k8s/kubernetes/bin/kube-proxy --logtostderr=true --v=4 --hostname-override=192.168.0.11 --cluster-cidr=10.254.0.0/...
    
   


4）查看集群状态
```

[root@elasticsearch01 cfg]# kubectl get nodes
NAME        STATUS   ROLES    AGE     VERSION
192.168.0.12   Ready    <none>   9m15s   v1.13.1
```
## 



5）同样操作部署node 192.168.0.13并认证csr，认证后会生成kubelet-client证书
```

注意期间要是kubelet，kube-proxy配置错误，比如监听IP或者hostname错误导致node not found，需要删除kubelet-client证书，重启kubelet服务，重启认证csr即可

[root@elasticsearch03 kubernetes]# ls ssl
ca-key.pem  kubelet-client-2018-12-27-20-13-52.pem  kubelet.crt  kube-proxy-key.pem  server-key.pem
ca.pem      kubelet-client-current.pem              kubelet.key  kube-proxy.pem      server.pem



[root@elasticsearch01 ~]# kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
192.168.0.13   Ready    <none>   13h   v1.13.1
192.168.0.12   Ready    <none>   14h   v1.13.1
```
# 






Flanneld网络部署(所有节点)
# 默认没有flanneld网络，Node节点间的pod不能通信，只能Node内通信，为了部署步骤简洁明了，故flanneld放在后面安装 flannel服务需要先于docker启动。flannel服务启动时主要做了以下几步的工作： 从etcd中获取network的配置信息 划分subnet，并在etcd中进行注册 将子网信息记录到/run/flannel/subnet.env中



6.1 etcd注册网段
```
[root@elasticsearch02 cfg]# /k8s/etcd/bin/etcdctl --ca-file=/k8s/etcd/ssl/ca.pem --cert-file=/k8s/etcd/ssl/server.pem --key-file=/k8s/etcd/ssl/server-key.pem --endpoints="https://192.168.0.11:2379,https://192.168.0.12:2379,https://192.168.0.13:2379"  set /k8s/network/config  '{ "Network": "10.254.0.0/16", "Backend": {"Type": "vxlan"}}'
```
# 
{ "Network": "10.254.0.0/16", "Backend": {"Type": "vxlan"}}

flanneld 当前版本 (v0.10.0) 不支持 etcd v3，故使用 etcd v2 API 写入配置 key 和网段数据； 写入的 Pod 网段 ${CLUSTER_CIDR} 必须是 /16 段地址，必须与 kube-controller-manager 的 --cluster-cidr 参数值一致；




6.2 flannel安装
## 1）解压安装
```

tar xf flannel-v0.10.0-linux-amd64.tar.gz
cp flanneld mk-docker-opts.sh /k8s/kubernetes/bin/
```
## 



2）配置flanneld
```

vim /k8s/kubernetes/cfg/flanneld
FLANNEL_OPTIONS="--etcd-endpoints=https://192.168.0.11:2379,https://192.168.0.12:2379,https://192.168.0.13:2379 -etcd-cafile=/k8s/etcd/ssl/ca.pem -etcd-certfile=/k8s/etcd/ssl/server.pem -etcd-keyfile=/k8s/etcd/ssl/server-key.pem -etcd-prefix=/k8s/network"



scp /k8s/kubernetes/cfg/flanneld root@192.168.0.12:/k8s/kubernetes/cfg/
scp /k8s/kubernetes/cfg/flanneld root@192.168.0.13:/k8s/kubernetes/cfg/
```
## 


创建flanneld systemd文件
```

vim /usr/lib/systemd/system/flanneld.service
[Unit]
Description=Flanneld overlay address etcd agent
After=network-online.target network.target
Before=docker.service
 
[Service]
Type=notify
EnvironmentFile=/k8s/kubernetes/cfg/flanneld
ExecStart=/k8s/kubernetes/bin/flanneld --ip-masq $FLANNEL_OPTIONS
ExecStartPost=/k8s/kubernetes/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/subnet.env
Restart=on-failure
 
[Install]
WantedBy=multi-user.target



注意
mk-docker-opts.sh 脚本将分配给 flanneld 的 Pod 子网网段信息写入 /run/flannel/docker 文件，后续 docker 启动时 使用这个文件中的环境变量配置 docker0 网桥； flanneld 使用系统缺省路由所在的接口与其它节点通信，对于有多个网络接口（如内网和公网）的节点，可以用 -iface 参数指定通信接口; flanneld 运行时需要 root 权限；


scp /usr/lib/systemd/system/flanneld.service root@192.168.0.12:/usr/lib/systemd/system/
scp /usr/lib/systemd/system/flanneld.service root@192.168.0.13:/usr/lib/systemd/system/
```
## 




3）配置Docker启动指定子网 修改EnvironmentFile=/run/flannel/subnet.env，ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS即可
```

vim /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
 
[Service]
Type=notify
EnvironmentFile=/run/flannel/subnet.env
ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target




scp /usr/lib/systemd/system/docker.service root@192.168.0.12:/usr/lib/systemd/system/
scp /usr/lib/systemd/system/docker.service root@192.168.0.13:/usr/lib/systemd/system/
```
## 




4）启动服务 注意启动flannel前要关闭docker及相关的kubelet这样flannel才会覆盖docker0网桥
```

systemctl daemon-reload
systemctl stop docker
systemctl start flanneld
systemctl enable flanneld
systemctl start docker
systemctl restart kubelet
systemctl restart kube-proxy
```
## 


5）验证服务
```
[root@elasticsearch02 bin]# cat /run/flannel/subnet.env 
DOCKER_OPT_BIP="--bip=10.254.35.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=false"
DOCKER_OPT_MTU="--mtu=1450"
DOCKER_NETWORK_OPTIONS=" --bip=10.254.35.1/24 --ip-masq=false --mtu=1450"



[root@elasticsearch02 bin]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 52:54:00:a4:ca:ff brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.12/24 brd 10.2.8.255 scope global eth0
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:06:0a:ab:32 brd ff:ff:ff:ff:ff:ff
    inet 10.254.35.1/24 brd 10.254.35.255 scope global docker0
       valid_lft forever preferred_lft forever
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN 
    link/ether 72:59:dc:2b:0a:21 brd ff:ff:ff:ff:ff:ff
    inet 10.254.35.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever



[root@elasticsearch01 k8s]# kubectl get nodes
NAME        STATUS   ROLES    AGE    VERSION
192.168.0.13   Ready    <none>   16h    v1.13.1
192.168.0.12   Ready    <none>   18h    v1.13.1
```
# 



-----------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------



kubectl 命令补全
```
echo "source <(kubectl completion bash)" >> /etc/profile
```



# 
pod
```
vim nginx_pod.yml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
spec:
  containers:
    - name: nginx
      image: 192.168.0.11:5000/wuxingge/nginx:latest
      ports:
        - containerPort: 80
        

kubectl create -f k8s_pod.yml


[root@k8s-master1 ~]# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE        NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          8m36s   10.254.7.2   192.168.0.12   <none>           <none>



kubectl delete -f k8s_pod.yml
```
        


进入pod
```
kubectl exec -it -n blog wordpress-deploy-864874b89d-sk4hr /bin/bash
```
#         
service
```

Service在Kubernetes是一个REST对象，类似于Pod。与所有REST对象一样，Service可以将定义POST到apiserver以创建新实例。
例如，假设您有一组Pods每个公开端口30000并带有标签"app=web"


vim nginx_service.yml 
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30000
  selector:
    app: web


    
kubectl create -f k8s_service.yml



[root@k8s-master1 ~]# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE    IP            NODE        NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          3m8s   10.254.79.2   192.168.0.12   <none>           <none>

[root@k8s-master1 ~]# kubectl get service -o wide
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes   ClusterIP   10.254.0.1    <none>        443/TCP        6h    <none>
nginx        NodePort    10.254.76.1   <none>        80:30000/TCP   23s   app=web




kubectl delete -f k8s_service.yml  
```
    
   
   



# kubernetes-dashboard

[https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#deploying-the-dashboard-ui](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#deploying-the-dashboard-ui)
[https://github.com/kubernetes/kubernetes/tree/7f23a743e8c23ac6489340bbb34fa6f1d392db9d/cluster/addons/dashboard](https://github.com/kubernetes/kubernetes/tree/7f23a743e8c23ac6489340bbb34fa6f1d392db9d/cluster/addons/dashboard)
[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)
[https://blog.csdn.net/nklinsirui/article/details/80581286](https://blog.csdn.net/nklinsirui/article/details/80581286)
[https://github.com/kubernetes/dashboard/issues/3472](https://github.com/kubernetes/dashboard/issues/3472)



# 自定义生成证书
```
[root@elasticsearch01 /]# mkdir /certs
[root@elasticsearch01 /]# openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
Generating a 2048 bit RSA private key
................+++
..............................................+++
writing new private key to 'certs/dashboard.key'
-----
No value provided for Subject Attribute C, skipped
No value provided for Subject Attribute ST, skipped
No value provided for Subject Attribute L, skipped
No value provided for Subject Attribute O, skipped
No value provided for Subject Attribute OU, skipped
```

```
[root@elasticsearch01 /]# ls /certs
dashboard.csr  dashboard.key
```

```
[root@elasticsearch01 /]# openssl x509 -req -sha256 -days 36500 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
Signature ok
subject=/CN=kubernetes-dashboard
Getting Private key
```

```
[root@elasticsearch01 /]# ls certs/
dashboard.crt  dashboard.csr  dashboard.key
```

```
[root@elasticsearch01 /]# kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kube-system
secret/kubernetes-dashboard-certs created
```


# 修改service配置，将type: ClusterIP改成NodePort,便于通过Node端口访问
```
[root@elasticsearch01 /]# wget https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
[root@elasticsearch01 /]# vim /k8s/yaml/kubernetes-dashboard.yaml 
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30443
  selector:
    k8s-app: kubernetes-dashboard
```
    
    
# 部署Kubernetes-dashboard
修改镜像地址为registry.cn-beijing.aliyuncs.com/minminmsn/kubernetes-dashboard:v1.10.1即可部署

```
[root@elasticsearch01 /]# vim /k8s/yaml/kubernetes-dashboard.yaml 
    spec:
      containers:
      - name: kubernetes-dashboard
        image: registry.cn-beijing.aliyuncs.com/minminmsn/kubernetes-dashboard:v1.10.1
```


        

```
[root@elasticsearch01 /]# kubectl create -f /k8s/yaml/kubernetes-dashboard.yaml 
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
Error from server (AlreadyExists): error when creating "/k8s/yaml/kubernetes-dashboard.yaml": secrets "kubernetes-dashboard-certs" already exists
```




```
[root@k8s-master1 yaml]# kubectl get pod -n kube-system -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
kubernetes-dashboard-cb55bd5bd-xrrs8   1/1     Running   0          35m   10.254.87.2   192.168.0.13   <none>           <none>
[root@k8s-master1 yaml]# kubectl get svc -n kube-system -o wide
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE   SELECTOR
kubernetes-dashboard   NodePort   10.254.207.120   <none>        443:31861/TCP   35m   k8s-app=kubernetes-dashboard
```



# 访问dashboard

选择token访问，token获取方法如下
```
[root@elasticsearch01 ~]# cat /k8s/yaml/admin-token.yaml 
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```


    
```
[root@elasticsearch01 yaml]# kubectl create -f admin-token.yaml 
clusterrolebinding.rbac.authorization.k8s.io/admin created
serviceaccount/admin created
```



```
[root@k8s-master1 ~]# kubectl describe secret/$(kubectl get secret -nkube-system |grep admin|awk '{print $1}') -nkube-system
Name:         admin-token-6qdlf
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin
              kubernetes.io/service-account.uid: 8484d410-8509-11e9-adf0-000c29b4d624


Type:  kubernetes.io/service-account-token


Data
====
ca.crt:     1359 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi02cWRsZiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6Ijg0ODRkNDEwLTg1MDktMTFlOS1hZGYwLTAwMGMyOWI0ZDYyNCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.Gb_T7kSzyPWh8bqcjCkfa7ND0I6yIlU9GbSK8qgFKktwyZ3_1G7i9tBIYk2FIGsqNJB9FiPx1DaKFz-h91pOGqA9RiGHdcABItKmcDSQcqCF7xw4OWRmgnT18HA_cweFRmywc24MuhN7scCp4jtpeFlaMN0ANC7NCG4b5ybiAJVvnFIKkiw6vbTgizBQX9qs2Op8miSHdoqzKU6ECcBMasDOGKu1jJuQVQJbTU9Zq_8o7jM7kdCoEOqu8ACye2W8P707LIR7pMD29cRnG3gf9-cCNxzf_yJMbB2YCX9V1VBpMFvIoIgvqN7GztTOrcawf5B_u2_VIHo27-g7EtgXRA
```




hosts解析
```
192.168.0.13  dashboard.wuxingge.com
```


https://192.168.0.13:41805

[https://dashboard.wuxingge.com:31861](https://dashboard.wuxingge.com:31861)







# 部署coredns

# 修改部署文件环境变量
在官网下载[https://github.com/coredns/deployment/tree/master/kubernetes](https://github.com/coredns/deployment/tree/master/kubernetes) 配置文件主要是**deploy.sh**和**coredns.yam.sed**，由于不是从kube-dns转到coredns，所以要注释掉kubectl相关操作，修改REVERSE_CIDRS、DNS_DOMAIN、CLUSTER_DNS_IP等变量为实际值，具体命令./deploy.sh -s -r 10.254.0.0/16 -i 10.254.0.10 -d clouster.local > coredns.yaml11


```
cd /coredns
./deploy.sh -s -r 10.254.0.0/16 -i 10.254.0.10 -d cluster.local > coredns.yaml
```



```
[root@k8s-master1 coredns]# diff coredns.yaml coredns.yaml.sed 
59c59
<         kubernetes cluster.local  10.254.0.0/16 {
---
>         kubernetes CLUSTER_DOMAIN REVERSE_CIDRS {
62c62
<         }
---
>         }FEDERATIONS
64c64
<         forward . /etc/resolv.conf
---
>         forward . UPSTREAMNAMESERVER
69c69
<     }
---
>     }STUBDOMAINS
171c171
<   clusterIP: 10.254.0.10
---
>   clusterIP: CLUSTER_DNS_IP
```



# 部署coredns
```
[root@elasticsearch01 coredns]# kubectl create -f coredns.yaml
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.extensions/coredns created
service/kube-dns created
```




# 修改kubelet dns服务参数并重启kubelet服务
[root@elasticsearch02 ~]# tail /k8s/kubernetes/cfg/kubelet
--v=4 \
--hostname-override=10.2.8.65 \
--kubeconfig=/k8s/kubernetes/cfg/kubelet.kubeconfig \
--bootstrap-kubeconfig=/k8s/kubernetes/cfg/bootstrap.kubeconfig \
--config=/k8s/kubernetes/cfg/kubelet.config \
--cert-dir=/k8s/kubernetes/ssl \
--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google-containers/pause-amd64:3.0 \
--cluster-dns=10.254.0.10 \
--cluster-domain=cluster.local. \
--resolv-conf=/etc/resolv.conf "


```
[root@elasticsearch02 ~]# systemctl restart kubelet.service 
[root@elasticsearch02 ~]# systemctl status kubelet.service 
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2019-01-03 16:00:20 CST; 6s ago
 Main PID: 31924 (kubelet)
   Memory: 80.2M
   CGroup: /system.slice/kubelet.service
           └─31924 /k8s/kubernetes/bin/kubelet --logtostderr=true --v=4 --hostname-override=10.2.8.65 --kubeconfig=/k8s/kubernetes...
```






# 使用busybox测试效果
注意：拿SVC服务来测试

```
[root@elasticsearch01 coredns]# kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools
If you don't see a command prompt, try pressing enter.
dnstools# nslookup kubernetes
Server:		10.254.0.10
Address:	10.254.0.10#53

Name:	kubernetes.default.svc.cluster.local
Address: 10.254.0.1
```


```
dnstools# nslookup nginx
Server:		10.254.0.10
Address:	10.254.0.10#53


Name:	nginx.default.svc.cluster.local
Address: 10.254.60.231
```


**nginx   解析为   cluster-ip**

```
[root@k8s-master1 ~]# kubectl get service -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes   ClusterIP   10.254.0.1      <none>        443/TCP        3d5h    <none>
nginx        NodePort    10.254.60.231   <none>        80:30000/TCP   2m14s   app=web
```






vim busybox.yaml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: busybox
    role: master
  name: busybox
spec:
  containers:
  - name: busybox
    image: myhub.fdccloud.com/library/busybox
    command:
    - sleep
    - "3600"
```


```
kubectl create -f busybox.yaml
```



```
[root@k8s-master1 ~]# kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          8s
```


```
[root@k8s-master1 ~]# kubectl exec -it busybox sh
/ # 
/ # 
/ # 
/ # nslookup kubernetes
Server:    10.254.0.10
Address 1: 10.254.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.254.0.1 kubernetes.default.svc.cluster.local

/ # nslookup kubernetes.default.svc.cluster.local
Server:    10.254.0.10
Address 1: 10.254.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default.svc.cluster.local
Address 1: 10.254.0.1 kubernetes.default.svc.cluster.local

/ # nslookup nginx
Server:    10.254.0.10
Address 1: 10.254.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx
Address 1: 10.254.159.187 nginx.default.svc.cluster.local
```









---




# 附:

报错:
**error: unable to upgrade connection: Forbidden (user=system:anonymous, verb=create, resource=nodes, subresource=proxy)**
```

绑定一个cluster-admin的权限
kubectl create clusterrolebinding system:anonymous   --clusterrole=cluster-admin   --user=system:anonymous
```



批量删除已停止容器
```
docker rm `docker ps -a |awk '/Exited/{print $1}'`
```

# 

部署wordpress
# 创建blog命名空间
```
kubectl create namespace blog
```

# 分步完成
## 第一步，创建一个MySQL的Deployment对象：(wordpress-db.yaml)

[root@k8s-master1 wordpress]# vim wordpress-db.yaml 
```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mysql-deploy
  namespace: blog
  labels:
    app: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
          name: dbport
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: rootPassW0rd
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          value: wordpress
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        hostPath:
          path: /var/lib/mysql
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: blog
spec:
  selector:
    app: mysql
  ports:
  - name: mysqlport
    protocol: TCP
    port: 3306
    targetPort: dbport
```

    
    

    
```
kubectl create -f wordpress-db.yaml
```



   
```
[root@k8s-master1 wordpress]# kubectl describe svc mysql -n blog
Name:              mysql
Namespace:         blog
Labels:            <none>
Annotations:       <none>
Selector:          app=mysql
Type:              ClusterIP
IP:                10.254.251.65
Port:              mysqlport  3306/TCP
TargetPort:        dbport/TCP
Endpoints:         10.254.87.3:3306
Session Affinity:  None
Events:            <none>
```


**可以看到Endpoints部分匹配到了一个Pod，生成了一个clusterIP：10.254.251.65**
**，现在我们是不是就可以通过这个****clusterIP****加上定义的3306端口就可以正常访问我们这个****mysql****服务了**

## 第二步. 创建Wordpress服务，将上面的wordpress的Pod转换成Deployment对象：（wordpress.yaml）

vim wordpress.yaml
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: wordpress-deploy
  namespace: blog
  labels:
    app: wordpress
spec:
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: wdport
        env:
        - name: WORDPRESS_DB_HOST
          value: 10.254.251.65:3306
        - name: WORDPRESS_DB_USER
          value: wordpress
        - name: WORDPRESS_DB_PASSWORD
          value: wordpress
```
          


```
kubectl create -f wordpress.yaml
```




```
[root@k8s-master1 wordpress]# kubectl get pods -n blog
NAME                                READY   STATUS    RESTARTS   AGE
mysql-deploy-99fb8596b-jrsr6        1/1     Running   0          9m12s
wordpress-deploy-77cd9f47b9-tsmwv   1/1     Running   0          47s
```




可以看到都已经是Running状态了，然后我们需要怎么来验证呢？是不是去访问下我们的wordpress服务就可以了，要访问，我们就需要建立一个能让外网用户访问的Service，前面我们学到过是不是NodePort类型的Service就可以？所以在上面的wordpress.yaml文件中添加上Service的信息：


```
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: blog
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
  - name: wordpressport
    protocol: TCP
    port: 80
    targetPort: wdport
```






```
[root@k8s-master1 wordpress]# kubectl apply -f wordpress.yaml 
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/wordpress-deploy configured
service/wordpress created
```





```
[root@k8s-master1 wordpress]# kubectl get pods -n blog -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
mysql-deploy-99fb8596b-2b4mm        1/1     Running   0          41m   10.254.87.3   192.168.0.13   <none>           <none>
wordpress-deploy-864874b89d-sk4hr   1/1     Running   0          41m   10.254.79.4   192.168.0.12   <none>           <none>
```

```
[root@k8s-master1 wordpress]# kubectl get service -n blog -o wide
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
mysql       ClusterIP   10.254.178.192   <none>        3306/TCP       41m   app=mysql
wordpress   NodePort    10.254.158.34    <none>        80:32255/TCP   41m   app=wordpress
```

```
[root@k8s-master1 wordpress]# kubectl get deployments -n blog -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES      SELECTOR
mysql-deploy       1/1     1            1           42m   mysql        mysql:5.7   app=mysql
wordpress-deploy   1/1     1            1           42m   wordpress    wordpress   app=wordpress
```







wordpress-deploy创建一个HPA对象，最小的 pod 副本数为1，最大为10，HPA会根据设定的 cpu使用率（10%）动态的增加或者减少pod数量

```
kubectl autoscale deployment wordpress-deploy --cpu-percent=10 --min=1 --max=10 -n blog
```









**如果mysql服务被重新创建了的话，它的clusterIP非常有可能就变化了，所以上面我们环境变量中的WORDPRESS_DB_HOST的值就会有问题，就会导致访问不了数据库服务了，这个地方我们可以直接使用Service的名称来代替host，这样即使clusterIP变化了，也不会有任何影响**


# 部署的应用整合到一个YAML文件中来：（wordpress-all.yaml）

[root@k8s-master1 wordpress]# cat wordpress-all.yaml 
```
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mysql-deploy
  namespace: blog
  labels:
    app: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
          name: dbport
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: rootPassW0rd
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          value: wordpress
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        hostPath:
          path: /var/lib/mysql

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: blog
spec:
  selector:
    app: mysql
  ports:
  - name: mysqlport
    protocol: TCP
    port: 3306
    targetPort: dbport


---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: wordpress-deploy
  namespace: blog
  labels:
    app: wordpress
spec:
  revisionHistoryLimit: 10
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      initContainers:
      - name: init-db
        image: busybox
        command: ['sh', '-c', 'until nslookup mysql; do echo waiting for mysql service; sleep 2; done;']
      containers:
      - name: wordpress
        image: wordpress
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: wdport
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql:3306
        - name: WORDPRESS_DB_USER
          value: wordpress
        - name: WORDPRESS_DB_PASSWORD
          value: wordpress
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: blog
spec:
  selector:
    app: wordpress
  type: NodePort
  ports:
  - name: wordpressport
    protocol: TCP
    port: 80
    nodePort: 32255
    targetPort: wdport
```
    
    
    

资源限制    
```
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
```





查看pod详细信息
```
kubectl describe pod -n blog wordpress-deploy-864874b89d-sk4hr
```





# wordpress持久化

[https://v1-13.docs.kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/](https://v1-13.docs.kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)


# 准备nfs
```
yum -y install nfs-utils
```



```
[root@k8s-master1 ~]# cat /etc/exports
/data *(rw,no_root_squash,sync)
```




```
mkdir /data/{web,db} -p
```


```
chown -R nfsnobody.nfsnobody /data
```

```
chmod -R 777 /data
```







# pv

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```



```
[root@k8s-master ~]# vim wordpress_nfs_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1-nfs
  namespace: blog
  labels:
    app: nfs1
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.0.11
    path: /data/web
```
    

# pvc
```
vim wordpress_nfs_pvc.yaml    
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc1-nfs
  namespace: blog
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      app: nfs1
```



# wordpress-all
```
[root@k8s-master1 wordpress]# vim wordpress-all.yaml 
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: mysql-deploy
  namespace: blog
  labels:
    app: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
          name: dbport
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: rootPassW0rd
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          value: wordpress
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        hostPath:
          path: /var/lib/mysql

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: blog
spec:
  selector:
    app: mysql
  ports:
  - name: mysqlport
    protocol: TCP
    port: 3306
    targetPort: dbport


---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: wordpress-deploy
  namespace: blog
  labels:
    app: wordpress
spec:
  replicas: 2
  revisionHistoryLimit: 10
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      initContainers:
      - name: init-db
        image: busybox
        command: ['sh', '-c', 'until nslookup mysql; do echo waiting for mysql service; sleep 2; done;']
      containers:
      - name: wordpress
        image: wordpress
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: wdport
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql:3306
        - name: WORDPRESS_DB_USER
          value: wordpress
        - name: WORDPRESS_DB_PASSWORD
          value: wordpress
        volumeMounts:
        - name: www
          mountPath: /var/www/html
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
      volumes:
      - name: www
        persistentVolumeClaim:
          claimName: pvc1-nfs

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: blog
spec:
  selector:
    app: wordpress
  type: NodePort
  ports:
  - name: wordpressport
    protocol: TCP
    port: 80
    nodePort: 32255
    targetPort: wdport
```







---

---





# wordpress持久化-2

[https://v1-13.docs.kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/](https://v1-13.docs.kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)


[https://v1-13.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://v1-13.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/)


[https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)


# 部署MySQL
## 创建Secret对象
```
kubectl create secret generic mysql-pass --from-literal=password=Wxg@123.com
```


```
kubectl get secrets
```


## pv
```
[root@k8s-master1 wordpress]# vim mysql_nfs_pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1-nfs
  labels:
    app: pv1-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.0.11
    path: /data/db
```

## pvc
```
[root@k8s-master1 wordpress]# vim mysql_nfs_pvc.yaml 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
```

## deployment_service
```
[root@k8s-master1 wordpress]# vim mysql-deployment.yaml 
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```




官方MySQL镜像
```
/etc/mysql/mysql.conf.d/mysqld.cnf
```



# 部署wordpress
## pv
```
[root@k8s-master1 wordpress]# vim wordpress_nfs_pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2-nfs
  labels:
    app: pv2-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 192.168.0.11
    path: /data/web
```

## pvc
```
[root@k8s-master1 wordpress]# vim wordpress_nfs_pvc.yaml 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
```


## deployment_service
```
[root@k8s-master1 wordpress]# vim wordpress-deployment.yaml 
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - name: wordpress
        image: wordpress:4.8-apache
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - name: wordpress
          containerPort: 80
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```






[root@k8s-master1 wordpress]# cat wordpress-deployment.yaml 
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - name: wordpress
      port: 80
      nodePort: 30737
  selector:
    app: wordpress
    tier: frontend
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - name: wordpress
        image: wordpress:4.8-apache
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - name: wordpress
          containerPort: 80
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```






# 附
[http://m.unixhot.com/kubernetes/kubernetes-kubeadm.html](http://m.unixhot.com/kubernetes/kubernetes-kubeadm.html)

[https://console.cloud.google.com/gcr/images/google-containers/GLOBAL](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL)



```
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.14.2
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.14.2
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.14.2
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.14.2
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd-amd64:3.2.24
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.5.1
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.4.1
docker pull mirrorgooglecontainers/coredns-amd64:1.2.2
```




```
docker pull gcr.io/google_containers/kube-apiserver-amd64:v1.14.2
docker pull gcr.io/google_containers/kube-controller-manager-amd64:v1.14.2
docker pull gcr.io/google_containers/kube-scheduler-amd64:v1.14.2
docker pull gcr.io/google_containers/kube-proxy-amd64:v1.14.2
docker pull gcr.io/google_containers/pause:3.1
docker pull gcr.io/google_containers/etcd-amd64:3.2.24
docker pull gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1
docker pull gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1
docker pull gcr.io/google_containers/kubernetes-dashboard-amd64:v1.4.1
docker pull gcr.io/google_containers/coredns:1.2.2
```




```
docker tag gcr.io/google_containers/kube-apiserver-amd64:v1.14.2           registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-apiserver-amd64:v1.14.2         
docker tag gcr.io/google_containers/kube-controller-manager-amd64:v1.14.2  registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-controller-manager-amd64:v1.14.2
docker tag gcr.io/google_containers/kube-scheduler-amd64:v1.14.2           registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-scheduler-amd64:v1.14.2         
docker tag gcr.io/google_containers/kube-proxy-amd64:v1.14.2               registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-proxy-amd64:v1.14.2             
docker tag gcr.io/google_containers/pause:3.1                              registry.cn-hangzhou.aliyuncs.com/wuxingge/pause:3.1                            
docker tag gcr.io/google_containers/etcd-amd64:3.2.24                      registry.cn-hangzhou.aliyuncs.com/wuxingge/etcd-amd64:3.2.24                    
docker tag gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1     registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard-amd64:v1.10.1   
docker tag gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1      registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard-amd64:v1.5.1    
docker tag gcr.io/google_containers/kubernetes-dashboard-amd64:v1.4.1      registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard-amd64:v1.4.1    
docker tag gcr.io/google_containers/coredns:1.2.2 registry.cn-hangzhou.aliyuncs.com/wuxingge/coredns:1.2.2
```



```
docker login --username=dong1226032602 registry.cn-hangzhou.aliyuncs.com
```



```
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-apiserver-amd64:v1.14.2         
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-controller-manager-amd64:v1.14.2
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-scheduler-amd64:v1.14.2         
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kube-proxy-amd64:v1.14.2             
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/pause:3.1                            
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/etcd-amd64:3.2.24                    
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard-amd64:v1.10.1   
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard-amd64:v1.5.1    
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/kubernetes-dashboard-amd64:v1.4.1  
docker push registry.cn-hangzhou.aliyuncs.com/wuxingge/coredns:1.2.2
```



  















