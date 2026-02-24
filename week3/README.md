# minikube 설치

```bash
brew install minikube
minikube start
```

# 6 쿠버네티스 시작하기
## 6.2 포드(Pod): 컨테이너를 다루는 기본 단위

### 6.2.1 포드 사용하기

```
# nginx-pod.yaml
kubectl apply -f nginx-pod.yaml

kubectl get pods

kubectl describe pod my-nginx-pod

# curl {ip_of_nginx} # 클러스터의 노드 중 한 곳에서 실행, Mac Minikube에서는 안됨

# 클러스터 노드로 접근 불가능하다면 테스트용 포드 생성
kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart=Never -- sh

kubectl delete -f nginx-pod.yaml # kubectl delete pod my-nginx-pod
```

### 6.2.2 포드 vs 도커 컨테이너

```
# nginx-pod-with-ubuntu.yaml
kubectl apply -f nginx-pod-with-ubuntu.yaml

kubectl exec -it my-nginx-pod -c ubuntu-sidecar-container -- bash
# 컨테이너 안에서 실행
# 포드 내의 컨테이너들이 네트워크 네임스페이스 등과 같은 리눅스 네임스페이스를 공유해서 사용하기에 nginx 서버로 접근 가능함
> curl localhost

kubectl delete -f nginx-pod-with-ubuntu.yaml
```

### 6.2.3 완전한 애플리케이션으로서의 포드
* 하나의 포드는 하나의 완전한 애플리케이션이며, 로그 수집 등이 필요할 때 사이드카 컨테이너를 생성해서 사용함

## 6.3 레플리카셋(Replica Set): 일정 개수의 포드를 유지하는 컨트롤러
* 사용이유: 정해진 수의 동일한 포드가 항상 실행되도록 관리(e.g. 노드 장애 등의 이유로 포드를 사용할 수 없을 때 다른 노드에서 포드를 다시 생성)

### 6.3.2 레플리카셋 사용하기

```
kubectl apply -f replicaset-nginx.yaml

kubectl get rs

kubectl delete -f replicaset-nginx.yaml
# 위와 같음
kubectl delete replicaset replicaset-nginx
kubectl delete rs replicaset-nginx
```

### 6.3.3 레플라케셋의 동작 원리
* 레플리카셋과 포드는 라벨 셀렉터를 통해 느슨한 연결을 유지하고 있음

### 6.3.4 래플리케이션 컨트롤러 vs 레플리카셋
* 래플리케이션 컨트롤러은 deprecated 되었으며, 레플리카셋부터 matchExpressions를 지원

## 6.4 디플로이먼트(Deployment): 레플리카셋, 포드의 배포를 관리

### 6.4.1 디플로이먼트 사용하기
* 대부분은 레플리카셋과 포드의 정보를 정의하는 디플로이먼트(Deployment) 오브젝트를 YAML파일에 정의해 사용함
* 디플로이먼트는 레플리카셋의 상위 오브젝트이기에 디플로이먼트를 생성하면 해당 디플로이먼트에 해당되는 레플리카셋도 함께 생성됨

```bash
kubectl apply -f deployment-nginx.yaml

kubectl get deploy
kubectl get rs
kubectl get pods

kubectl delete -f deployment-nginx.yaml
```

### 6.4.2 디플로이먼트를 사용하는 이유
* 애플리케이션의 업데이트와 배포를 더욱 편하게 하기 위함
    * 레플리카셋의 변경 사항을 저장하는 리비전(revision)을 남겨 롤백 가능하게 함
    * 무중단 서비스를 위해 포드의 롤링 업데이트의 전략을 지정할 수 있음

```
# --record=true 옵션으로 디플로이먼트를 변경하면 변경사항을 위와 같이 디플로이먼트에 기록함
kubectl apply -f deployment-nginx.yaml --record

kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record
kubectl get pods
kubectl get replicasets
# NAME                             DESIRED   CURRENT   READY   AGE
# my-nginx-deployment-6d48fb5b7    3         3         3       82s # 이전 레플리카셋
# my-nginx-deployment-6fdddc5dc8   0         0         0       6m31s

kubectl rollout history deployment my-nginx-deployment
# deployment.apps/my-nginx-deployment
# REVISION  CHANGE-CAUSE
# 1         kubectl apply --filename=deployment-nginx.yaml --record=true
# 2         kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record=true

# rollback to version1
kubectl rollout undo deployment my-nginx-deployment --to-revision=1
kubectl get replicasets
# NAME                             DESIRED   CURRENT   READY   AGE
# my-nginx-deployment-6d48fb5b7    0         0         0       4m30s
# my-nginx-deployment-6fdddc5dc8   3         3         3       9m39s

# 포드 템플릿으로부터 계산된 해시값은 각 레플리카셋의 라벨 셀렉터(matchLabels)에서 pod-template-nash라는 이름의 라벨값으로서 자동으로 설정됩니다.
# 따라서 여러 개의레 플리카셋은 겹치지 않는 라벨을 통해 포드를 생성하게 됩니다.

kubectl describe deploy my-nginx-deployment
# Name:                   my-nginx-deployment
# Namespace:              default
# CreationTimestamp:      Wed, 25 Feb 2026 08:26:44 +0900
# Labels:                 <none>
# Annotations:            deployment.kubernetes.io/revision: 3
#                         kubernetes.io/change-cause: kubectl apply --filename=deployment-nginx.yaml --record=true
# Selector:               app=my-nginx
# Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
# StrategyType:           RollingUpdate
# MinReadySeconds:        0
# RollingUpdateStrategy:  25% max unavailable, 25% max surge
# Pod Template:
#   Labels:  app=my-nginx
#   Containers:
#    nginx:
#     Image:         nginx:1.10
#     Port:          80/TCP
#     Host Port:     0/TCP
#     Environment:   <none>
#     Mounts:        <none>
#   Volumes:         <none>
#   Node-Selectors:  <none>
#   Tolerations:     <none>
# Conditions:
#   Type           Status  Reason
#   ----           ------  ------
#   Available      True    MinimumReplicasAvailable
#   Progressing    True    NewReplicaSetAvailable
# OldReplicaSets:  my-nginx-deployment-6d48fb5b7 (0/0 replicas created)
# NewReplicaSet:   my-nginx-deployment-6fdddc5dc8 (3/3 replicas created)
# Events:
#   Type    Reason             Age                  From                   Message
#   ----    ------             ----                 ----                   -------
#   Normal  ScalingReplicaSet  11m                  deployment-controller  Scaled up replica set my-nginx-deployment-6fdddc5dc8 from 0 to 3
#   Normal  ScalingReplicaSet  6m16s                deployment-controller  Scaled up replica set my-nginx-deployment-6d48fb5b7 from 0 to 1
#   Normal  ScalingReplicaSet  6m8s                 deployment-controller  Scaled down replica set my-nginx-deployment-6fdddc5dc8 from 3 to 2
#   Normal  ScalingReplicaSet  6m8s                 deployment-controller  Scaled up replica set my-nginx-deployment-6d48fb5b7 from 1 to 2
#   Normal  ScalingReplicaSet  6m1s                 deployment-controller  Scaled down replica set my-nginx-deployment-6fdddc5dc8 from 2 to 1
#   Normal  ScalingReplicaSet  6m1s                 deployment-controller  Scaled up replica set my-nginx-deployment-6d48fb5b7 from 2 to 3
#   Normal  ScalingReplicaSet  5m53s                deployment-controller  Scaled down replica set my-nginx-deployment-6fdddc5dc8 from 1 to 0
#   Normal  ScalingReplicaSet  111s                 deployment-controller  Scaled up replica set my-nginx-deployment-6fdddc5dc8 from 0 to 1
#   Normal  ScalingReplicaSet  110s                 deployment-controller  Scaled down replica set my-nginx-deployment-6d48fb5b7 from 3 to 2
#   Normal  ScalingReplicaSet  108s (x4 over 110s)  deployment-controller  (combined from similar events): Scaled down replica set my-nginx-deployment-6d48fb5b7 from 1 to 0

# remove resource
kubectl delete deployment,pod,rs --all
```