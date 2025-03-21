### Network

- Bridge
    - VM이 독립적인 IP로 동작
- Host-Only
    - 외부 네트워크와 격리된 내부 폐쇄망 구성
- NAT
    - 호스트가 NAT 역할
    - VM이 내부 IP 주소 사용
    - 모든 VM에 같은 IP 할당

|  | Bridge | Host-Only | NAT |
| --- | --- | --- | --- |
| 호스트 - VM | ✅ | ✅ | 🔺 |
| VM - VM | ✅ | ✅ | ❌ |
| 인터넷 | ✅ | ❌ | ✅ |

<br>


### DHCP

- DHCP(Dynamic Host Configuration Protocol)
    - 네임 서버 주소, IP 주소, 게이트웨이 주소를 자동 할당
    - 영구적인 할당이 아닌 임대 개념의 할당
        - dhclient.leases 파일에 임대받은 IP 대역이 기록되어 있음
    - DHCP 서버 + 클라이언트로 구성
    - 라우터(공유기) 내에 DHCP 서버 내장 - 인터넷을 사용하는 기계에 DHCP 클라이언트 내장
- 구축 과정
    - 우분투 OS를 사용하는 VM 2대를 사용하여 각각 DHCP 서버, 클라이언트로 구축한다.
    - DHCP 서버는 DHCP 패키지(isc-dhcp-server) 설치 및 설정, 고정 IP 할당이 필요하다.
    
    <details>
    <summary>구축</summary>
    <div markdown="1">
    
    - DHCP 서버
        - DHCP 패키지 설치
            
            ```bash
            sudo apt-get update
            sudo apt-get install isc-dhcp-server
            ```
            
        - 고정 IP 할당 필요
            
            ```bash
            vim /etc/netplan/00-installer-config.yaml
            ```
            
            ```yaml
            network:
              version: 2
              ethernets:
                ens18:
                  addresses: [192.168.1.20/24]
                  routes:
            	      - to: default
            	        via: 192.168.1.1
                  nameservers:
                	  addresses: [8.8.8.8, 8.8.4.4]
            ```
            
            ```bash
            sudo netplan apply
            ```
            
        - DHCP 설정
            - dhcpd.conf 설정
                
                ```bash
                sudo vim /etc/dhcp/dhcpd.conf
                ```
                
                ```bash
                # option domain-name 부분 주석 처리
                
                subnet 192.168.0.0 netmask 255.255.255.0 {
                  range 192.168.0.2 192.168.0.254;
                  option routers 192.168.0.1;
                  option subnet-mask 255.255.255.0;
                }
                ```
                
            - NIC 설정
                
                ```bash
                vim /etc/default/isc-dhcp-server
                ```
                
                ```bash
                INTERFACESv4="ens18"
                INTERFACESv6="ens18"
                ```
                
        - DHCP 서버 재시작
            
            ```bash
            sudo systemctl start isc-dhcp-server
            ```
            
        
    - DHCP 클라이언트
        - dchp 서버로부터 IP를 할당받도록 설정
            
            ```bash
            vim /etc/netplan/00-installer-config.yaml
            ```
            
            ```yaml
            network:
              version: 2
              ethernets:
                ens18:
                  dhcp4: true
            ```
            
            ```bash
            sudo netplan apply
            ```
            
        - dhcp 서버로부터 IP 갱신
            
            ```bash
            sudo dhclient -r ens18
            sudo dhclient ens18
            ```
            

    - 클라이언트가 새로운 IP 값이 아닌 기존의 IP 값을 계속 받아옴
        - DHCP 리스 파일을 삭제 → 새로운 IP 값을 받아옴
            
            ```bash
            sudo rm /var/lib/dhcp/dhclient.leases
            ```
    </div>
    </details>