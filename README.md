# K8S-Project

# README.md: Docker 이미지 관리 및 레지스트리 구성

## 1. 사전 요구 사항 (Prerequisites)

이 프로젝트는 모든 워커 노드가 마스터 노드의 사설 레지스트리에서 이미지를 가져올 수 있도록 구성되어야 합니다.

* **사설 레지스트리 IP (Master Node):** 172.16.101.10:5000 (설치 완료 가정)
* **워커 노드 IP:** 172.16.101.20(worker1), 172.16.101.30(worker2)

## 2. 보안 설정: Insecure Registry 등록 (모든 노드 필수)

Docker 또는 Containerd가 사설 레지스트리(`172.16.101.10:5000`)에 HTTP(비보안)로 접속하도록 **모든 마스터 및 워커 노드**에 설정해야 합니다.

### A. Containerd 설정 (권장)

```bash
# 모든 노드(master, worker1, worker2)에 SSH로 접속 후 실행
sudo vi /etc/containerd/config.toml

### 실행 순서 ###
infra
project-ns.yml

[목적] 프로젝트의 모든 리소스(Pod, Service, Secret)가 존재할 격리 공간을 가장 먼저 생성합니다.

secrets.yml

[의존] 모든 애플리케이션 (DB, WAS)

[설명] WAS가 DB에 접속할 암호(light-db-secret, heavy-db-secret)를 먼저 생성하여 Pod에 주입할 준비를 합니다.

rbac.yml

[의존] provisioner.yml

[설명] Provisioner Pod가 PersistentVolume을 자동으로 생성하고 관리할 수 있도록 필수 권한을 부여합니다.

provisioner.yml

[의존] rbac.yml

[설명] PV를 자동으로 생성하는 '로봇' Pod를 배포하고 실행합니다. (이 Pod가 실행되어야 StorageClass가 작동합니다.)

ssd-sc.yml/hdd-sc.yml

[의존] provisioner.yml

[설명] Light Tier와 Heavy Tier가 사용할 다이나믹 스토리지 클래스(ssd-sc, hdd-sc)를 정의합니다.

metallb-config.yml

[의존] 없음

[설명] Ingress Controller가 외부 IP를 할당받을 수 있도록 MetalLB의 IP 대역(172.16.101.200-220)을 정의합니다.

main-ingress.yml

[의존] metallb-config.yml (IP) 및 모든 WAS/Web Service 이름

[설명] 모든 인프라가 준비된 후, 3tier.com에 대한 최종 라우팅 규칙(경로/호스트)을 클러스터에 적용합니다.

DB Tier: db/
1. service.yml
2. statefulset.yml (Service가 StatefulSet보다 먼저)

WAS Tier: was/
1. service.yml
2. deployment.yml
3. hpa.yml

Web Tier: web/
1. service.yml
2. deployment.yml

Monitoring: monitoring/
1. components.yaml
2. pro-values.yml (helm install) -> helm install <별칭> pro/pro/kube-prometheus-stack -f pro-values.yml
