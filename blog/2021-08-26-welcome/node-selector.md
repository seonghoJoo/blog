# 스케쥴링

## Labels and Selectors

We have deployed a number of PODs. They are labelled with `tier`, `env` and `bu`. How many PODs exist in the `dev` environment (`env`)?

```yaml
kubectl get pods --selector env=dev
```

경험에 따르면, CPU sort 하는 것도 시험에 나온다고 하였다. 

```yaml
kubectl top po -A --sort-by=cpu
```

How many PODs are in the `finance` business unit (`bu`)?

```yaml
kubectl get pods --selector bu=finance
```

How many objects are in the `prod` environment including PODs, ReplicaSets and any other objects?

```yaml
kubectl get all --selector env=prod
```

Identify the POD which is part of the `prod` environment, the `finance` BU and of `frontend` tier?

```yaml
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

replicaset-definition.yaml

```yaml
apiVersion : apps/v1
kind : ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: app1
    function: Front-end
spec:
  replicas: 3
  selector:
	matchLabels:
	  app: App1
	template:
	  metadata:
	    labels:
		  app: App1
		  function: Front-end
		spec:
		  containers:
		    - name : simple-webapp
			  image: simple-webapp
```

service-definition.yaml

```yaml
apiVersion : v1
kind : Service
metadata:
  name: my-service
spec:
  selector:
    app: App1
	ports:
	- protocol:TCP
	port : 80
	targetPort: 9376
```

## Node Selector

kubectl label nodes node-name label-key=label-value

ex)
kubectl label nodes node-1 size=Large 라고 설정을 하고

```yaml
spec:
	containers:
	- name : data-processor
	  image: data-processor
	nodeSelector: Large
```
## Node Affinitiy

### Node Affinity Type

```java
requiredDuringSchedulingIngnoredDuringExecution - Type1 : 강제성이 있음.
preferredDuringSchedulingIngnoredDuringExecution - Type2 : 어쩔 수 없으면 함.
```

Apply a label `color=blue` to node `node01`

```bash
kubectl label nodes node01 color=blue
```

Create a new deployment named `blue` with the `nginx` image and 3 replicas.

```bash
kubectl create deployment blue --image=nginx
kubectl scale deployment/blue --replicas=3
```

Which nodes `can` the pods for the `blue` deployment be placed on?

Make sure to check taints on both nodes!

```bash
kubectl describe node controlplane | grep Taints
kubectl describe node node1 | grep Tains

check Taint both!
```

Create a new deployment named `red` with the `nginx` image and `2` replicas, and ensure it gets placed on the `controlplane` node only.

Use the label key - `node-role.kubernetes.io/control-plane` - which is already set on the `controlplane` node.

```bash
kubectl create deployment red --image=nginx --replicas=2 --dry-run=client -o yaml  

kubectl create deployment red --image=nginx --replicas=2 --dry-run=client -o yaml > red.yaml 
 

affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

## Resource Requirements and Limit

개념) Kube-Scheduler가 노드의  Mem / CPU를 보고 pod를 Node에 할당 

### 할당 단위

CPU : 1m(최소 단위 이 밑으로 내려갈 수  없음.) 1 (CPU Core를 의미)

The `elephant` pod runs a process that consumes 15Mi of memory. Increase the limit of the `elephant` pod to 20Mi.

Delete and recreate the pod if required. Do not modify anything other than the required fields.

```bash
kubectl create pod elephant --image=polinux/stress  --dry-run=client -o yaml > elephant.yaml
 
kubectl replace -f elephant.yaml --force
```

## Daemon Sets

모든 노드에 각각 존재함. 모니터링에서 사용됨.

kube-scheduler의 영향을 받지 않음

```bash
 kubectl get daemonsets --all-namespaces
```

Which namespace is the `kube-proxy` Daemonset created in?

What is the image used by the POD deployed by the `kube-flannel-ds` **DaemonSet**?

```bash
kubectl describe daemonset kube-flannel-ds --namespace=kube-flannel
```

Deploy a **DaemonSet** for `FluentD` Logging.

Name: elasticsearch

Namespace: kube-system

Image: registry.k8s.io/fluentd-elasticsearch:1.20
kubectl create Daemonset elasticsearch --namespace=kube-system --image=registry.k8s.io/fluentd-elasticsearch:1.20 -o yaml > elasticsearch.yaml

```bash
kubectl create Daemonset은 없음

kubectl create deployment elasticsearch --namespace=kube-system --image=registry.k8s.io/fluentd-elasticsearch:1.20 --dry-run=client -o yaml

kubectl create deployment elasticsearch --namespace=kube-system --image=registry.k8s.io/fluentd-elasticsearch:1.20 --dry-run=client -o yaml > fluentD.yaml

```

## Static Pod

각 node 안 kubelet에서 생성을 함.

kube-scheduler의 영향을 받지 않음

pod.yaml은 `etc/kubernetes/manifests` 에 존재하며 이를 가져온다.

## Multiple Scheduler

scheduler 만들기

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
leaderElection:
  leaderElect: false
```

Pod에 custom scheduler 적용하는 방법 

```yaml
apiVersion : v1
kind : Pod
metadata:
  name: nginx
spec:
  containers:
	- image: nginx
	  name: nginx
	shcedulerName: my-custom-scheduler
```

### View Events

```bash
kubectl get evnets -o wide

kubectl logs my-custom-scheulder --name-space=kube-system
```

What is the name of the POD that deploys the default kubernetes scheduler in this environment?

```bash
kubectl get pods --namespace=kube-system

kubectl get pods -A
```

Deploy an additional scheduler to the cluster following the given specification.

Use the manifest file provided at `/root/my-scheduler.yaml`. Use the same image as used by the default kubernetes scheduler.

## Scheduling 4단계

Scheduling Queue → Filtering → Scoring → Binding

Scheduling Queue : PrioritySort

Filtering : NodeResourcesFit, NodeName, NodeUnshcedulable

Scoring : NodeResourcesFit, ImageLocality 

Binding : DefaultBinder