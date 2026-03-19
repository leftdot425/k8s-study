# 9 퍼시스턴트 볼륨(PV)과 퍼시스턴트 볼륨 클레임(PVC)

* 쿠버네티스에서는 포드가 특정 노드에서만 생성되는 것이 아니므로 어느 노드에서도 접근할 수 있는 Persistent Volume을 사용하는 것이 일반적임
* 대표적인 Persistent Volume에는 NFS, AWS의 EBS, Ceph, GlusterFS이 존재함

## 9.1 로컬 볼륨: hostPath, emptyDir

* hostPath는 호스트와 볼륨을 공유하기 위해서 사용되며, emptyDir는 포드의 컨테이너 간에 볼륨을 공유하기 위해 사용됨

### 9.1.1 워커 노드의 로컬 디렉터리를 볼륨으로 사용: hostPath



### 9.1.2 포드 간의 컨테이너 간 임시 데이터 공유: emptyDir

## 9.2 네트워크 볼륨

## 9.3 PV, PVC를 이용한 볼륨 관리

### 9.3.1 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임을 사용하는 이유
### 9.3.2 퍼시스턴트 볼륨과 퍼시스턴트 볼륨 클레임 사용하기
### 9.3.3 퍼시스턴트 볼륨을 선택하기 위한 조건 명시
### 9.3.4 퍼시스턴트 볼륨의 라이프사이클과 Reclaim Policy
### 9.3.5 StorageClass와 Dynamic Provisioning