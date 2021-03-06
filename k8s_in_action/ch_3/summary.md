# 3장 파드

## 1. 파드

- 함께 배치된 컨테이너 그룹, k8s의 기본 빌딩 블록

- 컨테이너를 개별적으로 배포하기 보다는 컨테이너를 지닌 파드로 배포하고 운영

  그러나 두개의 노드에 동시에는 걸쳐진 상태로 배포는 불가능

- 파드가 필요한 이유

  - 컨테이너는 단일 프로세스로만 실행하는것으로 설계

    -> 동일한 표준 출력으로 로그를 기록하기 때문에, 어떤 프로세스의 로그인지 파악.         	하기어렵다.

- 여러 프로세스를 단일 컨테이너로 묶지 않기 때문에 컨테이너를 함께 묶고 관리할 다른 상위 구조가 필요한데, 이게 파드가 필요한 이유다.

### 파드에서 컨테이너 간 부분 격리

- 컨테이너간 완벽하게 격리되어있지만, 그룹 안에 있는 컨테이너가 특정한 리소스를 공유하기 위해 각 컨테이너가 완벽하게 격리되지 않도록 해야한다.
- 쿠버네티스는 파드 안에 있는 모든 컨테이너가 자체 네임스페이스가 아닌 동일한 리눅스 네임스페이스를 공유하도록 도커를 설정한다.
- 파드의 모든 컨테이너는 동일한 네트워크 네임스페이스와 UTS 네임스페이스안에서 실행되기 때문에, 모든 컨테이너는 같은 호스트 이름과 네트워크 인터페이스를 공유한다.
  - UTS: Unix Timesharing System Namespace, 호스트 이름을 네임스페이스 별로 격리한다.
  - 최신 쿠버네티스와 도커는 동일한 PID 네입스페이스를 공유할 수 있지만, 기본적으로는 비활성화 상태이다.

### 컨테이너가 동일한 IP와 포트 공간을 공유하는 방법

- 동일한 파드 안의 컨테이너가 동일한 네트워크 네임스페이스에서 실행되기 때문에, 동일한 IP 주소와 포트를 공유하고 있다. 그렇기에 실행중인 프로세스간 동일한 포트가 사용하지 않도록 주의해야 한다.
- 파드 안에 있는 모든 컨테이너는 동일한 루프백 네트워크 인터페이스를 갖기 때문에, 컨테이너들이 로컬 호스트를 통해 서로 통신할 수 있다.

### 파드 간 플랫 네트워크

- 쿠버네티스 클러스터의 모든 파드는 하나의 플랫한 공유 네트워크 주소 공간에 상주한다. 그렇기에 다른 파드의 IP 주소를 사용해 접근하는 것이 가능하다.
- 컴퓨터 간의 LAN 통신과 비슷하다.

### 파드에서 컨테이너의 적절한 구성

- 2개의 서비스(프론트엔드, 백엔드)를 한개의 파드에 넣지 말아야 하는 이유
  - 항상 2개가 같은 파드에서 실행해야 할 이유는 없다.
  - 워커 노드가 2개가 있을 때 한개의 노드를 놀게 한다.
  - 스케일링 문제
    - 쿠버네티스는 파드가 스케일링의 기본단위다.
    - 프론트엔드와 백엔드가 서로 다른 스케일링 요구 사항을 가지고 있다면 만족하지 못한다.

- 파드에서 여러 컨테이너를 사용하는 경우
  - 애플리케이션이 하나의 주요 프로세스와 하나 이상의 부완 프로세스로 구성될 경우
  - 단일 파드로 넣을지, 두개의 별도 파드로 구성할지에 대한 질문
    1. 컨테이너를 함께 실행해야 하는가? 혹은 서로 다른 호스트에서 실행할 수 있는가?
    2. 여러 컨테이너가 모여 하나의 구성 요소를 나타내는가? 혹은 개별적인 구성 요소인가?
    3. 컨테이너가 함께 혹은 개별적으로 스케일링 되어야 하는가?
  - 기본적으로는 분리된 파드에서 컨테이너를 실행하는 것이 좋다.

## 2. YAML 또는 JSON 디스크립터로 파드 생성

- 쿠버네티스 리소스는 일반적으로 쿠버네티스 REST API 엔드포인트에 JSON 혹은 YAML 매니페스트를 전송해 생성한다.

- YAML 파일 사용 이점

  - 파일을 이용해서 오브젝트를 정의하면 버전 관리 시스템에 넣어서 관리하는 것이 가능해진다

  - 생성된 파드의 YAML 정의가 어떻게 되어 있는지 살펴보기 위한 커맨드

    ```
    kubectl get po <name> -o yaml
    ```

  - `-o`는 출력타입을 결정하는 옵션

  - Metadata: 이름, 네임스페이스, 레이블 및 파드에 관한 기타 정보를 포함

  - Spec: 파드 컨테이너, 볼륨, 기타 데이터 등 파드에 관한 실제 명서

  - Status: 파드 상태, 컨테이너 설명 및 상태, 파드 IP, 기타 정보 등

### kubectl create 명령으로 파드 만들기

- 커맨드

  ```
  // 커맨드 실행
  kubectl create 0f kubia-manual.yaml
  
  // 실행 결과
  pod/kubia-manual created
  
  // 생성 파드 목록 보기 실행
  kubectl get pods
  
  // 실행 결과
  NAME           READY   STATUS    RESTARTS   AGE
  kubia-manual   1/1     Running   0          11s
  kubia-zxzij    1/1     Running   0          1h
  ```

### 애플리케이션 로그 보기

- 파드 로그 보기

  ```
  kubectl logs kubia-manual
  ```

- 여러 컨테이너를 포함하는 파드

  ```
  kubectl logs kubia-manual -c kubia
  ```

  - 현재 존재하는 파드의 컨테이너 로그만 가져올 수 있고, 파드가 삭제되면 로그도 같이 삭제된다.
  - 파드 삭제 이후에도 로그를 보려면 로그를 중앙 저장소에 저장하는 설정을 해주면 된다.

### 파드에 요청 보내기

- 포트 포워딩: 서비스를 거치지 않고 특정 파드에 연결하고 싶을 때 사용한다.

- 커맨드

  ```
  // 포트 포워딩 실행
  kubectl port-forward kubia-manual 8888:8080
  
  // 결과
  Forwarding from 127.0.0.1:8888 -> 8080
  Forwarding from [::1]:8888 -> 8080
  
  // 로컬 호스트에서 명령 보내기
  curl localhost:8888
  
  // 결과
  You've hit kubia-manual
  ```

## 3. 레이블을 이용한 파드 구성

- 레이블
  - 파드와 모든 다른 쿠버네티스 리소스를 조직화할 수 있는 단순하면서 강력한 쿠버네티스 기능이다.
  - 각 파드에 대해 개별적으로 작업을 수행하기보다 특정 그룹에 속한 모든 파드에 관해 한 번에 작업하기를 원할때 사용한다.
  - 레이블은 리소스에 첨부하는 키-값 쌍으로, 이 쌍은 레이블 셀렉터를 사용해 리소스를 선택할 때 활용된다.

### 파드 생성
- 커멘드
  ```
  // 파드 생성 시 레이블 설정
  kubectl create -f kubia-manual-with-labels.yaml

  // 실행 결과
  pod/kubia-manual-v2 created
  ```
  kubectl get pods 명령은 레이블을 표시하지 않는 것이 기본값이라 --show-labels 스위치를 사용해 레이블을 볼 수 있다.

  
  ```
  // 파드 목록 조회
  kubectl get po --show-labels

  // 실행 결과
  NAME           READY   STATUS    RESTARTS   AGE     LABELS
  kubia-hrpbr     1/1    Running   0          7d16h   <none>
  kubia-htb95     1/1    Running   0          7d1h    <none>
  kubia-manual    1/1    Running   0          119s    <none>
  kubia-manual-v2 1/1    Running   0          119s    env=prod
  kubia-tctjg     1/1    Running   0          7d1h    run=kubia
  ```

## 4. 레이블 셀럭터를 이용한 파드 부분 집합 나열
> 레이블 셀렉터는 특정 값과 레이블을 갖는지 여부에 따라 리소스를 필터링하는 기준이 된다.
  - 특정한 키를 포함하거나 포함하지 않는 레이블
  - 특정한 키와 값을 가진 레이블
  - 특정한 키를 갖고 있지만, 다른 값을 가진 레이블

## 5. 레이블과 셀렉터를 이용해 파드 스케줄링 제한
> 쿠버네티스는 모든 노드를 하나의 대규모 배포 플랫폼으로 노출하기 때문에, 파드가 어느 노드에 스케줄링됐느냐는 중요하지 않다.
- 워커 노드 분류에 레이블 사용
  - 노드를 포함한 모든 쿠버네티스 오브젝트에 레이블을 부착할 수 있다.
- 특정 노드에 파드 스케줄링
  - 스케줄러가 특정 기능을 제공하는 노드를 선택하도록 요청하려면, 파드의 YAML 파일에 노드 셀렉터를 추가해야한다.
    > spec 섹션 안에 nodeSelector 필드를 추가했다. 파드를 생성할 때, 스케줄러는 특정 기능을 가지고있는 레이블이 설정된 노드 중에서 선택한다.
- 하나의 특정 노드로 스케줄링
  - 파드를 특정한 노드로 스케줄링하는 것도 가능하다. 그러나 nodeSelector에 실제 호스트 이름을 지정할 경우에 해당 노드가 오프라인 상태인 경우 파드가 스케줄링되지 않을 수 있다.

## 6. 파드에 어노테이션 달기
```
어노테이션은 키-값 쌍으로 레이블과 거의 비슷하지만 식별 정보를 갖지 않는다.

레이블은 오브젝트를 묶는 데 사용할 수 있지만, 어노테이션은 그렇게 할 수 없다.

레이블 셀렉터를 통해서 오브젝트를 선택하는 것이 가능하지만 어노테이션 셀럭터와 같은 것은 없다.
```
> 어노테이션이 유용하게 사용되는 경우는 파드나 다른 API 오브젝트에 설명을 추가해두는 것이다.

> 오브젝트를 만든 사람 이름을 어노테이션으로 지정해두면, 클러스터에서 작업하는 사람들이 좀 더 쉽게 협업할 수 있다.

### 7. 요약
- 특정 컨테이너를 파드로 묶어야 하는지 여부를 결정하는 방법.
- 파드는 여러 프로세스를 실행할 수 있으며 컨테이너가 아닌 세계의 물리적 호스트와 비슷하다.
- YAML 또는 JSON 디스크립터를 작성해 파드를 작성하고 파드 정의와 상태를 확인 할 수 있다.
- 레이블과 레이블 셀렉터를 사용해 파드를 조직하고 한 번에 여러 파드에서 작업을 쉽게 수행할 수 있다.
- 노드 레이블과 셀렉터를 사용해 특정 기능을 가진 노드에 파드를 스케줄링할 수 있다.
- 어노테이션을 사용하면 사람 또는 도구, 라이브러리에서 더 큰 데이터를 파드에 부착할 수 있다.
- 네임스페이스는 다른 팀들이 동일한 클러스터를 별도 클러스터를 사용하는 것처럼 이용할 수 있게 해준다.
- *!!중요!!* 네임스페이스를 사용하면 오브젝트를 별도 그룹으로 분리해 특정한 네임스페이스 안에 속한 리소스를 대상으로 작업할 수 있게 해주지만, 실행 중인 오브젝트에 대한 격리는 제공하지 않는다.
- kubectl explain 명령으로 쿠버네티스 리소스 정보를 빠르게 찾을 수 있다.