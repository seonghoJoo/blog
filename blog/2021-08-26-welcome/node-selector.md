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