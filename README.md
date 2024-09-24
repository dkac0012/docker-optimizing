# 🐳 Docker 경량화 방법 분석

docker의 build작업을 경량화 하여 저장공간 확보, 배포 속도 개선, 보안성 확보를 할 수 있습니다. 

## 경량화 이미지 사용

docker image를 사용함에 있어 **경량화된 이미지**(Alpine Linux,destroless)가 존재한다.
경량화된 이미지를 사용할 경우 불필요한 도구들(curl,wget,bash)로 인한 접근 방지, 필수 패키지만 존재하여 유지보수 및 보안성 강화등의 이점을 얻을 수 있다.

## 레이어 수 최소화

여러 명령을 단일 명령어로 결합하여 Docker 이미지의 레이어 수를 줄입니다.

```bash RUN apt-get update
RUN apt-get install -y git
RUN apt-get clean
rm -rf /var/lib/apt/lists
```

```bash RUN apt-get update && \ 
apt-get install -y git && \ 
apt-get clean && \ 
rm -rf / var /lib/apt/lists/*
```

## 불필요한 파일 무시

dockerignore를 활용한 불필요한 파일 무시
```bash node_modules
Dockerfile*
.git
.github
.gitignore
dist/**
README.md
```

## 특정 태그 사용

태그를 사용하여 버전간의 충돌을 방지
```bash FROM nginx:<tag>```

## 🔬 경량화 파일 비교 분석

### default

```bash FROM openjdk:17
COPY . /usr/src/
WORKDIR /usr/src/project
RUN javac Main.java
CMD ["java", "Main"]
```

### Dockerfile.alpine

```bash FROM openjdk:17-alpine
COPY . /usr/src/
WORKDIR /usr/src/project
RUN javac Main.java
CMD ["java", "Main"]
```

### Dockerfile.distroless

```bash # Stage 1: 빌드 단계
FROM openjdk:17 AS build
WORKDIR /usr/src/project
COPY . .
RUN javac Main.java

# Stage 2: 실행 단계 (Distroless 사용)
FROM gcr.io/distroless/java17
COPY --from=build /usr/src/project /usr/src/project
WORKDIR /usr/src/project
CMD ["Main"]
```

### 이미지별 용량 확인

![image](https://github.com/user-attachments/assets/fbf17579-7501-4f4f-a9a8-989d8aa0cdf5)

default는 ubuntu 기본 이미지 파일로 가장 많은 용량을 차지하며 alpine > distroless 순으로 용량이 경량화 되는 것을 알 수 있다.


