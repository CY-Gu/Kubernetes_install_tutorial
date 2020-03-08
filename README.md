---
title: Kubernetes Install Tutorial
tags: kubernetes, k8s, install
---

# Kubernetes安裝教學


首先，要安裝kubernetes，要先了解k8s是叢集(cluster)的架構，每個叢集當中會有一個Master node 和數個Slave/Worker node(以下稱Slave node)，視你的需求而定。因此，我們需要分別安裝Master node 和Slave node，兩者可安裝在不同的實體機器或虛擬機上，這邊示範安裝在VMWare Ubuntu18.04上。此外，虛擬機的配置有一些最低限制，例如 : 
- **Master node**
    - CPU : 2 core 
    - RAM : 4 G

:::warning 注意 
:zap: 只配1 core的話到時候Master節點會無法初始化~
:::

- **Slave node**
    - CPU : 1 core
    - RAM : 4 G

有些套件在Master和Slave上皆必須安裝，有些則只要單獨安裝在Master或Slave上即可。以下介紹如何安裝kubernetes以及如何將Slave node 加入Master node的cluster。


---

## On Both Master node & Slave node
### 以下這些步驟Master及Slave皆須執行
**Step 1 :**  更新 repositpories
```=
sudo su
apt-get update
```
**Step 2 :**  關閉 swap space
```=
swapoff -a
vim /etc/fstab
```
將 /swapfile 這一行註解起來(前方加上 '#' )。

**Step 3 (optional) :**  修改 hostname
```=
vim /etc/hostname
```
將原本名稱刪掉，改成Master和Node各自的hostname，可任取。
這邊我將Master改為kmaster，而Slave改為knode。

接著重啟虛擬機，可發現hostname成功修改！

**Step 4 :**  修改 host file
```=
vim /etc/hosts
```
在檔案內加入一行指令 : 
```=
<IP-Address-of-node><tab><hostname-of-node>
```
例如 : 
```=
192.168.67.128	kmaster
192.168.67.129	knode
```
PS : 同一個叢集的節點都需加入喔~

**Step 5 :**  安裝 ssh
```=
apt-get install openssh-server
```

**Step 6 :**  安裝 Docker
```=
apt-get update
apt-get install -y docker.io
```
**Step 7 :**  建立 kubernetes 環境
```=
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
```
**Step 8 :**  安裝 kubernetes 工具
```=
apt-get install -y kubelet kubeadm kubectl
```
**Step 9 :**  更新 kubernetes configuration
```=
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
在檔案內加入一行指令 : 
```=
Environment=”cgroup-driver=systemd/cgroup-driver=cgroupfs”
```


---

## Only On Master node

初始化Master node
```=
sudo kubeadm init --pod-network-cidr=<ip-of-container-network-interface> --apiserver-advertise-address=<ip-address-of-master>
```
若要使用Calico CNI，則將 **ip-of-container-network-interface** 改為**192.168.0.0/16**；若要使用Flannel CNI，則改為**10.244.0.0/16**。初始化後，即可看到以下訊息

```=
W0225 08:10:50.654131   11265 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0225 08:10:50.654560   11265 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.3
[preflight] Running pre-flight checks
	[WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kmaster kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.67.128]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kmaster localhost] and IPs [192.168.67.128 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kmaster localhost] and IPs [192.168.67.128 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0225 08:11:46.664565   11265 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0225 08:11:46.665274   11265 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 16.506641 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.17" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node kmaster as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kmaster as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 9gg19y.0i05a6asikxkdv41
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.67.128:6443 --token 9gg19y.0i05a6asikxkdv41 \
    --discovery-token-ca-cert-hash sha256:df730b9dc365367eca126dba28d64268e200fd0d81ebd0563668ede2cdd4eb7d 
```

接著，依序輸入系統給的三個指令
```=
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

並且將最後兩行指令存起來，之後knode要加入此叢集，執行此指令即可。
```=
kubeadm join 192.168.67.128:6443 --token 9gg19y.0i05a6asikxkdv41 \
    --discovery-token-ca-cert-hash sha256:df730b9dc365367eca126dba28d64268e200fd0d81ebd0563668ede2cdd4eb7d 
```

---

## Only On Slave node

這邊一般來說，只要將剛剛 kubeadm join... 指令輸入，即可成功加入集群。

但若是隔了一段時間後，有新的node想加入此叢集，會發現即使輸入正確指令，也無法成功 join，卡在以下畫面

```=
george@knode:~$ sudo kubeadm join 192.168.67.128:6443 --token 9gg19y.0i05a6asikxkdv41 \
>     --discovery-token-ca-cert-hash sha256:df730b9dc365367eca126dba28d64268e200fd0d81ebd0563668ede2cdd4eb7d 
[sudo] password for george: 
W0227 08:13:19.054898   14242 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
    
```


這是因為token時間太久失效了(token的有效期限default為24小時)，這時需要通過下列步驟生成一個新的token (在Master上生成)
```=
kubeadm token create
```
然後通過下列指令查看新的token
```=
root@kmaster:/home/george# kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
chhuh4.hcrk93gse24eggkz   23h         2020-02-28T08:16:49-08:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
```
獲取ca認證sha256編碼的hash值
```=
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

將舊token更換為新token，hash值也換成新的，再重新join一次，就成功加入拉~
```=
george@knode:~$ sudo kubeadm join 192.168.67.128:6443 --token chhuh4.hcrk93gse24eggkz     --discovery-token-ca-cert-hash sha256:df730b9dc365367eca126dba28d64268e200fd0d81ebd0563668ede2cdd4eb7d 
W0227 08:17:39.125117   15247 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.17" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
執行 **kubectl get no**，即可發現兩個node，代表安裝完成了，恭喜你！
```=
NAME      STATUS     ROLES    AGE   VERSION
kmaster   NotReady   master   2d    v1.17.3
knode     NotReady   <none>   36s   v1.17.3
```
---
## Additional Settings

但是，有沒有發現這兩個節點的STATUS都是 **NotReady**，所以我們還需要做一些額外工作。但是目前還不知道原因是甚麼，我們先查詢一下 log
```=
journalctl -f -u kubelet
```
會發現一直重複以下訊息
```=
root@kmaster:/home/george# journalctl -f -u kubelet
-- Logs begin at Tue 2020-02-25 07:24:12 PST. --
Feb 27 08:33:27 kmaster kubelet[999]: W0227 08:33:27.039411     999 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
Feb 27 08:33:27 kmaster kubelet[999]: E0227 08:33:27.423510     999 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
Feb 27 08:33:28 kmaster kubelet[999]: E0227 08:33:28.644022     999 summary_sys_containers.go:47] Failed to get system container stats for "/system.slice/docker.service": failed to get cgroup stats for "/system.slice/docker.service": failed to get container info for "/system.slice/docker.service": unknown container "/system.slice/docker.service"
```
這是因為 kubelet 參數多了 network-plugin=cni，但卻沒安裝 cni，所以打開設定檔把 network-plugin=cni 的參數移除。設定檔為 **/var/lib/kubelet/kubeadm-flags.env** (k8s v1.11以後版本皆適用)
```=
vim /var/lib/kubelet/kubeadm-flags.env
```
將 --network-plugin=cni 這一參數刪掉，修改後應是這樣
```=
KUBELET_KUBEADM_ARGS="--cgroup-driver=cgroupfs --pod-infra-container-image=k8s.gcr.io/pause:3.1 --resolv-conf=/run/systemd/resolve/resolv.conf"   
```
改完之後重新啟動
```=
systemctl daemon-reload
systemctl restart kubelet
```
再用 **kubectl get no**查看訊息
```=
root@kmaster:/home/george# kubectl get no
NAME      STATUS     ROLES    AGE   VERSION
kmaster   Ready      master   2d    v1.17.3
knode     NotReady   <none>   20m   v1.17.3
```
node狀態成功Ready，knode記得也要設定才會Ready喔~

---

但是又會發現，knode的 ROLES標記為 **\<none>**，而kmaster則標記了master，這是因為k8s只會標記master節點，其他節點default是沒有標記的，我們可以手動為任一節點設置 ROLES

```=
root@kmaster:/home/george# kubectl label node knode node-role.kubernetes.io/worker=worker
node/knode labeled
root@kmaster:/home/george# kubectl get no
NAME      STATUS   ROLES    AGE   VERSION
kmaster   Ready    master   2d    v1.17.3
knode     Ready    worker   27m   v1.17.3
```

成功啦，所有安裝步驟到這裡就OK囉，開始練習使用Kubernetes吧~
### Thank you! :sheep: 

You can find me on

- george4908090@gmail.com
