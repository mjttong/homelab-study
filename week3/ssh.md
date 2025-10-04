# Week 3: SSH

## 정의

- Secure Shell
- 원격 접속을 위해 사용하는 프로토콜로, client-server 구조로 구성

## 특징

- 원격 접속 시 챌린지 방식 사용 가능
  - 해당 방식에서는 비대칭 키를 통한 인증 진행
  - 클라이언트는 키페어를 소유, 해당 키페어의 공개키는 서버의 `authorized_keys`에 기록
  - 서버도 자체 키페어(호스트키)를 소유, 해당 키페어의 공개키는 클라이언트의 `known_hosts`에 기록
  - 클라이언트-서버는 원격 접속에서 위의 키페어를 통해 서로를 검증
- 원격 접속 시 패스워드 방식 사용 가능
  - 해당 방식에서는 서버의 사용자 계정 비밀번호를 통한 인증 진행
  - 무차별 대입 공격(brute-force attack)에 취약하기 때문에 권장되지 않음
- 원격 접속 후의 통신은 대칭키 방식으로 암호화되어 이루어짐

## ssh client `config`

- `~/.ssh/config` 파일을 통해 ssh 접속 정보를 미리 저장해두고 간편하게 별칭으로 접속 가능

```plain text
Host [alias]
  HostName [ip | domain]
  User [username]
  Port [port]
  IdentityFile [ssh private key path]
  ForwardAgent [yes | no]
  ProxyJump [alias]
```

### `ForwardAgent`

- ssh-agent는 개인 키를 메모리에 저장 후 인증에 사용
- `ForwardAgent` 활성화 시 클라이언트의 ssh-agent가 중간 서버로 포워딩되어 개인키를 중간 서버로 복사하지 않고도 중간 서버에서 최종 서버로 인증 가능
  - 개인키는 중간 서버에 저장되지 않고, 인증 요청만 클라이언트의 ssh-agent로 전달되어 서명 작업이 수행
- bastion host(클라이언트 > bastion host > 최종 서버)에서 사용

### `ProxyJump`

- 중간 서버를 거쳐 최종 서버에 접속하는 경우 사용
- 해당 옵션을 통해 중간 서버를 자동으로 경유, 여러 번의 수동 접속이 불필요함
- `ForwardAgent`와 함께 사용해 자동 접속 가능

## ssh 보안 설정

- `/etc/ssh/sshd_config` 설정 파일을 통한 ssh 보안 설정 중요
- `PasswordAuthentication no`로 패스워드 인증 사용 금지
- `PermitRootLogin no`로 루트계정 로그인 금지
- 그 외에도 포트 변경, 세션 관련 보안 설정 가능
- `fail2ban`을 통해 ssh 로그인 시도에 빈번하게 실패하는 IP 주소 차단

## github ssh keys

- github profile에서 SSH 공개 키 등록 가능
- 해당 키는 `github.com/[user].keys` 링크를 통해 조회 가능

![Image](/week3/ssh/ssh.png)
