# Week 4: WOL

## 컴퓨터 원격 부팅

- WOL(Wake On Lan)
  - 유선 랜을 통해 매직 패킷을 전송해 전원을 켬
  - PC가 유선 랜에 연결되어 있어야만 사용 가능
- 스마트 플러그
  - 외부에서 스마트 플러그 전원을 켜서 컴퓨터 전원을 켬
  - 스마트 플러그를 구매해야 가능
- 이 중 WOL을 사용해 진행
  - Proxmox는 네트워크 구성 시 유선 랜 연결을 권장, 따라서 해당 PC가 유선 랜에 연결되어 있어 WOL을 사용하기 적절했음

## WOL 원리

- 유선 랜을 통해 매직 패킷을 브로드캐스트로 전송
  - 매직 패킷은 FF:FF:FF:FF:FF:FF + 대상의 MAC 주소로 구성
  - 일반적으로 UDP 사용, TCP는 연결 지향으로 데이터 전송 전에 연결이 미리 설정되어야 해 WOL에서는 부적절
- 대상 NIC는 매직 패킷을 수신, PC에게 깨어나라는 신호 전송
- WOL을 사용하기 위해서는 NIC의 일부가 계속 켜져 있어야 함
  - BIOS에서 WOL을 지원해야 NIC의 일부가 계속 켜져 있도록 설정 가능

## WOL 설정

### BIOS 설정

- 물리 서버의 BIOS 접근 및 `Wake on LAN`, `Wake from PCIe` 등의 옵션 활성화

### OS 설정

```shell
sudo apt install ethtool
```

- 패키지 설치

```shell
ethtool [interface] | grep Wake-on
ethtool -s [interface] wol g
```

- wake on 옵션 활성화
- `Wake-on: g` 로 되어 있다면 매직 패킷 수신 가능

```shell
# /etc/network/interface:
iface [interface] inet manual
  post-up /usr/sbin/ethtool -s [interface] wol g
```

- 재부팅 후에도 WOL이 유지되도록 설정

### 포트 포워딩

- 외부에서 내부망에 매직 패킷을 전송하기 위해서는 포트 포워딩 필요
- 포트 포워딩 설정
  - IP 주소: 브로드캐스트 IP, WOL로 깨울 장치의 IP
  - 프로토콜: UDP
  - 외부 포트번호: 임의의 번호(5자리, 랜덤한 값)
  - 내부 포트번호: 9번
- 단, 공유기 포트 포워딩은 가정용으로 사용하는 내부망을 외부에 노출하는 것과 같기 때문에 보안상 부적절
  - 웬만하면 사용하지 않는 것이 좋으며, tailscale VPN 내부망을 최대한 활용하는 것이 적절
  - 단, tailscale은 L3 계층에서 작동하므로 L2 계층의 WOL 매직 패킷을 전송하려면 추가적인 중계 장치 필요
  - <https://tailscale.com/blog/wake-on-lan-tailscale-upsnap>
- 이후 스마트폰의 Wake On Lan 어플 설치 및 설정을 통해 WOL 가능
- 만약 ipTIME 공유기를 사용 중이라면 ipTIME 공유기의 WOL 기능을 설정 후 ipTIME 모바일 앱으로 WOL 가능

## WOL 미지원

- BIOS에 WOL 관련 설정 옵션이 존재하지 않음
- suspend 상태에서만 WOL 가능, hibernate 또는 전원 종료 시 WOL 불가능

### ACPI level: suspend states

- S0: 전원 켜짐, 작동 상태
- S1: power on suspend
- S2: cpu power off, 현재 잘 사용하지 않음
- S3: suspend to ram(suspend), ram을 제외한 대부분 구성 요소의 전원이 꺼짐
- S4: suspend to disk(hibernate), 시스템 상태를 디스크에 저장한 후 전원 차단
- S5: 전원 꺼짐(soft off)

### WOL 미지원 분석

- S3에서는 WOL이 지원
  - RAM에 전원이 공급되기 때문에, NIC도 대기 전력을 받아 매직 패킷 감지
- S4, S5에서 NIC에 대기 전력이 공급되려면 메인보드가 WOL을 지원해야 함
  - 그러나 BIOS에 해당 옵션이 없는 것을 보아 메인보드 자체가 S4, S5 WOL을 지원하지 않는 것으로 파악
