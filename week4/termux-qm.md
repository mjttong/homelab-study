# week4: termux, qm command

## termux 사용

- 갤럭시 탭에 tailscale을 설치하여 proxmox GUI로 작업을 진행, 모바일 환경에서 Proxmox GUI 작업이 너무 불편
- 안드로이드 환경에서 SSH 접속 및 CLI 작업이 가능한 터미널 환경 필요
- 안드로이드에서 리눅스 터미널 환경을 제공하는 termux를 설치하게 됨

### termux 설치 및 설정

- github에서 ternum 설치
  - <https://github.com/termux/termux-app>
- termux 설정 진행

```shell
# ssh, vim 설치
pkg install openssh vim -y

# storage 권한 획득
termux-setup-storage

# ssh 키 가져오기
curl https://github.com/mjttong.keys >> ~/.ssh/authorized_keys
```

## qm 명령어

- qm 명령어를 통해 VM 관리 및 접근 가능

```shell
# VM 목록 확인
qm list

# VM 시작 및 종료
qm start [VM_ID]
qm shutdown [VM_ID]

# VM 터미널 접근
qm terminal [VM_ID]
```

### `qm terminal`: 직렬 포트 접속

- `qm terminal [VM_ID]` 명령으로 직렬 포트를 통해 VM에 접속 가능
- VM 하드웨어에서 직렬 포트 추가, `/etc/default/grub` 설정 후 해당 명령으로 접속 가능

```shell
# /etc/default/grub:
GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0,115200n8"
```

```bash
sudo update-grub
sudo reboot
```

- 해당 직렬 포트 접속은 비상 시 복구용으로만 권장되는 방식
  - 매우 느린 속도, 낮은 보안성 등의 여러 문제 존재
- 일반적인 환경에서는 굳이 직렬포트 접속을 설정하지 않음

### qemu guest agent

- qemu guest agent를 통해 VM 내부 정보(IP, file system, memory 등 지표)를 쉽게 파악 가능
- VM에서 QEMU Guest agent 옵션이 활성화되어 있어야 하고, qemu-guest-agent 데몬이 실행중이어야 함
