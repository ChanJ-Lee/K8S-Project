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
