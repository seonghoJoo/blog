# 쿠버네시트 치팅시트

---

```bash
# Create a new pod with the nginx image.
kubectl run nginx --image=nginx

# Create a new pod with the name redis and the image redis123.
kubectl run redis --image=redis123 --dry-run=client -o yaml

kubectl create -f redis.yaml

# Now change the image on this pod to redis.
# Once done, the pod should be in a running state.
kubectl edit pod redis

# If you used a pod definition file then update the image from redis123 to redis 
# in the definition file via Vi or 
# Nano editor and then run kubectl apply command to update the image :-

```

What does the `READY` column in the output of the `kubectl get pods` command indicate?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e2420219-92c6-460d-a6e0-5b36d02defa0/1522f297-13ff-4a89-82a3-4f00325ba12d/Untitled.png)

```bash
# Delete any one of the 4 PODs.

# run replicaset
kubectl create -f replicaset.yaml

kubectl create -f deployment-definition.yaml

# Create a new Deployment with the below attributes using your own deployment definition file.
# Name: httpd-frontend;
# Replicas: 3;
# Image: httpd:2.4-alpine
```

먼저, 명령어를 사용할 때 아주 유용한 두 가지 옵션을 알아둬야 해요:

1. **`-dry-run`**: 이 옵션은 명령어를 실행했을 때 리소스가 바로 생성되지 않게 합니다. 즉, 명령어가 올바른지, 리소스를 생성할 수 있는지만 확인하고 싶을 때 **`-dry-run=client`** 옵션을 사용하세요.
2. **`o yaml`**: 이 옵션은 리소스 정의를 YAML 형식으로 화면에 출력합니다.

이 두 옵션을 조합해서 빠르게 리소스 정의 파일을 생성한 다음, 필요에 따라 수정하여 리소스를 생성할 수 있어요. 이 방법을 사용하면 처음부터 파일을 만들 필요가 없어서 편리하죠.

### **POD 생성**

- NGINX Pod 생성: **`kubectl run nginx --image=nginx`**
- POD 매니페스트 YAML 파일 생성 (생성하지 않음, **`-dry-run`**): **`kubectl run nginx --image=nginx --dry-run=client -o yaml`**

### **Deployment 생성**

- Deployment 생성: **`kubectl create deployment --image=nginx nginx`**
- Deployment YAML 파일 생성 (생성하지 않음, **`-dry-run`**): **`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`**
- 4개의 복제본이 있는 Deployment 생성: **`kubectl create deployment nginx --image=nginx --replicas=4`**
- Deployment 스케일링: **`kubectl scale deployment nginx --replicas=4`**
- YAML 파일 저장 및 수정 후 Deployment 생성: **`kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`**

### **Service 생성**

- ClusterIP 타입의 redis-service Service 생성하여 포드 redis를 6379 포트에 노출: **`kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml`**
- NodePort 타입의 nginx Service 생성하여 포드 nginx의 80 포트를 노드의 30080 포트에 노출: **`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`**

### pod 생성

kubectl expose pod redis --port=6379 --name redis-service
service/redis-service exposed

kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3

kubectl run custom-nginx --image=nginx --port=8080

Create a new deployment called `redis-deploy` in the `dev-ns` namespace with the `redis` image. It should have `2` replicas.

kubectl create deployment redis-deploy —image=redis —replicas=2 —namespace=dev-ns

### pod를 특정 node에 배포하기