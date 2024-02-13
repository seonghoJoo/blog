# 스케쥴링

## Labels and Selectors

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
