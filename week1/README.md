### 홈서버를 구축해보자!

- 목적
    - AWS를 사용하면서 직접 온프레미스 환경을 경험하고 싶어졌다.
    - 온프레미스 환경 경험 + 간단한 웹서버 구축을 목표로 진행한다.
- 환경
    - 그램 2019 노트북 사용
    - Proxmox 설치 → 우분투 VM 생성

### Proxmox

- Hypervisor
    - Hypervisor는 VM의 생성 및 관리를 맡는 가상화 소프트웨어
    - 가상화: 물리적인 리소스를 가상 리소스로 분할해 사용
    - 서버 가상화 방식: 한 대의 물리적 하드웨어 서버 → 여러 대의 가상 서버(VM)로 분할
- Hypervisor 유형
    - 1형: 호스트 시스템 위에 OS 없이 바로 실행 → Proxmox
    - 2형: 호스트 OS 위에서 실행 → VMware, VirtualBox
- 구축하기
    1. Proxmox iso 파일 다운로드
    2. USB 드라이브 설정
        - 부팅 가능한 USB 드라이브 만들기
            - USB에 iso 파일을 넣고 rufus를 다운받아 이미지를 구움
        - USB에 Ventoy 설치 후 iso 파일을 넣어 사용하기
            - 여러 운영체제를 하나의 USB 내에서 선택 가능
    3. 이후 BIOS에 진입해 부팅 우선순위를 USB 부팅으로 변경 후 Proxmox 설치

- 윈도우 OS 및 데이터가 손실?
    - 문제 상황: 윈도우 OS를 유지한 상태에서 Proxmox OS를 설치하려 했는데, Proxmox 설치 후 윈도우가 실행되지 않는다.
    - 과정
        1. 윈도우에서 파티션을 156GB(윈도우용) + 100GB(Proxmox용)으로 나눔
        2. 부팅 우선순위를 USB 부팅으로 변경, ventoy가 적용된 USB에서 Proxmox iso 이미지를 통해 Proxmox 설치
        3. 이후 부팅 우선순위를 윈도우 부트 매니저로 바꿨는데도 불구하고 윈도우가 실행되지 않고 proxmox만 실행됨
    - 원인: proxmox 설치 과정에서 “기존의 모든 파티션과 데이터가 손실됨” 문구 확인
        - 설치 과정에서 파티션 경계 삭제 → 윈도우 os 및 데이터 삭제
        - 윈도우는 삭제되어도 BIOS에는 윈도우 부트 매니저가 남아있을 수 있음

### Tailscale

- VPN(Virtual Private Network, 가상 사설망)
    - 사설망 구축
        - 전용망 구축
        - 인터넷(공중망)을 사용해 사설망과 같은 환경 구축 = 가상 사설망
    - 인터넷(공중망) 상에서 구축되는 논리적인 전용망 → 가상의 터널 사용
- VPN 유형
    - Client-to-Site
        - 사용자가 VPN 클라이언트를 통해 VPN 서버에 연결 요청
        - 사용자 → VPN Client → ISP → VPN Server → Internet
            - 사용자 → ISP → VPN Server까지 터널링 생성
            - ISP는 트래픽 확인 불가
            - 사용자가 VPN 서버로 보낸 모든 트래픽이 암호화
            - VPN 서버는 사용자의 트래픽을 복호화해 인터넷 상으로 트래픽 전달
    - Site-to-Site
        - 두 네트워크 간 연결
    - m2m(machine to machine)
- Tailscale
    - WireGuard 프로토콜 기반 Mesh VPN
    - m2m VPN
    - Zero Trust 보안: 네트워크 내부/외부와 무관하게 항상 신뢰하지 않음
- Tailscale 사용
    - 홈서버 및 원격 접속할 노트북에 모두 Tailscale을 설치한다.
    - 설치 후 Tailscale에서 IP를 확인한다.
    - IP와 8006 포트를 통해 Proxmox 페이지로 진입한다.