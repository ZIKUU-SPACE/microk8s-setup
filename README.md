# Ubuntu ServerのMicroK8Sを設定する

## Ubuntu Serverのインストール

Ubuntu Serverをインストールする際のオプションでK8SとDockerを選択する。どちらもStable版。

インストールが終わったら、

- apt updateでリポジトリを更新
- apt upgradeでソフトウェアを更新
- ap install avahi-daemonでｍDNSを有効化
- カーネルを最新のものに更新
- NTPサーバーの日本のサーバーに変更
- タイムゾーンをAsia/Tokyoに変更

## /etc/hostsの修正

クラスターに参加するノードをホスト名とIPアドレスの関係を追加する。この説明では３つのホストk8s-master、k8s-worker1、k8s-worker2をクラスターするのが前提。

```
127.0.0.1 localhost
127.0.1.1 k8s-worker1

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# microk8sでクラスターに参加するホスト
192.168.0.240 k8s-master
192.168.0.242 k8s-worker1
192.168.0.243 k8s-worker2
```

## ノードの追加 [マスターノード（control plane）になるホストでの作業]

次のコマンドでログインユーザーをmicrok8sグループに追加する。

```
sudo usermod -aG microk8s $USER
```

ここで一旦ログアウトしログインし直す。

ノード追加のコマンドを実行。

```
microk8s add-node
```

こんな感じの出力が得られる。
```
From the node you wish to join to this cluster, run the following:
microk8s join 192.168.0.240:25000/ce3b9760fabb76e4375fabb0de4a0197/da43151e6288

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 192.168.0.240:25000/ce3b9760fabb76e4375fabb0de4a0197/da43151e6288 --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 192.168.0.240:25000/ce3b9760fabb76e4375fabb0de4a0197/da43151e6288
microk8s join 2400:4051:20a0:5300:a00:27ff:fec8:8857:25000/ce3b9760fabb76e4375fabb0de4a0197/da43151e6288

```

## ワーカーノードのクラスターへの参加 [ワーカーノードになるホストでの作業]

次のコマンドでログインユーザーをmicrok8sグループに追加する。

```
sudo usermod -aG microk8s $USER
```

ここで一旦ログアウトしログインし直す。

マスターノードでadd-nodeを実行した際の出力のこの部分
```
Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 192.168.0.240:25000/ce3b9760fabb76e4375fabb0de4a0197/da43151e6288 --worker
```

に表示されるコマンドを入力する。

```
microk8s join 192.168.0.240:25000/ce3b9760fabb76e4375fabb0de4a0197/da43151e6288 --worker
```

こんな表示が得られれば成功。

```
Contacting cluster at 192.168.0.240

The node has joined the cluster and will appear in the nodes list in a few seconds.

This worker node gets automatically configured with the API server endpoints.
If the API servers are behind a loadbalancer please set the '--refresh-interval' to '0s' in:
    /var/snap/microk8s/current/args/apiserver-proxy
and replace the API server endpoints with the one provided by the loadbalancer in:
    /var/snap/microk8s/current/args/traefik/provider.yaml
```

## クラスターの状況を確認する [マスターノード (control plane)になったホストで実行]

ノードの構成を確認する。

```
$ microk8s kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
k8s-master    Ready    <none>   39m   v1.27.5
k8s-worker1   Ready    <none>   15s   v1.27.5
```

クラスターの状態。

```
$ microk8s status
microk8s is running
high-availability: no
  datastore master nodes: 192.168.0.240:19001
  datastore standby nodes: none
addons:
  enabled:
    dns                  # (core) CoreDNS
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
  disabled:
    cert-manager         # (core) Cloud native certificate management
    community            # (core) The community addons repository
    dashboard            # (core) The Kubernetes dashboard
    gpu                  # (core) Automatic enablement of Nvidia CUDA
    host-access          # (core) Allow Pods connecting to Host services smoothly
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    ingress              # (core) Ingress controller for external access
    kube-ovn             # (core) An advanced network fabric for Kubernetes
    mayastor             # (core) OpenEBS MayaStor
    metallb              # (core) Loadbalancer for your Kubernetes cluster
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
    minio                # (core) MinIO object storage
    observability        # (core) A lightweight observability stack for logs, traces and metrics
    prometheus           # (core) Prometheus operator for monitoring and logging
    rbac                 # (core) Role-Based Access Control for authorisation
    registry             # (core) Private image registry exposed on localhost:32000
    storage              # (core) Alias to hostpath-storage add-on, deprecated
```

## ダッシュボード（管理用のWeb UI）を有効化する

Dashboardアドオンを有効化する。

```
$ microk8s enable dashboard
Infer repository core for addon dashboard
Enabling Kubernetes Dashboard
Infer repository core for addon metrics-server
Enabling Metrics-Server
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
clusterrolebinding.rbac.authorization.k8s.io/microk8s-admin created
Adding argument --authentication-token-webhook to nodes.
Metrics-Server is enabled
Applying manifest
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
secret/microk8s-dashboard-token created

If RBAC is not enabled access the dashboard using the token retrieved with:

microk8s kubectl describe secret -n kube-system microk8s-dashboard-token

Use this token in the https login UI of the kubernetes-dashboard service.

In an RBAC enabled setup (microk8s enable RBAC) you need to create a user with restricted
permissions as shown in:
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
```

ログインに必要なトークンを生成する。生成したトークンはどこかにコピペしておく。
```
$ microk8s kubectl create token default
eyJhbGciOiJSUzI1NiIsImtpZCI6IkxYSGhHc2hsZktGWFVWOGpwLTZEYXJ3Y1Y0Unh6aVBFZ20xMkFfN3NXN1UifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTY5NzcwMDY3MiwiaWF0IjoxNjk3Njk3MDcyLCJpc3MiOiJodHRwczovL2t1YmVybmV0ZXMuZGVmYXVsdC5zdmMiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImRlZmF1bHQiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6IjA0MmM1MzNkLWYzNWYtNDY4MS1iNzAyLTNmZTAxNDIzMWJlNCJ9fSwibmJmIjoxNjk3Njk3MDcyLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZWZhdWx0In0.CKSLBs_pP5kMlT09DBqgswP47P755oCHsIDRDckuM9FhdMXr8CRDBnoSYxEIl9AgXsox8BPEWlGBaH4_SPefy1AVJp2iPK6BGXL6fpWuWZNQj9W7OudYjD4TWi1Qv5aTMlR3CW4qKgUwAGG2PfZzXTrIXphQpjYUr9FHz3aFXDBSf8hy9RhAYOtdieA3CUKxg55xPoTAS41LdZ2hjaQuT5_okve3_3of6qMgQ_epvYXhh6CdyVcKr9VClhkXhL2MASVD9bjDBIp2rYxP-egAxalnp2SMFOR52l_695gXUNWtpA6iuiMoRugN6ACv2HC9ba1gerQ5P935RqDt9hzu4A

```

ダッシュボードを起動する。

```
$ microk8s kubectl port-forward --address 0.0.0.0 -n kube-system service/kubernetes-dashboard  10443:443
Forwarding from 0.0.0.0:10443 -> 8443
```

--address 0.0.0.0 というオプションはLAN上の他のマシンからダッシュボードにアクセスできるようにするためのもの。

## ダッシュボードへのアクセス

Webブラウザから https://192.168.0.240:10443 にアクセスする。
ログイン画面でコピペしておいたトークンを入力してログインする。