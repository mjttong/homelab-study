## To-do

- 안드로이드에서 서버 원격 접속 설정
- WOL 설정
- DNS

---

## 안드로이드에서 서버 원격 접속 설정

### 목표

- 평소 갤럭시탭을 자주 들고 다니는데… 홈서버 작업을 하기에는 너무 불편하다.
    - Tailscale을 설치하고 Proxmox GUI로 접근하면 되긴 하지만 너무 불편하다.
- ssh 연결로 작업할 수 있도록 터미널이 필요하다고 생각했다.

### 진행

1. termux, tailscale 어플 설치
    - termux는 깃허브에서 다운받았다.
2. termux 설정
    
    ```bash
    # ssh 설치
    pkg install openssh
    
    # storage 권한 획득
    termux-setup-storage
    
    # ssh 키 가져오기
    curl https://github.com/mjttong.keys >> id.pub
    
    # vim 설치
    pkg install vim -y
    ```
    

### qm 명령어

- qm 명령어를 사용하면 Proxmox GUI 콘솔에 접속하지 않고 VM을 관리 및 접근할 수 있다.
    
    ```bash
    # VM 목록 확인
    qm list
    
    # VM 시작 및 종료
    qm start [VM_ID]
    qm shutdown [VM_ID]
    
    # VM 터미널 접근
    qm terminal [VM_ID]
    ```
    

- `qm terminal [VM_ID]`
    - ssh 연결 말고 VM에 접근하는 방법은 없을까 고민하다 qm terminal로 접근 가능하다는 것을 알게 되었다.
    - qm 명령어를 사용해 터미널에 접근하려면 설정이 필요하다.
        1. VM의 하드웨어에서 직렬 포트 추가
        2. VM 내에 접근해 `/etc/default/grub` 설정 수정
            
            ```bash
            sudo vim /etc/default/grub
            ```
            
            ```bash
            GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0, 115200n8"
            GRUB_TERMINAL="serial console"
            ```
            
            ```bash
            sudo update-grub
            sudo reboot
            ```
            

- qemu guest agent
    - qemu guest agent를 사용하면 VM  내부 정보를 쉽게 파악할 수 있다.
    - 나는 IP를 쉽게 확인하려는 용도로 설치했다.
        
        ```bash
        # proxmox에서 진행
        qm set [VM_ID] -agent enabled=1
        ```
        
        - Proxmox GUI에서 VM의 옵션에서도 QEMU Guest Agent를 활성화할 수 있다.
        
        ```bash
        # VM 내부에서 진행
        sudo apt update
        sudo apt install qemu-guest-agent
        
        sudo systemctl start qemu-guest-agent
        sudo systemctl enable qemu-guest-agent
        sudo systemctl status qemu-guest-agent
        ```
        
        - qemu guest agent가 제대로 실행되지 않는다면, VM을 완전히 종료한 후 다시 실행하자.
        
        ```bash
        qm guest cmd [VM_ID] network-get-interfaces
        ```
        
        - 이후 Proxmox shell에서 위의 명령어를 사용해 IP를 확인 가능하다.
        - 또한 Proxmox GUI의 VM 요약 정보에서도 IP를 확인할 수 있다.

## WOL 설정

### 목표

- 외출 전에 노트북을 키고 → 이동 시간동안 노트북 방치 → 이동 후 노트북 접속
- 이동 시간동안 나가는 전기가 너무 아깝다…
- 내가 원할 때 컴퓨터를 끄고 킬 수는 없을까?

- 컴퓨터를 원격으로 부팅할 때 주로 사용하는 방법은 2가지가 있다.
    - WOL(Wake On Lan)
        - 유선 랜을 통해 매직 패킷을 전송해 PC를 킨다.
        - 집에서 노트북에 유선 랜을 꽂아 사용하고 있어 충분히 사용 가능해보인다.
    - IoT 플러그
        - 플러그 구매가 필요하다.
- WOL을 사용해 진행해보기로 했다.

### WOL 원리

1. 대상 MAC 주소가 포함되는 매직 패킷이 브로드캐스트 IP를 사용해 네트워크로 전송된다.
    - 매직 패킷은 FF:FF:FF:FF:FF:FF + 대상의 MAC 주소로 구성된다.
    - 일반적으로 UDP를 사용한다.
    - TCP는 연결 지향으로 데이터 전송 전에 연결이 미리 설정되어야 해 WOL에서는 부적절하다.
2. 대상 NIC는 매직 패킷을 수신한 후 컴퓨터에게 깨어나라는 신호를 보낸다.
- WOL이 작동하려면
    - NIC의 일부가 계속 켜져 있어야 한다.
    - BIOS에서 WOL을 지원해야 한다.

### 진행

1. BIOS 설정
    - 물리서버의 BIOS 접근 및 설정 진행
    - 그러나 내 PC에서는 BIOS에서 WOL 관련 설정이 보이지 않아 설정하지 않았다.
2. 공유기 설정
    - WOL에서 서버의 MAC 주소 등록
    - DDNS 설정
    - 포트 포워딩 설정
        - IP 주소: WOL로 깨울 장치
        - 프로토콜: UDP
        - 외부 포트번호: 임의의 번호
        - 내부 포트번호: 9번
3. 스마트폰 어플 설정
    - Wake On Lan 어플 설치
    - 홈서버 등록
        - MAC 주소: WOL로 깨울 장치 MAC 주소
        - 브로드캐스트 주소 등록
        - 기기 IP: 공유기의 DDNS → 공유기가 WOL 패킷을 받아 브로드캐스트
        - 포트: 포트 포워딩에서 설정한 외부 포트번호
4. Proxmox 설정
    - 패키지 설치
        
        ```bash
        sudo apt install net-tools ethtool wakeonlan
        ```
        
    - wake on 옵션 활성화 확인
        
        ```bash
        ethtool eth0 | grep Wake-on
        ```
        
        - `Wake-on: g` 로 되어 있다면 매직 패킷을 전달받을 수 있는 상태이다.
    - wol 옵션 활성화
        
        ```bash
        ethtool -s eth0 wol g
        ```
        
    - wol 옵션 활성화 유지 설정 및 MAC 주소 고정
        
        ```bash
        iface eth0 inet manual
        				hwaddress ether [MAC address]
                post-up /sbin/ethtool -s eth0 wol g
        ```
        
    - `/etc/systemd/system/wol.service` 파일 생성 및 활성화
        
        ```bash
        vim /etc/systemd/system/wol.service
        ```
        
        ```bash
        [Unit]
        Description=Configure Wake-up on LAN
         
        [Service]
        Type=oneshot
        ExecStart=/sbin/ethtool -s enp5s0 wol g
         
        [Install]
        WantedBy=basic.target
        ```
        
        ```bash
        systemctl start wol.service
        systemctl enable wol.service
        systemctl is-enabled wol.service
        systemctl daemon-reload
        ```
        

### 문제 발생

- suspend 상태에서만 WOL 가능
    
    ```bash
    systemctl suspend
    ```
    
    - 런레벨 0이나 hibernate 상태에서는 WOL이 작동하지 않는다.
    - BIOS가 해당 레벨에서 WOL을 지원하지 않는 것으로 추정된다.

- 외부 랜에서는 WOL 실패
    - 내부 랜에서 WOL 작동 과정
        - WOL 어플이 매직 패킷을 내부 랜으로 전송
        - 어플은 브로드캐스트로 직접 매직 패킷 전송 가능, 내부 랜에서 전송된 브로드캐스트 패킷은 차단되지 않음
        - 이후 매직 패킷의 MAC 주소에 해당하는 기기가 Wake
    - 외부 랜에서 WOL 작동 과정
        - 외부에서 들어오는 브로드캐스트 패킷은 차단됨
        - 공유기는 기본적으로 외부에서 들어오는 특정 포트의 패킷이 어디로 가야 하는지 모름 → 패킷을 폐기
        - 포트 포워딩 설정 시 외부의 특정 포트에서 들어온 패킷을 특정 내부 포트 및 IP로 전달 → 이후 내부 네트워크에서 패킷이 처리 가능해 브로드캐스트됨
    - 결론적으로, 외부 랜에서는 포트 포워딩이 필요하다.
        - 포트포워딩 설정 및 WOL 어플 설정 변경 이후 외부 랜에서도 WOL이 성공했다!

- 이제 원격으로 부팅할 수 있고, 안드로이드에서 터미널 접근 및 ssh 연결이 가능하다.
    - 다음 목표로는 GUI에 접근하지 않고 CLI 환경에서 VM 생성, OS 설치, 접근 등 모든 작업을 다 처리하고 싶다.

## DNS

### 개념

- DNS
    - 도메인 이름을 IP주소로 변환
    - 계층 구조
        - 루트 네임서버(.)
        - TLD 네임서버(com, go, org …)
        - 권한 네임서버
            - 특정 도메인의 IP 주소를 최종적으로 저장하고 있는 서버
        - 재귀적 네임서버
            - 클라이언트 - DNS 네임서버 사이 중개
            - DNS 캐시 서버
- DNS 질의 과정
    1. 내 PC의 DNS 캐시 확인
    2. 캐시가 없는 경우 브라우저가 재귀적 DNS 서버로 질의
    3. 재귀적 DNS 서버 내 캐시가 없는 경우 루트 DNS 서버에 질의
    4. 루트는 TLD DNS 서버의 IP를 return
    5. TLD DNS 서버에 질의 시 권한 DNS 서버의 IP를 return
    6. 권한 DNS 서버가 질의한 DNS의 IP 주소 반환 및 재귀적 DNS 서버가 이를 저장
    7. 브라우저에 IP 주소 전달
    

### DNS 구성 요소

- 존
    - dns 존은 논리적 영역으로 dns 간의 관계를 바탕으로 자유롭게 범위를 지정할 수 있다.
    - 예를 들어, mjttong.com이라는 하나의 도메인이 있다고 하자.
        - blog.mjttong.com, community.mjttong.com 등의 도메인은 mjttong.com 존 내에 속할 수도 있고, 별개의 존으로 관리될 수도 있다.
        - 각각의 하위 도메인은 상위 도메인과 논리적 관계에 따라 자유롭게 설정된다.
    - 존은 존 파일을 통해 관리된다.
- 레코드
    - A 레코드: 도메인을 IPv4로 매핑
    - AAAA 레코드: 도메인을 IPv6로 매핑
    - CNAME 레코드: 도메인 간 매핑
    - NS 레코드: 도메인 이름을 관리하는 네임 서버 제공
    - PRP 레코드: IP 주소에 대한 도메인 확인
    - SOA 레코드: DNS 영역에 대한 핵심적인 정보 저장 및 2차 네임서버 관련 매개변수 설정

## DNS 실습

### 실습 정리

- 네트워크 → Bridge 모드로 진행
- 우분투 데스크탑
    - 브라우저 역할 및 DNS 질의
    - 네임서버가 서버1(`192.68.1.11`)
- 우분투 서버1 `192.68.1.11`
    - Public DNS 질의는 8.8.8.8로 포워딩
    - Private DNS 질의는 서버 2로 포워딩
    - 네임서버가 `8.8.8.8` 및 서버2(`192.168.1.12`)
- 우분투 서버2 `192.168.1.12`
    - Private DNS 질의만 처리
    - 네임서버가 자기 자신(`192.168.1.12`)
    - nginx 웹서버 설치

- DNS 흐름
    - 데스크탑의 DNS 질의 → 서버 1로 질의
    - Public DNS
        - 서버 1에서 포워딩 → 8.8.8.8로 질의
        - 8.8.8.8이 응답
    - Private DNS
        - 서버 1에서 포워딩 → 서버 2로 질의
        - 서버 2에서 응답

### DNS 설정

- bind9 설정 파일
    - `/etc/bind/named.conf` 메인 설정 파일
    - `/etc/bind/named.conf.local` 로컬 DNS 설정
    - `/etc/bind/named.conf.options` 전역 옵션 설정
    - 설정 파일은 `named-checkconf` 로 검사할 수 있다.
- 존 파일 설정
    - `db.[도메인 이름]` 로 명명
    
    ```bash
    $TTL    86400
    @       IN      SOA     ns1.CloudClubDNS.com. admin.CloudClubDNS.com. (
                         2025033101         ; Serial
                               3600         ; Refresh
                               1800         ; Retry
                             604800         ; Expire
                              86400 )       ; Negative Cache TTL
    ;
    
    @        IN      NS      ns1.CloudClubDNS.com.
    
    ns1      IN      A       192.168.1.12
    www      IN      A       192.168.1.12
    ```
    
    - TTL, SOA 레코드, NS 레코드는 필수적으로 작성한다.
    - SOA
        
        ```bash
        	@       IN      SOA     [mname]. [rname] (
                                      2         ; Serial
                                 604800         ; Refresh
                                  86400         ; Retry
                                2419200         ; Expire
                                 604800 )       ; Negative Cache TTL
        ```
        
        - mname → 도메인 이름 설정
        - rname → 도메인 관리자의 email 주소를 도메인 형식으로 작성
            - abc@gmail.com → abc.gmail.com.
        - 매개변수 → 2차 DNS 서버의 갱신과 관련있는 값
            - 2차 DNS 서버는 1차 DNS 서버의 데이터를 복제하며, 고가용성을 위해 운영되는 서버이다.
            - Serial: 존 파일 변경 시 증가하는 값
            - Refresh: 1차 DNS 서버의 SOA를 조회하는 주기
            - Retry: 1차 DNS 서버에 재시도 요청을 보내는 주기 → refresh보다 짧게 설정
            - Expire: 1차 DNS 서버의 정보 갱신 실패 시 최대 정보 요청 시간
            - Negative Cashe TTL: DNS 서버에 정상적인 응답이 없는 경우, 응답을 캐시하는 시간 간격
    - `@` 는 루트 도메인(CloudClubDNS.com)을 표현한다.
    - 루트 도메인에 접두사를 붙여 서버 간 구별한다.
        - ns(name server), www(web server), mail(mail server) 등
    - 도메인의 끝이 `.` 으로 끝나면 절대 도메인, 아니면 상대 도메인이다. 상대 도메인에는 뒤에 루트 도메인이 붙어 해석된다.
    

### 우분투 데스크탑 설정

- netplan 설정
    
    ```bash
    sudo vim /etc/netplan/XX.yaml
    ```
    
    ```yaml
    network:
        ethernets:
            ens18:
                dhcp4: true
                nameservers:
                  addresses:
                    - 192.168.1.11
        version: 2
    ```
    
- `/etc/resolv.conf` 네임서버 등록
    
    ```bash
    sudo vim /etc/resolv.conf
    ```
    
    ```bash
    nameserver 192.168.1.11
    ```
    

### 우분투 서버1 `192.168.1.11`

- netplan 설정
    
    ```yaml
    network:
        ethernets:
            ens18:
                dhcp4: false
                addresses:
                  - 192.168.1.11/24
                routes:
                  - to: default
                    via: 192.168.1.1
                nameservers:
                  addresses:
                    - 192.168.1.12
                    - 8.8.8.8
        version: 2
    ```
    
- `/etc/resolv.conf` 네임서버 등록
    
    ```bash
    sudo vim /etc/resolv.conf
    ```
    
    ```bash
    nameserver 192.168.1.12
    nameserver 8.8.8.8
    ```
    
- bind9 설치
    
    ```bash
    sudo apt install bind9
    ```
    
- `named.conf.local`
    
    ```bash
    sudo vim /etc/bind/named.conf.local
    ```
    
    ```bash
    zone "dns1.com" {
        type forward;
        forward only;
        forwarders { 192.168.1.12; };
    };
    ```
    
- `named.conf.options`
    
    ```bash
    sudo vim /etc/bind/named.conf.options
    ```
    
    ```
    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { any; };
        listen-on { any; };
        listen-on-v6 { any; };
        forwarders { 8.8.8.8; };
        dnssec-validation no;
    };
    ```
    
- 파일 설정 검사
    
    ```bash
    sudo named-checkconf /etc/bind/named.conf.local
    sudo named-checkconf /etc/bind/named.conf.options
    
    sudo systemctl restart bind9
    ```
    

### 우분투 서버2 `192.168.1.12`

- netplan 설정
    
    ```yaml
    network:
        ethernets:
            ens18:
                dhcp4: false
                addresses:
                  - 192.168.1.12/24
                routes:
                  - to: default
                    via: 192.168.1.1
                nameservers:
                  addresses:
                    - 192.168.1.12
        version: 2
    ```
    
- bind9 설치
    
    ```bash
    sudo apt install bind9
    ```
    
- 존 파일 생성
    
    ```bash
    sudo vim /etc/bind/db.dns1.com
    ```
    
    ```bash
    $TTL    86400
    @       IN      SOA     ns1.dns1.com. admin.dns1.com. (
                         2025033101         ; Serial
                               3600         ; Refresh
                               1800         ; Retry
                             604800         ; Expire
                              86400 )       ; Negative Cache TTL
    ;
    
    @        IN      NS      ns1.dns1.com.
    
    ns1      IN      A       192.168.1.12
    www      IN      A       192.168.1.12
    ```
    
- `named.conf.local`
    
    ```bash
    sudo vim /etc/bind/named.conf.local
    ```
    
    ```bash
    zone "dns1.com" {
        type master;
        file "/etc/bind/db.dns1.com";
    };
    ```
    
- `named.conf.options`
    
    ```bash
    sudo vim /etc/bind/named.conf.options
    ```
    
    ```
    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { any; };
        listen-on { any; };
        listen-on-v6 { any; };
        dnssec-validation no;
    };
    ```
    
- 파일 설정 검사
    
    ```bash
    sudo named-checkzone dns1.com /etc/bind/db.dns1.com
    sudo named-checkconf /etc/bind/named.conf.local
    sudo named-checkconf /etc/bind/named.conf.options
    
    sudo rndc flush
    sudo systemctl restart bind9
    ```
    
- nginx 설치
    
    ```bash
    sudo apt install nginx
    ```
    

### DNS 질의

![Image](https://github.com/user-attachments/assets/8cb7b459-7bc7-4372-a33e-23bedd213fc8)

![Image](https://github.com/user-attachments/assets/5e6f5d81-0c5b-4a16-803a-c3d5f418855a)

- 우분투 데스크탑에서 Private DNS 질의를 보내 응답을 성공적으로 받았다.

![Image](https://github.com/user-attachments/assets/76c8ede3-653e-40b1-b8c5-0ad0faf0d0a4)

- Public DNS로 질의를 보내 응답을 성공적으로 받았다.

### 와이어샤크 사용하기

```bash
sudo tcpdump -i ens18 tcp -w UbuntuDesktop.pcap
```

```bash
nslookup www.dns1.com
nslookup www.google.com
```

- 윈도우에서 wireshark를 설치하고 우분투에서 캡쳐한 파일을 윈도우로 가져와 분석하려 했지만 분석이 전혀 안된다.

![Image](https://github.com/user-attachments/assets/0d05ef3c-2a54-4b3d-94e6-05de56eea705)

- 우분투 데스크탑에서 wireshark를 설치하고 분석하자.
    - wireshark에서 ssh 관련 패킷만 뜬다.
    - wireshark 관련 학습이 필요하다…