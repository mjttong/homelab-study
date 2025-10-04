# basic settings

## BIOS 설정

- 빠른 부팅 비활성화
- Secure Boot 비활성화
  - ventoy를 통해 proxmox ISO 이미지를 부팅할 예정이기 때문에 Secure Boot를 잠시 비활성화해야 함
  - asus: 안전 부팅 키 삭제
  - msi: 비활성화
- 가상화 기술 관련 설정
  - asus: Advanced > Intel Virtualization Technology Enabled
- 부팅 우선순위를 ventoy가 설치된 USB를 가장 높게 설정

### Proxmox 기초 설정

- [유효한 구독 없음](https://svrforum.com/os/138940)

```bash
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak 
sed -i "s/\\tExt.Msg.show/void/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
```

- [유료 리포지토리 삭제](https://god-logger.tistory.com/136)
  - enterprise 리포지토리를 모두 비활성화
  - Proxmox 웹 GUI에서도 설정 가능

```bash
# 파일 : /etc/apt/sources.list.d/ceph.list
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list 

# 파일 : /etc/apt/sources.list
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" >> /etc/apt/sources.list
```

### Tailscale 설치

- tailscale console > Add device > generate install script
- 해당 스크립트로 tailscale 설치 진행

### SSH 설정

- `ssh-copy-id` 또는 직접 복사해서 proxmox의 `~/.ssh/authorized_keys`로 ssh 접속에 사용할 공개 키 전달
- `/etc/ssh/sshd_config`를 통한 ssh 보안 설정
  - `PasswordAuthentication no`, `PermitRootLogin no` 설정

```shell
# /etc/hosts.allow
# 해당 IP 대역은 tailscale IP 대역인 100.64.0.0/10이다.
sshd: 100.64.0.0/255.192.0.0

# /etc/hosts.deny
sshd: ALL
```

- tcp wrapper를 통해 접근 제어 설정도 가능

### IP 및 호스트 이름 설정

- 호스트 이름 설정
  - `/etc/hosts` 수정
    - Proxmox Web UI > 해당 노드 > 호스트
  - `/etc/hostname` 수정
- IP 설정
  - `/etc/network/interfaces`
  - Proxmox Web UI > 해당 노드 > 네트워크 설정
  - 설정 후 `systemctl restart networking`

### `apt upgrade` 는 Proxmox에서 사용하면 안된다?

- [reddit: Stop using or suggesting to others to use apt upgrade](https://www.reddit.com/r/Proxmox/comments/18c3dah/psa_stop_using_or_suggesting_to_others_to_use_apt/)
- `apt upgrade` 대신 `apt full-upgrade` (`apt dist-upgrade`) 사용
  - `apt upgrade` 는 설치된 패키지의 업그레이드만 진행
    - 새로운 패키지 설치 및 제거를 진행하지 않음
    - 의존성 변경이 필요한 패키지는 업그레이드하지 않으며 변경을 요구하지 않는 패키지만 업그레이드
  - `apt full-upgrade` 는 의존성 변경이 필요한 패키지도 업그레이드
    - 새로운 의존성을 설치하거나 충돌하는 패키지를 제거 가능
- Proxmox는 단순한 Debian 환경이 아니며, 복잡한 의존성 및 자체 패키지가 얽힌 시스템
  - `apt upgrade` 만 진행할 경우, 의존성 변화가 필요한 패키지가 업데이트되지 않을 수 있으며, 시스템이 부분적으로만 최신 상태가 됨
  - 따라서 Proxmox 기능이 정상 동작하지 않을 수 있음
