## Linux에서 PAM 모듈을 사용


### 💡 목표
사용자의 인증을 담당하는 모듈을 사용하여 비밀번호 8자리 이상으로 규제한다.

![image](https://github.com/user-attachments/assets/408daeb5-3738-480b-a602-e483c8b21c36)

<br/>

Clone 메뉴를 통해 가상 머신을 복제한다.
<img width="807" alt="스크린샷 2024-09-20 14 55 30" src="https://github.com/user-attachments/assets/614f52bd-bf80-4adb-9568-a4f7886e5778">     

### Bridged 모드

가상 머신이 네트워크에 직접 연결된 독립 장치처럼 작동하며, 가상 머신이 네트워크 서비스를 제공하거나 다른 장치와의 직접적인 통신이 필요할 때 선호되는 모드입니다.   

### 환경 변수 파일 수정

다음 명령어를 실행하여 네트워크 설정 파일을 수정합니다.

```bash
sudo vi /etc/netplan/00-installer-config.yml
```

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s1:
      dhcp4: true
    enp0s2:
      addresses:
        - 192.168.64.100/24
      routes:
        - to: default
          via: 192.168.64.1
      nameservers:
        addresses:
          - 8.8.8.8
      dhcp4: false
```

netplan 변경사항을 저장하여 반영한다.
```bash
sudo netplan apply
```   
   
❓**enp0s2.address 주소 설정 이유**  

로컬 PC의 네트워크 설정과 동일한 서브넷에 맞춘 조정이 필요하다. 또한, 할당되지 않은 IP를 부여해야 한다.
   
**고정 IP 할당한 결과**
![image](https://github.com/user-attachments/assets/cdc87a34-df84-4441-acec-c3b4f631d4e8)   

```bash
sudo vi /etc/security/pwquality.conf
```
```shell
minlen = 8
```