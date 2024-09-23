# Linux 환경에서 Average Load Test

## 📃 개요
서버에 CPU 집약적인 작업과 I/O 대기 상태의 작업을 부여하여, __서버의 평균 부하(Average Load)와 CPU 사용률(CPU Usage)__ 간의 관계를 분석하고, 다양한 부하 조건에서 서버 성능과 안정성을 평가하는 작업을 진행하였습니다. 이를 통해 서버의 성능 한계, 안정성, 확장성을 파악하고, 병목 현상 및 장기 운영 시 발생할 수 있는 성능 저하 문제를 식별하는 데 중점을 두었습니다.


## ⚒️ 도구
시스템의 I/O 성능, CPU 사용률, 프로세스별 자원 사용을 실시간으로 모니터링하고 분석하여, 평균 부하 증가의 원인을 파악할 때 사용하는 도구입니다.

1. **iostat**
2. **mpstat**: 멀티 코어 CPU 분석
3. **pidstat**: 진행 중인 프로세스 분석


|              | **mpstat**                                    | **pidstat**                                   |
|--------------|----------------------------------------------|----------------------------------------------|
| **초점**     | 멀티코어 CPU 성능 분석                        | 개별 프로세스 성능 분석                       |
| **주요 기능**| 각 CPU 코어의 성능 메트릭 및 평균 사용률 제공 | 프로세스별 CPU, 메모리, I/O, 컨텍스트 스위치 메트릭 제공 |
| **사용 사례**| 시스템의 CPU 병목 현상 분석                   | 특정 프로세스의 리소스 사용 및 성능 문제 진단   |


## ⚙️ 환경 세팅

1. 2 CPUs, 8GB RAM
2. 패키지 설치 `sudo apt install stress sysstat`


## 🔨 stress와 sysstat
|              | **stress**                                    | **sysstat**                                   |
|--------------|----------------------------------------------|----------------------------------------------|
| **주요 목적**| 시스템의 부하를 인위적으로 증가시키는 도구 | 시스템 성능을 모니터링하고 분석하는 도구    |
| **주요 기능**| CPU, 메모리, I/O 스트레스 테스트             | mpstat, pidstat 등을 통한 성능 모니터링       |
| **사용 사례**| 안정성 테스트, 성능 벤치마킹                  | 실시간 모니터링, 성능 분석                   |
| **결과 확인**| 부하 상태에서의 시스템 반응                   | 각 CPU 및 프로세스의 성능 메트릭             |


## 👀 Test 하기 전 Average Load 확인하기
`uptime`를 사용하여 시스템 가동 시간, 현재 시각, 로그인한 사용자 수, 그리고 __시스템 평균 부하(Average Load)__ 를 출력합니다.
```shell
ubuntu@servername:~$ uptime
12:25:28 up  4:26,  4 users,  load average: 0.04, 0.03, 0.00
```
1. 현재 시각: 명령어 실행 시점의 시스템 시간입니다.
2. 시스템 가동 시간: 서버가 마지막으로 재부팅된 이후 얼마나 오랫동안 가동 중인지에 대한 시간입니다.
3. 현재 로그인 사용자 수: 시스템에 로그인한 사용자 수 입니다.
4. 평균 부하 (Load Average): 시스템의 1분, 5분, 15분 동안의 평균 부하입니다. 각 숫자는 시스템에서 처리해야 할 작업 큐의 길이를 나타내며, 이는 CPU와 I/O를 포함한 작업 부하를 반영합니다.
 

## 1️⃣ Test 1. CPU-intensive process    


**Terminal 1** ▶️ `stress --cpu 1 --timeout 600`

`stress --cpu 1 --timeout 600`은 CPU를 대상으로 한 스트레스 테스트를 10분(600초) 동안 수행하는 명령어입니다.

- __cpu 1__: CPU 집약적인 작업을 수행하는 1개의 가상 CPU 작업을 생성합니다. 이 작업은 CPU를 지속적으로 사용하여 부하를 가중시킵니다.
- __timeout 600__: 테스트가 600초(10분) 동안 실행됩니다. 이 시간이 지나면 작업이 자동으로 종료됩니다.

즉, CPU에 부하를 주는 작업을 1개의 프로세스로 600초 동안 수행하여 CPU 사용률을 높이는 방식으로 시스템을 테스트하는 것을 의미합니다.       

![image](https://github.com/user-attachments/assets/50640452-a617-43c1-ac78-db420ace7dec)


**Terminal 2** ▶️ `uptime`    

![image](https://github.com/user-attachments/assets/6da75941-e277-42b3-b368-99b980debc5b)
Average Load가 증가한 것을 확인할 수 있습니다.    


**Terminal 3** ▶️ `mpstat -P ALL 5`    

`mpstat -P ALL 5`는 mpstat 도구를 사용하여 모든 CPU 코어에 대한 실시간 CPU 성능 메트릭을 5초 간격으로 표시하는 명령어입니다.     

- -P ALL: 모든 CPU 코어에 대한 정보를 출력합니다. 각 CPU 코어의 성능을 개별적으로 볼 수 있으며, 전체 CPU의 평균 성능도 함께 표시됩니다.
- 5: 5초 간격으로 CPU 성능 데이터를 업데이트하여 출력합니다. 실시간으로 CPU 사용률을 계속 모니터링할 수 있습니다.
  
![image](https://github.com/user-attachments/assets/2ff83490-42b8-4655-8a23-98ecfab0505a)
CPU는 100% 사용하지만 iowait는 0.00%인 것을 확인할 수 있습니다.


## 2️⃣ Test 2. I/O-intensive process
**Terminal 1** ▶️ `stress -i 1 --timeout 600`

`stress -i 1 --timeout 600`은 I/O 관련 스트레스 테스트를 10분(600초) 동안 수행하는 명령어입니다.

- -i 1: 1개의 I/O 집약적인 작업을 생성합니다. 이 작업은 CPU 대신 주로 I/O(입출력) 작업을 반복적으로 실행하여, 디스크나 파일 시스템과의 상호작용을 통해 부하를 가중시킵니다.
- --timeout 600: 이 테스트는 600초(10분) 동안 실행된 후 자동으로 종료됩니다.

즉, I/O에 집중된 작업을 통해 시스템의 입출력 성능이나 처리 능력을 테스트하는 데 사용됩니다. CPU 사용률은 높지 않을 수 있지만, 평균 부하(Average Load)는 높아질 수 있습니다.    

![image](https://github.com/user-attachments/assets/90278fb6-2451-4774-bc73-48e8422131c8)

**Terminal 2** ▶️ `uptime`    

![image](https://github.com/user-attachments/assets/d5e8b7a7-aac0-4383-a9b2-2f8a187ffcd8)
Average Load가 증가한 것을 확인할 수 있습니다.

**Terminal 3** ▶️ `mpstat -P ALL 5 1`
![image](https://github.com/user-attachments/assets/cbdd4905-5fe3-4ed6-aaf4-f4b1124b9e6b)

__pidstat 이용해서 high iowait 찾기__     

`pidstat -u 5 1`    

각 프로세스의 CPU 사용량을 표시하며, 프로세스의 사용자 모드(%usr), 시스템 모드(%system), 대기 시간(%wait), 전체 CPU 사용률(%CPU) 등을 보여줍니다. 이를 통해 특정 프로세스가 CPU를 얼마나 사용하는지 모니터링할 수 있습니다.
![image](https://github.com/user-attachments/assets/1c050b8b-1d7b-448f-a63d-aaafcdbc3abf)

- -u: 프로세스별 CPU 사용률 정보를 출력합니다. 이 옵션은 프로세스가 사용자 모드와 시스템 모드에서 각각 얼마나 많은 CPU를 사용하는지 보여줍니다.
- 5: 5초 동안 대기 후 데이터를 수집합니다. 5초마다 데이터가 갱신되도록 설정할 수 있지만, 여기서는 한 번만 실행되므로 5초 후 수집합니다.
- 1: 총 한 번만 데이터를 수집하여 출력합니다.


## 3️⃣ Test 3. 많은 프로세스 실행
**Terminal 1** ▶️ `stress -c 8 --timeout 600`    

stress 도구를 사용하여 8개의 CPU 작업을 동시에 수행하며, 600초(10분) 동안 CPU에 부하를 가하는 명령어입니다.    


- -c 8: 8개의 CPU 작업(쓰레드)을 생성하여, CPU에 부하를 가합니다. 즉, 8개의 코어 혹은 스레드에 대해 연산 작업을 수행하게 됩니다.
- –timeout 600: 이 명령을 600초(10분) 동안 실행한 후 자동으로 종료됩니다.

즉, CPU 성능 및 안정성 테스트를 위해 CPU에 고의적으로 부하를 주는 데 사용됩니다. CPU 집약적인 작업이 시스템에 미치는 영향을 확인하고, 과부하 상태에서 시스템이 어떻게 반응하는지 평가할 수 있습니다.     


![image](https://github.com/user-attachments/assets/e9b46bcb-8af5-4f61-93d7-3c760f3928c6)

**Temrinal 2**
![image](https://github.com/user-attachments/assets/94cb7849-0598-4287-a17d-0b8ee2b3c128)

**Terminal 3**
![image](https://github.com/user-attachments/assets/8bd92de0-3b11-4034-8b8c-f4028e4e7070)

## 4️⃣ Test 4. (예제) 여러 스레드가 CPU 스케줄링을 기다리는 상태
**Terminal 1**
![image](https://github.com/user-attachments/assets/29f48924-0c8c-4b59-9945-9f0dfd911163)

**Terminal 2**
![image](https://github.com/user-attachments/assets/11dd5a63-cc7b-4cff-81e0-1148a1dd4f7c)

**Terminal 3**
![image](https://github.com/user-attachments/assets/69579ce2-b67c-4f0d-995d-c09e38cb4102)
**stress 프로세스** - 6854, 6855, 6856, 6857

- 거의 100%에 가까운 CPU 사용량을 차지하고 있다.
- %usr가 매우 높기 때문에 이 프로세스들은 CPU를 강하게 사용하는 작업을 하고 있음을 알 수 있다.
- %system은 0.00%로 사용자 모드에서만 CPU를 소모하고 시스템 호출은 거의 발생하지 않는 상태이다.
- %wait도 0.00%로 I/O 대기 없이 CPU만 집중적으로 사용하고 있다.

**pidstat 프로세스** - 6861

- 매우 적은 리소스를 사용하고 있다.
- %CPU는 0.20%로 주로 시스템 리소스를 수집하는 작은 작업을 수행하는 것을 의미한다.