# 搭建Redis集群

## 集群初始化命令
集群初始化需借助外部工具
```bash
// 替换阿里源
$ cat > /etc/apt/sources.list << EOF
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
> EOF

// 安装工具
apt-get update
apt-get install -y vim wget python2.7 python-pip redis-tools dnsutils

pip install redis-trib==0.5.1



// 创建Master节点
redis-trib.py create \
  `dig +short redis-cluster-0.redis-service.default.svc.cluster.local`:6379 \
  `dig +short redis-cluster-1.redis-service.default.svc.cluster.local`:6379 \
  `dig +short redis-cluster-2.redis-service.default.svc.cluster.local`:6379

// 为每个Master节点添加从节点
redis-trib.py replicate \
  --master-addr `dig +short redis-cluster-0.redis-service.default.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-cluster-3.redis-service.default.svc.cluster.local`:6379

redis-trib.py replicate \
  --master-addr `dig +short redis-cluster-1.redis-service.default.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-cluster-4.redis-service.default.svc.cluster.local`:6379

redis-trib.py replicate \
  --master-addr `dig +short redis-cluster-2.redis-service.default.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-cluster-5.redis-service.default.svc.cluster.local`:6379

```