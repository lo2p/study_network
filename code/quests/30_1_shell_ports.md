Shell Script 실무 연습문제 - 네트워크 프로세스 관리
사용 가능한 문법 제한사항
허용: if문, 변수, for문, 배열, 기본 shell 명령어
금지: awk, sed
핵심 명령어: ss, lsof, curl, telnet, kill, ps

연습문제 1: 포트 사용 여부 확인 스크립트
문제: 여러 포트 번호를 인수로 받아서 각 포트가 사용 중인지 확인하는 스크립트를 작성하세요.
요구사항:
스크립트 실행: ./check_ports.sh 80 443 22 3000 8080
ss 명령어 사용하여 포트 상태 확인
사용 중인 포트는 ✓, 사용하지 않는 포트는 ✗ 표시
실행 예제:
```bash
$ ./check_ports.sh 80 443 22 3000 8080
포트 사용 상태 확인 중...
포트 80: ✓ 사용 중
포트 443: ✗ 사용 안함
포트 22: ✓ 사용 중
포트 3000: ✗ 사용 안함
포트 8080: ✓ 사용 중

총 5개 포트 중 3개가 사용 중입니다.
```

check_ports.sh
```bash
#!/bin/bash

echo "포트 사용 상태 확인 중..."

V_COUNT=0

for i in $*; do
        V_PORTSTAT=$(lsof -i :$i)
        if [ -n "$V_PORTSTAT" ]; then
                echo "포트 $i: ✓ 사용 중"
                ((V_COUNT++))
        else
                echo "포트 $i: ✗ 사용 안함"
        fi
done

echo "총 $# 개 포트 중 $V_COUNT 개가 사용 중입니다."
```


연습문제 2: 특정 포트 프로세스 종료 스크립트
문제: 특정 포트를 사용하는 프로세스를 찾아서 종료하는 스크립트를 작성하세요.
요구사항:
스크립트 실행: 
./kill_port.sh 8080
lsof -i :포트번호 사용하여 프로세스 찾기
프로세스가 있으면 PID 출력 후 종료
프로세스가 없으면 해당 메시지 출력
실행 예제:

kill_port.sh
```bash
$ ./kill_port.sh 8080
포트 8080 사용 프로세스 검색 중...
발견된 프로세스:
  PID: 12345, 프로세스명: node
  PID: 12346, 프로세스명: nginx

프로세스 종료 중...
  PID 12345 종료 완료
  PID 12346 종료 완료

포트 8080이 해제되었습니다.
```

프로세스가 없는 경우:
```bash
$ ./kill_port.sh 9999
포트 9999 사용 프로세스 검색 중...
포트 9999를 사용하는 프로세스가 없습니다.
```

kill_port.sh

> `lsof -i :$port -t` 을 사용하면 PID만 출력
> `ps -p $PID -o comm` 을 사용하면 full CMD의 command 부분만 출력

```bash
#!/bin/bash

for port in "$@"; do
    echo "포트 $PORT 사용 프로세스 검색 중..."

    # lsof -i :$port -t 사용 가능
    V_PIDS=$(lsof -i :$port | tr -s " " | cut -d" " -f2 | tail -n +2)

    if [ -z "$V_PIDS" ]; then
        echo "포트 $port 를 사용하는 프로세스가 없습니다."
        continue
    fi

    # ps -p $PID -o comm 사용 가능
    V_PNAME=$(ps -p $PID | tr -s " " | cut -d" " -f2 | tail -n +2)

    echo "발견된 프로세스:"
    echo "  PID: $PID, 프로세스명: $V_NAME"
    echo
    echo "프로세스 종료 중..." 
    echo

    if kill -9 "$V_PIDS"; then
        echo "  PID $V_PIDS 종료 완료"
    else
        echo "  PID $V_PIDS 종료 실패"
    fi

    echo "포트 $PORT이 해제되었습니다."
done
```

연습문제 3: 웹 서비스 상태 모니터링 스크립트
문제: 여러 웹 서비스의 상태를 확인하는 모니터링 스크립트를 작성하세요.
요구사항:
설정 파일 형태로 서버 정보 관리: IP:포트 형식
curl 명령어로 HTTP 응답 확인 (타임아웃 5초)
연결 가능하면 ✓, 불가능하면 ✗ 표시
실행 예제:

```bash
$ ./monitor_services.sh
웹 서비스 상태 모니터링

========================

서비스 상태 확인 중...
192.168.1.100:80   ✓ 정상 (응답시간: 0.234초)
192.168.1.100:443  ✗ 연결 실패
localhost:3000     ✓ 정상 (응답시간: 0.012초)
192.168.1.200:8080 ✗ 타임아웃
localhost:22       ✗ HTTP 서비스 아님

========================
총 5개 서비스 중 2개 정상
모니터링 완료: 2024-07-28 14:30:15
```

monitor_list.txt
```txt
192.168.0.100:80
192.168.0.100:43
localhost:3000
192.168.0.100:8080
localhost:22
```

monitor_services.sh

> `for line in $(cat monitor_list.txt)` 공백 = 라인
> `curl -w` stdout & stderr : 응답시간 및 http response 코드 만 추출 

```bash
#!/bin/bash

V_TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

V_TOTAL=0
V_SUCCESS=0

echo "웹 서비스 상태 모니터링"
echo
echo "========================"
echo
echo "서비스 상태 확인 중..."

for ipp in $(cat monitor_list.txt); do
    ((V_TOTAL++))

    HOST=$(echo "$ipp" | cut -d':' -f1)
    PORT=$(echo "$ipp" | cut -d':' -f2)
    URL="http://$HOST:$PORT"

    RESPONSE=$(curl -s --connect-timeout 2 -o /dev/null -w "%{time_total} %{http_code}" "$URL")
    V_EXITCODE=$?

    if [ $V_EXITCODE -eq 0 ]; then
        V_TIME=$(echo $RESPONSE | cut -d" " -f1)
        V_HTTP_CODE=$(echo $RESPONSE | cut -d" " -f2) 
        if [ "$V_HTTP_CODE" -ge 200 ] && [ "$V_HTTP_CODE" -lt 400 ]; then
            echo "$ipp  ✓ 정상 (응답시간: ${V_TIME}초)"
            ((V_SUCCESS++))
        else
            echo "$ipp  ✗ 연결 실패"
        fi
    elif [ $V_EXITCODE -eq 28 ]; then
        echo "$ipp  ✗ 타임아웃"
    elif [ $V_EXITCODE -eq 7 ]; then
        echo "$ipp  ✗ 연결 실패"
    else
        echo "$ipp  ✗ HTTP 서비스 아님"
    fi
done

echo
echo "========================"
echo "총 ${V_TOTAL}개 서비스 중 ${V_SUCCESS}개 정상"
echo "모니터링 완료: $V_TIMESTAMP"
```

연습문제 4: 네트워크 연결 테스트 스크립트
문제: telnet을 이용하여 원격 서버의 포트 연결 상태를 확인하는 스크립트를 작성하세요.
요구사항:
스크립트 실행: ./test_connection.sh google.com:80 localhost:22 192.168.1.1:443
telnet 명령어 사용 (타임아웃 3초)
연결 성공/실패 상태 표시
실행 예제:

```bash
$ ./test_connection.sh google.com:80 localhost:22 192.168.1.1:443

네트워크 연결 테스트 시작
==========================

google.com:80 연결 테스트 중...
  ✓ 연결 성공

localhost:22 연결 테스트 중...
  ✓ 연결 성공

192.168.1.1:443 연결 테스트 중...
  ✗ 연결 실패 (타임아웃)

==========================
테스트 완료: 3개 중 2개 성공
```


test_connection.sh

> `talent`에 timeout이 없어서 `timeout` 으로 대체함
> `&>`: redirect stdout과 stderr를 파일로 
> `2>&1`: stdout과 stderr를 stdout으로

```bash
#!/bin/bash

echo "네트워크 연결 테스트 시작"
echo "=========================="

V_TOTAL=0
V_SUCCESS=0

for ipp in "$@"; do
    ((V_TOTAL++))
    
    V_HOST=$(echo "$ipp" | cut -d':' -f1)
    V_PORT=$(echo "$ipp" | cut -d':' -f2)

    echo "$ipp 연결 테스트 중..."

    if timeout 3 telnet "$V_HOST" "$V_PORT" < /dev/null 2>&1 | grep -q "Connected"; then
        echo "✓ 연결 성공"
        ((V_SUCCESS++))
    else
        echo "✗ 연결 실패 (타임아웃)"
    fi
done

echo "=========================="
echo "테스트 완료: ${V_TOTAL}개 중 ${V_SUCCESS}개 성공"
```

연습문제 5: 프로세스별 네트워크 포트 사용 현황 스크립트
문제: 현재 네트워크 포트를 사용하는 프로세스들의 정보를 정리하여 출력하는 스크립트를 작성하세요.
요구사항:
ss 명령어로 LISTEN 상태 포트 확인
lsof 명령어로 프로세스 정보 확인
포트 번호별로 정렬하여 출력
실행 예제:
```bash
$ ./port_usage.sh
네트워크 포트 사용 현황
======================

LISTEN 상태 포트 분석 중...

포트 22 (SSH)
  프로세스: sshd (PID: 1234)
  주소: 0.0.0.0:22

포트 80 (HTTP)
  프로세스: nginx (PID: 5678)
  주소: 0.0.0.0:80

포트 3306 (MySQL)
  프로세스: mysqld (PID: 9012)
  주소: 127.0.0.1:3306

포트 8080 (사용자 정의)
  프로세스: java (PID: 3456)
  주소: 0.0.0.0:8080

======================
총 4개 포트가 LISTEN 상태입니다.
```

port_usage.sh

> `IFS` 사용해서 for 루프의 구분자를 `\n` 으로 변경

```bash
#/bin/bash

echo "네트워크 포트 사용 현황"
echo "======================"
echo
echo "LISTEN 상태 포트 분석 중..."
echo

V_TOTAL=0

V_IPP=$(ss -tulpnH | tr -s " ")

IFS=$'\n'

for raw_ipp in $V_IPP; do
    ((V_TOTAL++))
    V_PROTO=$(echo "$raw_ipp" | cut -d" " -f1)
    V_IPP=$(echo "$raw_ipp" | cut -d" " -f5)
    V_PORT=$(echo "$raw_ipp" | cut -d" " -f5 | cut -d":" -f2)
    V_PROCESS=$(lsof -i :"$V_PORT" | tr -s " " | tail -n +2 | cut -d" " -f1)
    V_PID=$(lsof -i :"$V_PORT" | tr -s " " | tail -n +2 | cut -d" " -f2)
    V_SERVICE_NAME=$(grep -w "${V_PORT}/${V_PROTO}" /etc/services | cut -f1 | head -n1)
    
    if [ -z "$V_SERVICE_NAME" ]; then
        V_SERVICE_NAME="사용자 정의"
    fi

    echo "포트 $V_PORT ($V_SERVICE_NAME)"
    echo "  프로세스: $V_PROCESS (PID: $V_PID)"
    echo "  주소: $V_IPP"
    echo
done

IFS=$' \t\n'

echo "======================"
echo "총 $V_TOTAL 개 포트가 LISTEN 상태입니다."
```

연습문제 6: 대량 포트 킬러 스크립트
문제: 포트 범위를 지정하여 해당 범위의 모든 사용 중인 포트를 찾아 프로세스를 종료하는 스크립트를 작성하세요.
요구사항:
스크립트 실행: ./kill_port_range.sh 3000 3010
지정된 범위의 포트 중 사용 중인 것만 확인
사용자에게 확인 후 프로세스 종료
실행 예제:
```bash
$ ./kill_port_range.sh 3000 3010
포트 범위 3000-3010 검색 중...

사용 중인 포트 발견:
  포트 3000: node (PID: 12345)
  포트 3001: node (PID: 12346)
  포트 3005: python (PID: 12347)

위 3개 프로세스를 종료하시겠습니까? (y/N): y

프로세스 종료 중...
  포트 3000 (PID: 12345) 종료 완료
  포트 3001 (PID: 12346) 종료 완료
  포트 3005 (PID: 12347) 종료 완료

모든 프로세스가 종료되었습니다.
```

kill_port_range.sh
```bash
#/bin/bash

V_COUNT=0

echo "포트 범위 $1-$2 검색 중..."
echo
V_PORTS=$(ss -tulpnH | tr -s " " | cut -d" " -f5 | cut -d":" -f2)

IFS=$'\n'
echo "사용 중인 포트 발견:"
for port in $V_PORTS; do
    if [ "$port" -ge $1 ] && [ "$port" -le $2 ]; then
        ((V_COUNT++))
        V_PID=$(lsof -i :"$port" | tr -s " " | tail -n +2 | cut -d" " -f2)
        V_PROCESS=$(lsof -i :"$port" | tr -s " " | tail -n +2 | cut -d" " -f1)
        echo "포트 $port: $V_PROCESS (PID: $V_PID)"
    fi
done

echo
read -p "위 $V_COUNT 개 프로세스를 종료하시겠습니까? (y/N): " V_DELETE_CHOICE
echo

if [[ "$V_DELETE_CHOICE" == "Y" || "$V_DELETE_CHOICE" == "y" ]]; then
    echo "프로세스 종료 중..."
    for port in $V_PORTS; do
        if [ "$port" -ge "$1" ] && [ "$port" -le "$2" ]; then
            V_PID=$(lsof -i :"$port" | tr -s " " | tail -n +2 | cut -d" " -f2)
            if kill -9 "$V_PID" &> /dev/null; then
                echo "포트 $port: (PID: $V_PID) 종료 완료"
            else
                echo "포트 $port: (PID: $V_PID) 종료 실패"
            fi
        fi
    done
else
    echo "프로세스의 종료를 취소하셨습니다"
fi

echo
echo "모든 프로세스가 종료되었습니다."
IFS=$' \t\n'
```

연습문제 7: 서비스 자동 재시작 스크립트
문제: 특정 포트의 서비스가 다운되었을 때 자동으로 재시작하는 스크립트를 작성하세요.
요구사항:
설정: 포트 번호, 재시작 명령어
curl 또는 telnet으로 서비스 상태 확인
다운 시 기존 프로세스 정리 후 재시작
실행 예제:
```bash
$ ./auto_restart.sh
서비스 자동 재시작 모니터
========================

설정된 서비스들:
  - 포트 3000: npm start
  - 포트 8080: java -jar app.jar
  - 포트 5000: python app.py

모니터링 시작...

[14:30:15] 포트 3000 상태 확인... ✓ 정상
[14:30:16] 포트 8080 상태 확인... ✗ 다운됨
[14:30:16] 포트 8080 서비스 재시작 중...
[14:30:16] 기존 프로세스 정리 완료
[14:30:17] 새 프로세스 시작: java -jar app.jar
[14:30:20] 포트 8080 재시작 완료 ✓
[14:30:21] 포트 5000 상태 확인... ✓ 정상

다음 확인까지 30초 대기 중...
```

auto_restart.sh
```bash
#!/bin/bash

V_RAW_DATA=$(cat << EOF
7000:python3 -m http.server
8000:python3 -m http.server
9000:python3 -m http.server
EOF
)

echo "서비스 자동 재시작 모니터"
echo "========================"
echo "설정된 서비스들:"
echo

IFS=$'\n'
for data in $V_RAW_DATA; do
    V_TMP_PORT=$(echo "$data" | cut -d":" -f1)
    V_TMP_CMD=$(echo "$data" | cut -d":" -f2-)
    echo "- 포트 $V_TMP_PORT: $V_TMP_CMD"
done

echo
echo "모니터링 시작..."
echo

while true; do
    for data in $V_RAW_DATA; do
        V_TIMESTAMP=$(date +"%H:%M:%S")

        # Extract port and command
        V_PORT=$(echo "$data" | cut -d":" -f1)
        V_CMD=$(echo "$data" | cut -d":" -f2-)

        # Get PID listening on port
        V_PID=$(lsof -t -i :"$V_PORT" 2>/dev/null | head -n 1)

        # Check if service is up via curl
        if curl --silent --fail --max-time 5 http://127.0.0.1:$V_PORT > /dev/null 2>&1; then
            echo "[$V_TIMESTAMP] 포트 $V_PORT 상태 확인... ✓ 정상"
        else
            echo "[$V_TIMESTAMP] 포트 $V_PORT 상태 확인... ✗ 다운됨"
            echo "[$V_TIMESTAMP] 포트 $V_PORT 서비스 재시작 중..."

            if [ -n "$V_PID" ]; then
                kill -9 "$V_PID"
                echo "[$V_TIMESTAMP] 기존 프로세스 정리 완료"
            fi

            echo "[$V_TIMESTAMP] 새 프로세스 시작: $V_CMD"
            nohup bash -c "$V_CMD $V_PORT" &> /dev/null &
            echo "[$V_TIMESTAMP] 포트 $V_PORT 재시작 완료 ✓"
        fi
    done
    echo
    echo "다음 확인까지 30초 대기 중..."
    sleep 30
    echo
done
unset IFS
```

연습문제 8: 포트 스캐너 스크립트
문제: 특정 IP 주소의 포트 범위를 스캔하여 열린 포트를 찾는 스크립트를 작성하세요.
요구사항:
스크립트 실행: ./port_scan.sh 192.168.1.100 20 30
telnet 명령어 사용하여 포트 연결 테스트
타임아웃 2초로 설정
열린 포트만 출력
실행 예제:
$ ./port_scan.sh 192.168.1.100 20 30
192.168.1.100 포트 스캔 (범위: 20-30)
=====================================

스캔 진행 중... [###########] 100%

열린 포트 발견:
  포트 22: SSH (OpenSSH)
  포트 80: HTTP
  포트 443: HTTPS

=====================================
총 11개 포트 중 3개가 열려있습니다.
스캔 완료 시간: 22초


추가 도전 과제
도전과제 1: 네트워크 트래픽 모니터링
ss 명령어로 연결 상태별 통계 출력
TCP/UDP 연결 수 집계
도전과제 2: 포트 사용 히스토리
주기적으로 포트 사용 현황을 로그 파일에 기록
시간대별 포트 사용 패턴 분석
도전과제 3: 멀티 서버 상태 확인
여러 서버의 동일 포트 상태를 동시에 확인
병렬 처리로 성능 향상

참고사항
유용한 명령어 조합:
# 포트 사용 확인
ss -tlnp | grep :포트번호

# 프로세스 정보 확인
lsof -i :포트번호

# 연결 테스트
curl -I --connect-timeout 5 http://IP:포트
timeout 3 telnet IP 포트

# 프로세스 강제 종료
kill -9 PID

이 연습문제들을 통해 실무에서 자주 사용하는 네트워크 관련 스크립트 작성 능력을 기를 수 있습니다!
