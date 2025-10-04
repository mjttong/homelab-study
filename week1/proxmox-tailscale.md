# Week 1: Proxmox, Tailscale

## 배경

- 목적
  - AWS를 사용하면서 직접 온프레미스 환경을 경험하고 싶어졌다.
  - 온프레미스 환경 경험 + 간단한 웹서버 구축을 목표로 진행한다.
- 환경
  - LG gram 2019 노트북 사용
  - Proxmox 설치 및 우분투 VM 생성

## Proxmox 사용하기

- Debian 기반의 오픈소스 하이퍼바이저 OS
- KVM 가상화, LXC 컨테이너를 모두 지원

### Hypervisor

- VM의 생성 및 관리를 맡는 가상화 소프트웨어
  - 물리적인 리소스를 논리적인 리소스로 구성 및 분할하는 가상화 기능을 지원
  - 한 대의 물리적 하드웨어 서버를 여러 대의 가상 서버(VM)로 분할하는 서버 가상화 지원
- Hypervisor 유형
  - 1형: 호스트 시스템 위에 OS 없이 바로 실행(Proxmox)
  - 2형: 호스트 OS 위에서 실행(VMware, VirtualBox)

### Proxmox OS 환경 구성

1. Proxmox iso 파일 다운로드
2. 부팅 가능한 USB 드라이브 만들기
    - USB 3.0에 ventoy 또는 balenaEtcher 설치
    - balenaEtcher는 하나의 OS에 대해 ISO 이미지를 구워 부팅 가능한 USB로 생성 가능
    - ventoy는 여러 ISO 이미지를 USB에 복사한 후 USB 부팅 시 원하는 ISO 선택 및 부팅 가능
3. PC에 USB를 연결한 후 BIOS에 진입해 부팅 우선순위를 USB 부팅으로 변경 후 Proxmox 설치

### Windows OS 멀티 부팅 고려

- 한 PC에서 파티션을 분할하고, 각각의 파티션에 Windows OS와 Proxmox OS를 설치해 멀티 부팅 환경을 구성하기 위해 시도
  - Proxmox 설치 과정에서 "기존의 모든 파티션과 데이터 손실" 문구 확인
  - 설치 과정에서 파티션 경계 삭제, 윈도우 OS, 데이터 삭제로 인해 멀티 부팅 구성 실패
- BIOS 환경에는 Windows Boot Manager가 남아있음
  - 부트로더는 파티션 경계 삭제, OS 및 데이터 삭제와는 무관하게 BIOS 영역에 존재
- 하이퍼바이저와 OS를 멀티 부팅으로 사용하는 발상은 지나치게 비효율적
  - 하이퍼바이저 위에 윈도우 VM을 띄우는 것이 더 적절한 방법
  - 이 과정을 통해 하이퍼바이저와 OS의 차이점, 멀티 부팅 환경의 적절한 구성, VM 등에 대해 고민해보고 제대로 이해할 수 있었기 때문에 해당 실패를 오히려 긍정적으로 생각

## Tailscale 사용하기

### VPN

- Virtual Private Network
- 인터넷(공중망) 상에서 구축되는 논리적인 전용망
- 가상의 터널을 사용해 사설망과 같은 환경을 구축

### VPN 유형

- Client-to-Site
  - 사용자 → VPN Client → ISP → VPN Server → Internet
  - 사용자가 자신의 PC에서 VPN 클라이언트를 실행
  - 클라이언트가 ISP를 통해 VPN 서버까지 암호화된 터널 생성
  - ISP나 중간 경로에서 해당 트래픽 확인 불가능
  - VPN 서버는 사용자가 전송한 트래픽을 복호화해 인터넷 상으로 트래픽 전달
- Site-to-Site
  - 두 네트워크를 안전하게 연결하기 위한 VPN
  - 각 네트워크에 VPN 게이트웨이 설치 및 암호화된 터널 형성
  - 사용자가 별도의 VPN 클라이언트 없이 네트워크 자원에 접근
- m2m(machine to machine)
  - 머신 간 안전하게 통신하기 위한 VPN
  - 각 머신에 VPN 소프트웨어 설치 및 암호화된 터널 형성

### Tailscale

- WireGuard 프로토콜 기반 m2m VPN
- Zero Trust 보안 사용
  - 네트워크 내부/외부와 무관하게 항상 신뢰하지 않음
- 한 계정을 기반으로 tailscale이 설치된 모든 기기를 자동으로 하나의 사설 네트워크(tailnet)으로 묶음
- 각 기기에 사설 IP 할당, 기기 간 mash 구조 형성
