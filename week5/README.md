# 9 퍼시스턴트 볼륨(PV)과 퍼시스턴트 볼륨 클레임(PVC)

* 쿠버네티스에서는 포드가 특정 노드에서만 생성되는 것이 아니므로 어느 노드에서도 접근할 수 있는 Persistent Volume을 사용하는 것이 일반적임
* 대표적인 Persistent Volume에는 NFS, AWS의 EBS, Ceph, GlusterFS이 존재함

## 9.1 로컬 볼륨: hostPath, emptyDir

* hostPath는 호스트와 볼륨을 공유하기 위해서 사용되며, emptyDir는 포드의 컨테이너 간에 볼륨을 공유하기 위해 사용됨

### 9.1.1 워커 노드의 로컬 디렉터리를 볼륨으로 사용: hostPath

* 디플로이먼트의 포드에 장애가 생겨 다른 노드로 포드가 옮겨갈 경우 이전 노드에 저장된 데이터는 유실되기에 권장되지 않음
* 특정 노드에만 포드를 배치하는 방법이 있지만, 호스트 서버에 장애가 생기면 데이터는 마찬가지로 유실됨
* 단, 쿠버네티스의 모든 워커 노드에 배포하는 경우라면 hostPath를 사용하는 것을 고려할 수 있음

```bash
kubectl apply -f hostpath-pod.yaml

kubectl exec -it hostpath-pod -- ls /etc/data

kubectl delete -f hostpath-pod.yaml
```

### 9.1.2 포드 간의 컨테이너 간 임시 데이터 공유: emptyDir

* emptyDir는 그 이름처럼 포드가 실행되는 도중에만 필요한 휘발성 데이터를 각 컨테이너가 함께 사용할 수 있는 임시 저장 공간
* 처음 생성될 때 비어있으며 포드 삭제시 데이터도 함께 삭제됨

```bash
kubectl apply -f emptydir-pod.yaml
kubectl exec -it emptydir-pod -c content-creator -- /bin/sh
echo Hello, Kubernetes! >> /data/test.html

kubectl describe pod emptydir-pod | grep IP
# IP: 10.244.0.5

kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart=Never -- curl 10.244.0.5/test.html
# Hello, Kubernetes!
```

## 9.2 네트워크 볼륨

* 온프레스 환경에서 사용되는 네트워크 볼륨는 대표적으로 NFS, iSCSI, GlusterFS, Ceph가 존재
* AWS의 경우 EBS(Elastic Block Store), GCP의 경우 gcePersistentDisk와 같은 볼륨을 포드에 마운트할 수 있음
* 네트워크 볼륨의 위치는 특별히 정해진 것이 없으며(클러스터 내부, 외부 모두 가능) 네트워크로 접근할 수 있으면 됨
* 대개 데이터의 읽기 및 쓰기 속도, 마운트 방식, 네트워크 볼륨 솔루션 구축 비용 등을 기준으로 볼륨을 선택함

* 교재에서는 NFS(Network File System)을 실습으로 이용, NFS는 대부분의 운영체제에서 사용 가능하며, 여러 개의 클라이턴트가 동시에 마운트할 수 있음
* 안정성은 여러 개의 스토리지를 클러스터링하는 다른 솔루션에 비해 안정성이 떨어지나, 하나의 서버만으로 간편하게 사용 가능함
* NFS는 크게 영속적인 데이터가 실제로 저장되는 NFS 서버와 NFS 서버에 마운트해 스토리지에 파일을 읽고 쓰는 NFS 클라이언트로 구성됨
* NFS 서버를 도입하려면 백업 스토리지를 별도로 구축해 NFS의 데이터 손실에 대비하거나, NFS 서버의 설정 튜닝 및 NFS 서버에 접근하기 위한 DNS 이름을 준비해야함

```bash
kubectl apply -f nfs-deployment.yaml
kubectl apply -f nfs-service.yaml

export NFS_CLUSTER_IP=$(kubectl get svc/nfs-service -o jsonpath='{.spec.clusterIP}')
cat nfs-pod.yaml | sed "s/(NFS_SERVICE_IP}/$NFS_CLUSTER_IP/g" | kubectl apply -f -

kubectl get service nfs-service -o json
# ...
# "spec": {
#     "clusterIP": "10.110.28.75",
#     "clusterIPs": [
#         "10.110.28.75"
#     ],
# ...

kubectl exec -it nfs-pod -- sh
df -h
# Filesystem                Size      Used Available Use% Mounted on
# ...
# 10.110.28.75:/          335.0G      2.7G    332.4G   1% /mnt
# ...
```

## 9.3 PV, PVC를 이용한 볼륨 관리

### 9.3.1 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임을 사용하는 이유

* 볼륨과 애플리케이션의 정의를 분리하기 위해 PV와 PVC가 사용됨
* 즉, 포드가 세부적인 사항을 몰라도 볼륨을 사용할수 있도록 PV와 PVC가 추상화함(네트워크 볼륨이 EBS인지, NFS인지 상관 없음)

![PV]
(pv.png)

* 쿠버네티스 클러스터를 관리하는 인프라 관리자와 개발자가 있을 때, 사용자가 디플로이먼트의 포드에 볼륨을 마운트해 사용하려면 다음의 과정을 거침
    1. 인프라 관리자는 네트워크 볼륨의 정보를 이용해 퍼시스턴트 볼륨 리소스를 생성함, 이때 네트워크 볼륨의 정보에는NFS나 iSCSI와 같은 스토리지 서버에 마운트하기 위한 엔드포인트가 포함될 수 있음
    2. 사용자(개발자)는 포드를 정의하는 YAML 파일에 ‘이 포드는 데이터를 영속적으로 저장해야 하므로 마운트할 수 있는 외부 볼륨이 필요하다’라는 의미의 퍼시스턴트 볼륨 클레임을 명시하고, 해당 퍼시스턴트 볼륨 클레임을 생성함
    3. 쿠버네티스는 기존에 인프라 관리자가 생성해뒀던 퍼시스턴트 볼륨의 속성과 사용자가 요청한 퍼시스턴트 볼륨 클레임의 요구 사항이 일치한다면 두 개의 리소스를 매칭시켜 바인드(bind)함, 포드가 이 퍼시스턴트 볼륨 클레임을 사용함으로써 포드의 컨테이너 내부에 볼륨이 마운트된 상태로 생성됨
* 이 때 중요한 점은 사용자(개발자)는 디플로이먼트의 YAML 파일에 볼륨의 상세한 스펙을 정의하지 않아도 됨

### 9.3.2 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임 사용하기

```bash
kubectl get pv,pvc
```

### 9.3.3 퍼시스턴트 볼륨을 선택하기 위한 조건 명시
### 9.3.4 퍼시스턴트 볼륨의 라이프사이클과 Reclaim Policy
### 9.3.5 StorageClass와 Dynamic Provisioning