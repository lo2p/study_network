# Network Shell Script 실습 문제

## 문제 환경 설정

실습을 위해 다음 파일들을 생성하세요:

# 1. 네트워크 로그 파일 생성

```bash
cat > network.log << 'EOF'
2024-01-15 10:30:25 192.168.1.100 CONNECT success
2024-01-15 10:30:30 192.168.1.101 CONNECT failed
2024-01-15 10:31:15 192.168.1.102 CONNECT success
2024-01-15 10:31:20 192.168.1.100 DISCONNECT success
2024-01-15 10:32:10 192.168.1.103 CONNECT success
2024-01-15 10:32:15 192.168.1.101 CONNECT success
2024-01-15 10:33:25 192.168.1.104 CONNECT failed
2024-01-15 10:33:30 192.168.1.102 DISCONNECT success
EOF
```

# 2. 접속 통계 파일 생성

```bash
cat > connections.txt << 'EOF'
192.168.1.100 5
192.168.1.101 12
192.168.1.102 8
192.168.1.103 3
192.168.1.104 15
192.168.1.105 7
EOF
```

---

## 문제 1: 네트워크 연결 상태 분석기

요구사항:

- network.log 파일을 분석하여 연결 성공/실패 통계를 출력하는 스크립트 작성
- 전체 연결 시도 수, 성공 수, 실패 수를 계산
- 성공률을 백분율로 표시 (소수점 제거)

출력 형태:

=== 네트워크 연결 분석 결과 ===

전체 연결 시도: X건

성공: Y건

실패: Z건

성공률: W%

제한사항:

- if문과 변수만 사용
- grep, wc, cut 명령어 활용
- 파일명은 스크립트 실행 시 첫 번째 인자로 받기

networkloganalyzer.sh
```bash
#!/bin/bash

V_COUNTS=$(wc -l < network.log)
V_SUCCESS=$(grep "success" network.log | wc -l)
V_FAILED=$(grep "failed" network.log | wc -l)
V_PERCENT=$(( V_SUCCESS * 100 / V_COUNTS ))

echo "=== 네트워크 연결 분석 결과 ==="
echo
echo "전체 연결 시도: "$V_COUNTS" 건"
echo "성공: "$V_SUCCESS" 건"
echo "실패: "$V_FAILED" 건"
echo "성공률: "$V_PERCENT" %"
```

---

## 문제 2: IP 주소별 접속 빈도 상위 리스트

요구사항:

- network.log에서 IP 주소별 접속 횟수를 계산
- 접속 횟수 기준으로 내림차순 정렬하여 상위 3개만 출력
- 각 IP의 첫 접속 시간도 함께 표시

출력 형태:

=== 접속 빈도 TOP 3 ===

1위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

2위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

3위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

제한사항:

- if문과 변수만 사용
- cut, sort, uniq, grep 명령어 활용
- head나 tail로 결과 제한

networktop.sh
```bash
#!/bin/bash

V_FIRST_IP=$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 1 | tr -s " " | cut -d" " -f"3")
V_SECOND_IP=$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 2 | tail -n 1 | tr -s " " | cut -d" " -f"3")
V_THIRD_IP=$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 3 | tail -n 1 | tr -s " " | cut -d" " -f"3")

V_FIRST_IP_COUNT=$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 1 | tr -s " " | cut -d" " -f"2")
V_SECOND_IP_COUNT=$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 2 | tail -n 1 | tr -s " " | cut -d" " -f"2")
V_THIRD_IP_COUNT=$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 3 | tail -n 1 | tr -s " " | cut -d" " -f"2")

V_FIRST_IP_TIME=$(sort -k"2" network.log | grep $V_FIRST_IP | head -n 1 | cut -d" " -f"1,2")
V_SECOND_IP_TIME=$(sort -k"2" network.log | grep $V_FIRST_IP | head -n 1 | cut -d" " -f"1,2")
V_THIRD_IP_TIME=$(sort -k"2" network.log | grep $V_FIRST_IP | head -n 1 | cut -d" " -f"1,2")

echo "$(cut -d" " -f"3" network.log | sort | uniq -c | sort -nr | head -n 1 | tr -s " " | cut -d" " -f"2")"

echo "=== 접속 빈도 TOP 3 ==="

echo "1위: $V_FIRST_IP ($V_FIRST_IP_COUNT회) - 첫 접속: $V_FIRST_IP_TIME"
echo "2위: $V_SECOND_IP ($V_SECOND_IP_COUNT회) - 첫 접속: $V_SECOND_IP_TIME"
echo "3위: $V_THIRD_IP ($V_THIRD_IP_COUNT회) - 첫 접속: $V_THIRD_IP_TIME"
```

---

## 문제 3: 서버 상태 점검 스크립트

요구사항:

- servers.sh 실행해 각 서버에 대해 ping 테스트 실행
- 응답 있는 서버와 없는 서버를 구분하여 출력
- 응답 시간이 100ms 이상인 서버는 "느림" 표시

출력 형태:

=== 서버 상태 점검 결과 ===

[정상] web01 (192.168.1.10) - 응답시간: XXms

[정상] web02 (192.168.1.11) - 응답시간: XXms (느림)

[오프라인] db01 (192.168.1.20) - 응답없음

...

제한사항:

- if문과 변수만 사용
- cut, ping 명령어 활용
- ping은 1회만 실행 (ping -c 1)

servers.sh
```bash
read -p "enter ip address to ping test: " V_IP

V_RAW_PING=$(ping -c 1 "$V_IP")
V_PACKET_COUNT=$(echo "$V_RAW_PING" | grep "packet loss" | cut -d" " -f"4")

# 응답없음 체크
if [ "$V_PACKET_COUNT" -eq 0 ]; then
        echo "[오프라인($V_IP)] - 응답없음"
        return 1
fi

V_RESPONSETIME=$(echo "$V_RAW_PING" | grep "time" | head -n 1 | cut -d" " -f"7" | cut -d"=" -f"2")
V_DEC_RES_TIME="${V_RESPONSETIME%%.*}"

if [ "$V_DEC_RES_TIME" -ge 100 ]; then
        echo "[정상] ($V_IP) - 응답시간: "$V_RESPONSETIME" ms (느림)"
else
        echo "[정상] ($V_IP) - 응답시간: "$V_RESPONSETIME" ms"
fi
```

---

## 문제 4: 네트워크 트래픽 임계값 모니터링

요구사항:

- connections.txt에서 접속 수가 10 이상인 IP를 "높음", 5-9는 "보통", 4 이하는 "낮음"으로 분류
- 각 분류별로 개수 계산하여 출력
- "높음" 분류의 IP들만 별도로 나열

출력 형태:

=== 트래픽 분석 결과 ===

높음(10회 이상): X개
보통(5-9회): Y개  
낮음(4회 이하): Z개

  

[주의 필요 IP 목록]

192.168.1.XXX (XX회)
192.168.1.XXX (XX회)

  

제한사항:

- if문과 변수만 사용
- cut, sort 명령어 활용
- 숫자 비교를 위한 조건문 사용

connection_count.sh
```bash
#!/bin/bash

V_HIGH_LIST=$(grep -E " [1-9][0-9]+$" connections.txt)
V_NORMAL_LIST=$(grep -E " [5-9]$" connections.txt)
V_LOW_LIST=$(grep -E " [0-4]$" connections.txt)

V_HIGH_COUNT=$(echo "$V_HIGH_LIST" | wc -l)
V_NORMAL_COUNT=$(echo "$V_NORMAL_LIST" | wc -l)
V_LOW_COUNT=$(echo "$V_LOW_LIST" | wc -l)

echo "=== 트래픽 분석 결과 ==="
echo
echo "높음(10회 이상): $V_HIGH_COUNT 개"
echo "보통(5-9회): $V_NORMAL_COUNT 개"
echo "낮음(4회 이하): $V_LOW_COUNT 개"
echo

echo "[주의 필요 IP 목록]"
echo
echo "$V_HIGH_LIST"
```

---

## 문제 5: 현재 시스템 네트워크 정보 수집기

요구사항:

- 현재 시스템의 IP 주소, 기본 게이트웨이, 활성 인터페이스 개수를 출력
- 인터넷 연결 상태 확인 (8.8.8.8로 ping 테스트)
- 모든 정보를 보기 좋게 정리하여 출력

출력 형태:

=== 시스템 네트워크 정보 ===

내부 IP: 192.168.1.XXX
기본 게이트웨이: 192.168.1.X
활성 인터페이스: X개
인터넷 연결: 정상/차단
  

제한사항:

- if문과 변수만 사용
- ip, hostname, ping, grep, wc 명령어 활용
- 각 정보를 변수에 저장 후 출력

```bash
#!/bin/bash

V_RAW_IP_A=$(ip -4 -o a | tr -s " ")
V_INTF_COUNT=$(echo "$V_RAW_IP_A" | wc -l)
V_INTERFACES=$(echo "$V_RAW_IP_A" | cut -d" " -f1,2)

echo "Interfaces"
echo "$V_INTERFACES"
read -p "Pick Interface by number: " V_INTF_NUM
echo

# user user selection to get specfic interface
V_USER_SELECT=$(echo "$V_RAW_IP_A" | grep "^$V_INTF_NUM: ")
V_US_INTF_NAME=$(echo "$V_USER_SELECT" | cut -d" " -f2)
V_US_INTF_IP=$(echo "$V_USER_SELECT" | cut -d" " -f4 | cut -d"/" -f1)
V_US_INTF_DGATEWAY=$(ip route show dev $V_US_INTF_NAME | cut -d" " -f3 | head -n 1)

# check connection
if ping -I "$V_US_INTF_NAME" -c1 -W1 8.8.8.8 >/dev/null 2>&1; then
  V_INT_STAT="정상"
else
  V_INT_STAT="차단"
fi

clear
echo "=== 시스템 네트워크 정보 ==="
echo
echo "내부 IP: $V_US_INTF_IP"
echo "기본 게이트웨이: $V_US_INTF_DGATEWAY"
echo "활성 인터페이스: $V_INTF_COUNT 개"
echo "인터넷 연결: $V_INT_STAT"
```