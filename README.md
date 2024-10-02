# 🌟 Spring Application Kubernetes Deployment Guide

## 📋 Overview
Spring 애플리케이션을 Docker로 이미지화하여 Docker Hub에 업로드하고, 이를 Kubernetes를 통해 배포하는 과정을 설명합니다. 이 단계별 설명을 통해 **쿠버네티스 환경에서 애플리케이션을 배포하고 외부에서 접근하는 방법**을 배울 수 있습니다.

## 🛠️ Prerequisites
- Docker 및 Docker Hub 계정
- Kubernetes 클러스터 (Minikube 사용)
- kubectl CLI

## 🚀 Steps to Deploy Spring Application on Kubernetes

### 1. Dockerfile 생성 📝
먼저, Spring 애플리케이션을 Docker 이미지로 만들기 위해 `Dockerfile`을 작성합니다.

```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY SpringApp-0.0.1-SNAPSHOT.jar /app/app.jar

EXPOSE 8999

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```
- FROM: OpenJDK 17을 사용하여 Java 환경을 설정합니다.
- WORKDIR: 애플리케이션이 실행될 작업 디렉토리를 /app으로 설정합니다.
- COPY: 빌드된 JAR 파일을 Docker 컨테이너의 /app 디렉토리로 복사합니다.
- EXPOSE: Spring 애플리케이션이 실행될 포트(8999)를 노출합니다.
- ENTRYPOINT: 컨테이너 실행 시 Java 애플리케이션을 실행합니다.

### 2. Docker 이미지 빌드 🏗️
위에서 작성한 Dockerfile을 기반으로 Docker 이미지를 빌드합니다.
```bash
docker build -t jiione/spring-app:latest .
```
- `-t` 옵션을 사용하여 이미지를 `<Dockerhub id>/spring-app:latest`로 태그합니다.

### 3. Docker Hub에 이미지 업로드 📤
이미지를 Docker Hub에 업로드하여 Kubernetes에서 이를 사용할 수 있도록 준비합니다.
```
docker push jiione/spring-app:latest
```
- Docker Hub 계정에 로그인한 상태여야 하며, <Dockerhub id>/spring-app:latest 이름으로 이미지를 업로드합니다.

### 4. Kubernetes에 배포하기 🚢
이제 Kubernetes 클러스터에 애플리케이션을 배포합니다. 쿠버네티스의 `kubectl` 명령을 사용하여 Deployment를 생성합니다.
```
kubectl create deployment spring-app --image=jiione/spring-app:latest --replicas=3
```
- **deployment 생성**: spring-app 이름의 디플로이먼트를 생성하고 3개의 복제본을 생성합니다.
- **--image**: Docker Hub에 업로드한 이미지를 지정합니다.

### 5. 애플리케이션 서비스 노출 🌐
`kubectl expose` 명령어를 사용하여 배포한 애플리케이션을 외부에서 접근할 수 있도록 LoadBalancer 타입의 서비스를 생성합니다.
```
kubectl expose deployment spring-app --type=LoadBalancer --port=8999
```
- **--type=LoadBalancer**: 외부에서 접속할 수 있는 IP를 생성하여 로드밸런서 서비스를 구성합니다.
- **--port**: 애플리케이션이 사용할 포트를 지정합니다.

### 6. 외부 접속 설정 🌍
로컬 환경에서 Minikube를 사용하는 경우, **minikube tunnel** 명령어를 실행하여 LoadBalancer의 외부 IP를 설정할 수 있습니다.
```
minikube tunnel
```
- 이 명령어를 실행하면 서비스에 외부 IP가 할당되어 외부에서 접속이 가능해집니다.

### 7. 결과 확인 ✅
1. `kubectl get services` 명령어를 사용하여 서비스의 상태를 확인합니다.
```
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1        <none>           443/TCP          96m
spring-app   LoadBalancer   10.101.226.213   10.101.226.213   8999:30905/TCP   10s
```
2. 이제 `EXTERNAL-IP:8999`를 통해 애플리케이션에 접속이 가능해집니다.


