# Maintenace Cluster

---

## OS update 시 특정 node에 존재하는 pod를 치워줘야 한다.

```bash
kubectl drain node-1

kubectl cordon node-1
```

We need to take `node01` out for maintenance. Empty the node of all applications and mark it unschedulable.

```bash
kubectl drain node01 --ignore-daemonsets
```

The maintenance tasks have been completed. Configure the node `node01` to be schedulable again.

```bash
#Mark node as schedulable.
kubectl uncordon NODE

```

## Cluster Upgrade

1) 마스터 노드 drain
2) 마스터 노드의 kubeadm 업그레이드
3) 마스터 노드의 kubectl, kubelet 업그레이드
4) 마스터 노드 uncordon
5) 워커 노드 drain
6) 워커 노드의 kubeadm 업그레이드
7) 워커 노드의 kubectl, kubelet 업그레이드(kubectl이 없을 경우 kubelet만 업그레이드)
	kubectl은 단순 클라이언트 도구이기 때문에 워커 노드에 없는 경우도 있다. 이 경우 kubelet만 업그레이드 한다.
8) 워커 노드 uncordon

```bash
# 마스터에서 작업
ssh master

	# 1.29.x-00에서 x를 최신 패치 버전으로 바꾼다.
	apt-mark unhold kubeadm && \
	apt-get update && apt-get install -y kubeadm=1.29.x-00 && \
	apt-mark hold kubeadm
	
	# 다운로드 된 버전 확인
	kubeadm version
	
	# 업그레이드 계획 확인
	# 클러스터를 업그레이드할 수 있는지를 확인하고, 업그레이드할 수 있는 버전을 가져온다. 또한 컴포넌트 구성 버전 상태가 있는 표를 보여준다.
	kubeadm upgrade plan
	
	# 이 업그레이드를 위해 선택한 패치 버전으로 x를 바꾼다.
	sudo kubeadm upgrade apply v1.29.x

# 마스터에서 나오기
exit

# <node-to-drain>을 드레인하는 노드의 이름으로 바꾼다.cordon된다.
kubectl drain <node-to-drain> --ignore-daemonsets

# 마스터에서 작업
ssh master

	# 노드안에서 kubelet 및 kubectl을 업그레이드한다. 1.29.x-00의 x를 최신 패치 버전으로 바꾼다
	apt-mark unhold kubelet kubectl && \
	apt-get update && apt-get install -y kubelet=1.29.x-00 kubectl=1.29.x-00 && \
	apt-mark hold kubelet kubectl
	
	# kubelet 재시작
	sudo systemctl daemon-reload
	sudo systemctl restart kubelet

# 마스터에서 나오기
exit

# uncordon 해주기
kubectl uncordon <node-to-uncordon>

k get nodes

# worker node 차례로 작업..
```

```bash
apt-get upgrade -y kubeadm=1.12.0-00
# 클러스터 버전이 업그레이드가 되었다.
kubeadm upgrade apply v1.12.0
# kubelet 버전을 업그레이드 해야한다. (Master부터)
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
kubectl uncordon 

# 워커노드 드레인
kubectl drain node-1

# 워커노드 업그레이드
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00

kubeadm upgrade node config --kubelet-version v1.12.0
systemctl restart kubelet

```

### 강의 내용에 따르면..

```bash
# control plane으로 접속 (master 노드)
ssh controlplane
	
	# 버전 살피기
	cat /etc/*release*
	
	apt-cache madison kubeadm
	
	# 업그레이드를 한다고 해도 바로 되진 않음.
	sudo kubeadm upgrade apply v.1.25.10
	
	# ControlPlane에 위치한 pod들을 빼낼 것
	kubectl drain controlplane --ignore-daemonset
	
	# kubeadm 업그레이드 하기, 1.29.0-00에서 x를 최신 패치 버전으로 바꾼다.
	sudo apt-mark unhold kubeadm && \
	sudo apt-get update && sudo apt-get install -y kubeadm='1.29.0-*' && \
	sudo apt-mark hold kubeadm
	# kubeadm 버전 확인 1.29.0
	kubeadm version

	# kubelet 업그레이드 하기, 1.29.x-00에서 x를 최신 패치 버전으로 바꾼다.
	sudo apt-mark unhold kubelet kubectl && \
	sudo apt-get update && sudo apt-get install -y kubelet='1.29.0-*' kubectl='1.29.0-*' && \
	sudo apt-mark hold kubelet kubectl
		
	# Restart Kubelet
	systemctl restart daemon-reload
	systemctl restart kubelet
	
	# uncordon node
	kubectl uncordon controlplane
	
exit

# control node01으로 접속 (worker 노드)
ssh node01
	
	sudo kubeadm upgrade node

	# node01에 위치한 pod들을 빼낼 것
	kubectl drain controlplane --ignore-daemonset

	# kubelet 업그레이드 하기, 1.29.x-00에서 x를 최신 패치 버전으로 바꾼다.
	apt-mark unhold kubeadm && \
	apt-get update && apt-get install -y kubeadm=1.25.x-00 && \
	apt-mark hold kubeadm
	
# Restart Kubelet
	systemctl restart daemon-reload
	systemctl restart kubelet
	
	# uncordon node
	kubectl uncordon node01
	
exit

```

## Backup and Restore Methods

모든 포드 배포 서비스의 명령어 가져오기

```bash
kubectl get all -all namespaces -o yaml > all-deploy-sevrice.yaml

```

What is the version of ETCD running on the cluster?

Check the ETCD Pod or Process

```bash
kubectl get pods -A
> kube-system    etcd-controlplane   1/1     Running   0          2m38s

kubectl logs -n kube-system etcd-controlplane | grep -i 'etcd-version'
```

Where is the ETCD server certificate file located? > /etcd/kubernetes/pki/etcd/server.crt

Where is the ETCD CA Certificate file located?  > /etcd/kubernetes/pki/etcd/ca.crt

The master node in our cluster is planned for a regular maintenance reboot tonight. While we do not anticipate anything to go wrong, we are required to take the necessary backups. Take a snapshot of the **ETCD** database using the built-in **snapshot** functionality.

Store the backup file at location `/opt/snapshot-pre-boot.db`

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \ 
  snapshot save /opt/snapshot-pre-boot.db
```

Luckily we took a backup. Restore the original state of the cluster using the backup file.

보통 host pod들은 /var/lib/etcd에 저장되어있다. 다른 곳을 설정해주어야한다.

```bash
ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-from-what-iwant-dir snapshot restore /opt/snapshot-pre-boot.db

vi /etc/kubernetes/manifests/etcd.yaml
```