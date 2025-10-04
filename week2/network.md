# Week 2: Proxmox Network

## Network mode

- Bridge
  - VM이 호스트와 동일한 네트워크에 연결
  - `vmbr0`과 같은 가상 브릿지 생성 및 해당 브릿지에 VM이 연결
- Host-Only
  - 외부 네트워크와 격리된 내부 폐쇄망 구성
- NAT
  - 호스트가 NAT 라우터 역할 수행
  - VM이 내부 사설망의 사설 IP 할당
  - 외부 인터넷에서 내부 VM으로 접근하기 위해서는 추가적인 설정 필수

|             | Bridge | Host-Only | NAT |
| ----------- | ------ | --------- | --- |
| 호스트 - VM | ✅ | ✅ | ✅ |
| VM - VM     | ✅ | ✅ | ✅ |
| 인터넷      | ✅ | ❌ | ✅ |

## ubuntu netplan

- ubuntu는 netplan을 통해 편리하게 네트워크 설정 가능
- `/etc/netplan/` 내의 yaml 파일 수정 및 `sudo netplan apply`로 적용 가능

```yaml
network:
  version: 2
  ethernets:
    eth0:                   # eth0 NIC에 대해 설정
      dhcp4: no             # DCHP 할당 안함
      addresses:            # 고정 IP 할당
        - 192.168.1.10/24
      routes:               # 라우팅 경로 설정
        - to: default
          via: 192.168.1.1
      nameservers:          # DNS 서버 설정
        addresses:
          - 8.8.8.8
```

## DHCP

- Dynamic Host Configuration Protocol
- 네임 서버 주소, IP 주소, 게이트웨이 주소를 자동 할당
- server-client로 구성되어 dhcp server가 dhcp 클라이언트에게 할당
- 영구적인 할당이 아닌 임대 개념의 할당
  - dhclient.leases 파일에 임대받은 IP 대역이 기록되어 있음
  - 임대기간 종료 전 갱신 필요
- 라우터(공유기) 내에 DHCP 서버 내장, 인터넷을 사용하는 기기에 DHCP 클라이언트 내장
