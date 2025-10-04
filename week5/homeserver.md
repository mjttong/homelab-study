# Week 5: homeserver config

## home network

![Image](/week5/homeserver/home-network.png)

- 홈 네트워크 구성을 다이어그램으로 정리함

## desktop1 뜯어보기

![Image](/week5/homeserver/desktop1.jpg)

- desktop1를 모두 뜯어보고, 내부 구조 파악 및 먼지 청소 진행

### 구성 요소

- CPU: i7-4790
- Memory: ddr3 삼성 8GB * 2
- GPU: Geforce GTX 750
- 메인보드: h81m-k
- 저장장치: SSD 2개(256GB, 120GB), HDD 1개(500GB)
- 파워 서플라이: Rexcool RV1 600

### 메모리 용량 부족

- 이전에 4GB * 2 메모리 사용
- proxmox를 사용하기에는 메모리 용량이 너무 부족한 것 같아 8GB * 2로 변경

### 파워 서플라이 문제

- 현재 해당 PC의 최대 전력은 대략 200W 정도로 최소 300-400W 이상의 파워가 필요
  - Rexcool RV1 600의 실제 출력은 250W 정도로 PC에 대해 여유가 매우 부족
- 해당 파워는 저품질 파워로써 위험 높음
- 파워 서플라이가 오래되어 노후화, 매우 위험
- 따라서 현재 사용중인 파워를 폐기한 후 교체하기로 함

## 홈서버 구성 계획

- LG gram 2019는 ubuntu OS 환경으로 구성
  - ubuntu의 gnome GUI 환경을 커스텀해보고 싶음
  - windows wsl2에서 리눅스 커널에 대한 제약 발생, 해당 ubuntu에서 제약을 극복하는 환경으로써 사용
  - 별도의 VM을 띄울 생각이 없으며 ubuntu OS만을 사용하고 싶어 proxmox 대신 ubuntu 설치
  - 나중에 Arch linux 설치, 구성해보고 싶음
- desktop2
  - desktop1에 비해 사양이 좋은 최신 PC
  - Proxmox OS 위에 Truenas VM을 띄워 nas를 구성할 계획
  - 가능하다면 Windows VM을 띄우고 moonlight, sunshine을 통해 원격 GUI 접속 환경을 구성해보고 싶음
