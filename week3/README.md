### 목표
- ssh 프로토콜에 대한 정확한 이해
  - AWS에서 EC2에 접근하기 위해 ssh 연결을 많이 사용했다. 그러나 ssh 연결이 무엇인지 정확히 파악하고 사용하지 못했다.
  - ssh가 어떤 방식으로 작동하는지, 어떤 config 옵션이 사용 가능한지 파악한다.
- github actions runner의 이해
  - github-hosted 대신 self-hosted runner를 사용해보면서 가상 러너에 대해 이해한다.

## SSH

### 정의

- 원격 접속을 위해 사용함
    - 데이터 암호화 전송(Secure Shell) → 보안 보장
- 클라이언트 - 서버 모델로 구성
- 대칭 키, 비대칭 키 암호화 방식 사용
    - 비대칭 키 방식 → 사용자 인증
        - 클라이언트가 키페어 생성 및 서버에게 공개 키 전달
        - 원격 접속 시 서버가 랜덤 값 생성, 공개 키로 암호화해 클라이언트에게 전달
        - 클라이언트는 받은 값을 비밀 키로 복호화해 서버에게 전달
        - 서버는 복호화된 값을 받아 생성한 값과 비교
        - 값이 일치하면 인증된 사용자로 판단 → 접속 허용
    - 대칭 키 방식 → 데이터 암호화 및 복호화
        - 원격 접속 연결 완료 후 대칭 키를 사용해 데이터 암호화 및 복호화

### ForwardAgent

- ssh-agent를 포워딩 가능하게 함
    - ssh-agent는 개인 키를 캐싱
    - ssh-agent는 캐싱된 개인 키로 서명해 인증을 도움
- 클라이언트 → 중간 서버 → 최종 서버 접속
    - 클라이언트는 비밀 키, 공개 키 소유
    - 중간 서버, 최종 서버는 공개 키만 소유
    - ForwardAgent 옵션 활성화
        - 클라이언트 → 중간 서버로 ssh 접속
        - 클라이언트의 ssh-agent를 중간 서버가 사용 가능(Fowarding)
        - ssh-agent가 클라이언트에서 저장한 비밀 키를 캐싱해뒀기 때문에 중간 서버에서 비밀 키가 없더라도 최종 서버에 인증 가능

## SSH 연결

### SSH 설정
- 윈도우 → Proxmox 내 우분투 VM → github 접속

- 윈도우 설정
    - ssh-agent 서비스 실행 여부 확인
        - 서비스에서 OpenSSH Authentication Agent의 시작 유형을 자동으로 변경
        - 작업관리자의 서비스에서 ssh-agent 실행
    - ssh 키 생성 및 ssh-agent 등록
        
        ```bash
        ssh-keygen
        ssh-add [프라이빗 키 경로]
        ```
        
    - .ssh 폴더에 config 파일 생성 및 아래의 설정 추가
        
        ```bash
        Host ubuntu
        HostName [ubuntu ip]
        IdentityFile [ssh private key path]
        ForwardAgent yes
        ```
        
- github 설정
    - 윈도우에서 생성한 공개 키를 등록

- 우분투 설정
    - ssh 서버 설치
        
        ```bash
        sudo apt update
        sudo apt install openssh-server
        ```
        
    - 22번 포트 개방
        
        ```bash
        sudo ufw allow ssh
        ```
        
    - 윈도우에서 생성한 공개 키를 .ssh에 저장
        
        ```bash
        curl https://github.com/[사용자].keys >> [key]
        ```

### ProxyJump를 통한 연결
- 같은 네트워크 SSH 연결
  - 윈도우와 우분투가 같은 공유기 내 무선 + 유선 랜 연결
  - 우분투의 private IP로 ssh 연결 성공

- 다른 네트워크 SSH 연결
    - 윈도우와 우분투가 다른 공유기 연결
    - 이 환경에서 우분투의 Private IP로 ssh를 보내면 연결 불가 → 다른 랜이기 때문
    - 우분투의 공인 IP를 확인해 공인 IP를 통한 ssh 연결 시도
        
        ```bash
        curl ipconfig.me
        ```
        
        - 연결에 실패
        - 이 IP는 우분투의 공인 IP가 아니라 공유기의 공인 IP여서 SSH 연결이 안되는 것으로 파악된다.

- ProxyJump 사용
    - 윈도우 → Proxmox(중계 서버) → 우분투로 접속
        
        ```bash
        ssh -J [username]@[tailscale IP] [username]@[ubuntu IP]
        ```
        
    - 더 쉽게 접속하기 위해 ssh config 파일을 다음과 같이 작성했다.
        
        ```bash
        Host proxmox
        	HostName [tailscale IP]
        	User [username]
        
        Host ubuntu
        	HostName [ubuntu IP]
        	IdentityFile [ssh private key path]
        	User [username]
        	ProxyJump proxmox
        	ForwardAgent yes
        ```

    - 아래의 명령어를 사용해 간편하게 ssh 연결이 가능하다.
        ```bash
        ssh ubuntu
        ```
        
    - ssh 접속 성공!

    ![Image](https://github.com/user-attachments/assets/423e5fd8-e905-419b-aec9-50b4d1d31069)

    - 접속하는 우분투의 IP가 변화할 때마다 config 설정을 변경해야 해 우분투에서 고정 IP 설정이 필요할 것 같다.

---

## Github actions

- self-hosted runner
    - github-hosted runner 외에도 직접 개인 서버를 사용해서 github actions를 실행 가능
    - github hosted runner → ubuntu-latest 사양
        
        
        | CPU | RAM | SSD | 아키텍처 |
        | --- | --- | --- | --- |
        | 4 | 16GB | 14GB | X64 |

### self-hosted runner 실습

- github actions를 실행할 리포지토리에서 self-hosted runner 추가
- 우분투 VM에서 github에서 제공하는 코드로 runner 다운로드
- self-hosted runner 실행
    
    ```bash
    sudo ./svc.sh install
    sudo ./svc.sh start
    ```
    
    ![Image](https://github.com/user-attachments/assets/7eedd36b-3d41-4dbf-855f-ca85352884b5)
    
    - 실행 시 idle 상태로 나타난다.
- actions 트리거
    
    ![Image](https://github.com/user-attachments/assets/c7746e3b-756a-4f6e-9854-e641c40d4e46)
    
    - 실행에 성공했다!