# 4장 레플리케이션과 그 밖의 컨트롤러: 관리되는 파드 배포

## 1. 파드를 안정적으로 유지하기
- 쿠버네티스에 컨테이너 목록을 제공하면 해당 컨테이너를 클러스터 어딘가에서 계속 실행되도록 할 수 있다는 것이다.
- 컨테이너의 주 프로세스에 크래시(Crash)가 발생하면 Kubelet이 컨테이너를 다시 시작한다.
    - 그러나 일례로 자바 애플리케이션이 메모리 누수가 있어서 OutofMemoryErrors를 발생시키기 시작하더라도 JVM 프로세스는 계속 실행될 것이다. (내부 애플리케이션 상태를 확인하는 방법) 
    - 애플리케이션이 무한 루프나 교착 상태에 빠져서 응답을 하지 않는 상황이라면? (외부 애플리케이션의 상태 확인 필요)

### 라이브니스 프로브 소개
- 라이브니스 프로브(liveness probe)를 통해 컨테이너가 살아 있는지 확인할 수 있다.
- 파드의 스펙에 각 컨테이너의 라이브니스 프로브를 지정할 수 있다.
- 주기적으로 프로브를 실행하고 프로브가 실패할 경우 컨테이너를 다시 시작한다.
- 쿠버네티스는 세 가지 메커니즘을 사용해 컨테이너에 프로브를 실행한다.
    - HTTP GET 프로브는 지정한 IP 주소, 포트, 경로에 HTTP GET 요청을 수행한다.
        - 응답 값이 오류를 나타내지 않는 경우 성공
        - 응답 값이 오류를 반환하거나 전혀 응답하지 않으면 실패 (다시 시작)
    - TCP 소켓 프로브는 컨테이너의 지정된 포트에 TCP 연결을 시도한다.
        - 연결 성공 시 성공
        - 연결 실패 시 실패 (다시 시작)
    - Exec 프로브는 컨테이너 내의 임의의 명령을 실행하고 명령의 종료 상태 코드를 확인한다.
        - 상태코드가 0이면 성공
        - 상태코드가 0이 아니면 실패 (다시 시작)

### 동작 중인 라이브니스 프로브 확인
```
kubectl get po kubia-liveness
```
```
// 실행 결과
NAME              READY   STATUS    RESTARTS   AGE
kubia-liveness     1/1    Running   0          7d16h
```