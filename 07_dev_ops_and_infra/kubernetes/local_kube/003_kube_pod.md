# 🚀 쿠버네티스 Pod(파드)

✅ Pod(파드)란?

- 도커에서는 하나의 프로그램을 실행시키는 단위를 컨테이너
- 쿠버네티스에서 하나의 프로그램을 실행시키는 단위 파드(Pod)
  - 파드는,
  - 쿠버네티스에서 가장 작은 단위
  - 일반적으로 하나의 파드가 하나의 컨테이너를 가짐
    (예외적으로 하나의 파드가 여러 개의 컨테이너를 가지는 경우도 있음)

## Pod 를 어떤 식으로 띄울 수 있을까?

- 쿠버네티스도 도커와 같이 이미지를 기반으로 파드를 띄워 실행시킨다.

### 실습

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  containers:
    - name: nginx-container
      image: nginx
      ports: # 이 컨테이너가 어떤 포트를 사용한다고 “명시”
        - containerPort: 80 # 컨테이너 내부에서 열리는 포트
```

- apiVersion

  - 이 리소스를 어떤 Kubernetes API 그룹/버전으로 해석할지 지정
  - v1 = core API group
  - Pod, Service, ConfigMap 같은 기본 리소스는 대부분 v1
  - Kubernetes는 API 기반 시스템 → 버전이 다르면 스펙도 다름

- kind

  - Pod = 쿠버네티스에서 가장 작은 실행 단위
  - 컨테이너 자체 ❌
  - 컨테이너를 감싸는 실행 래퍼
  - 하나의 Pod 에 컨테이너 1개 이상 가능

- metadata

  - metadata: 리소스의 식별 정보 영역
  - name: 클러스터 내에서 이 Pod를 구분하는 고유 이름

- spec

  - 이 리소스를 어떻게 만들고 실행하지에 대한 명세
  - 쿠버네티스 yaml 의 핵심은 항상 spec

#### ⚠️ 매우 중요한 포인트

```yaml
ports:
  - containerPort: 8001
```

- ❗ 이 줄은 포트를 “열어주지 않는다”
- 단순한 메타데이터
- Service / kubectl describe 에서 참고용
- 실제 포트 바인딩은 아님

### 위의 yaml 파일을 활용한 명령어

```shell
╰─ kubectl apply -f nginx-pod.yaml
pod/nginx-pod created


╰─ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          14s


╰─ curl localhost
curl: (7) Failed to connect to localhost port 80 after 0 ms: Couldn't connect to server


╰─ kubectl exec -it nginx-pod -- bash


╰─ root@nginx-pod:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
...


╰─ kubectl port-forward pod/nginx-pod 80:80
Unable to listen on port 80: Listeners failed to create with the following errors: [unable to create listener: Error listen tcp4 127.0.0.1:80: bind: permission denied unable to create listener: Error listen tcp6 [::1]:80: bind: permission denied]
error: unable to listen on any of the requested ports: [{80 80}]


╰─ sudo kubectl port-forward pod/nginx-pod 80:80
Password:
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80


╰─ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
...


╰─ kubectl delete pod nginx-pod
```

### Pod 와 Docker Container 의 중요한 차이점

- 도커는 컨테이너 내부와 외부의 네트워크가 서로 독립적으로 분리
- 쿠버네티스에서는 파드 내부의 네트워크를 컨테이너가 공유해서 같이 사용
- 파드의 네트워크는 로컬 컴퓨터의 네트워크와는 독립적으로 분리
  - 따라서, 위의 yaml 을 사용하여 pod 를 생성해도, localhost:80 에 들어갈 수 없는 이유가
    여기에 있음. Port Forwarding 을 해줘야 함
- 파드의 내부 네트워크를 외부에서도 접근할 수 있도록 포트포워딩(=포트 연결시키기) 활용

## Java/Spring pod 띄우기 실습

- myblod-backend 프로젝트 진입
- feature/kube-prac
- 프로젝트 루트에 Dockerfile 생성

```dockerfile
FROM amazoncorretto:21-alpine

COPY build/libs/myblog-backend-0.0.1-SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```shell
docker build -t spring-server .
```

```yml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
```

```shell
kubectl apply -f spring-pod.yml

╰─ kubectl get pods
NAME         READY   STATUS         RESTARTS   AGE
spring-pod   0/1     ErrImagePull   0          5s
```

### ErrImagePull 의 원인 -> ImagePullPolicy

- 이미지를 명시하였는데,
  - imagePullPolicy 이 없다면, ErrImagePull
  - imagePullPolicy 를 명시하여도, ImagePullBackOff

> 태그가 있어야 함

```yml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod

spec:
  containers:
    - name: spring-container:latest
      image: spring-server
      ports:
        - containerPort: 8080
      imagePullPolicy: Always
```

- imagePullPolicy
  - Always: 항상 로컬에서
  - IfNotPresent: 로컬에 없다면, 외부 레지스트리에서
