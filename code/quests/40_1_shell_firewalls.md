🧪 문제 1: 특정 IP 차단 상태 확인 후 차단 설정
✅ 실행 예시
```bash
$ sudo ./problem1.sh
[INFO] 현재 rich rule 목록에 192.168.0.100 차단 룰이 존재하지 않습니다.
[INFO] 차단 룰을 추가합니다...
success
```

또는
```bash
$ sudo ./problem1.sh
[INFO] 192.168.0.100은 이미 차단되어 있습니다.
[SKIP] 추가 작업을 수행하지 않습니다.
```

```bash
#!/bin/bash

V_IP="192.168.0.31"

if [ -z $(sudo firewall-cmd --list-rich-rules | grep "$V_IP" | cut -d" " -f4 | cut -d"=" -f2 | tr -d '"') ]; then
        echo "[INFO] 현재 rich rule 목록에"$V_IP" 차단 룰이 존재하지 않습니다."
        echo "[INFO] 차단 룰을 추가합니다..."
        sudo firewall-cmd --permanent --add-rich-rule="rule family=\"ipv4\" source address=\"$V_IP\" reject" &> /dev/null
        sudo firewall-cmd --reload
else
        echo "[INFO] "$V_IP"은 이미 차단되어 있습니다."
        echo "[SKIP] 추가 작업을 수행하지 않습니다."
fi
```

🔒 문제 2: 포트 8080이 열려 있다면 닫기
✅ 실행 예시
```bash
$ sudo ./problem2.sh
[INFO] 포트 8080/tcp 이 열려 있습니다. 제거합니다...
success
```

또는
```bash
$ sudo ./problem2.sh
[INFO] 포트 8080/tcp 이 열려 있지 않습니다. 아무 작업도 수행하지 않습니다.
```

```bash
#/bin/bash

V_PORT="8080/tcp"

if [ -n "$(sudo firewall-cmd --list-ports | grep "8080/tcp")" ]; then
        echo "[INFO] 포트 $V_PORT 이 열려 있습니다. 제거합니다..."
        sudo firewall-cmd --permanent --remove-port="$V_PORT" &> /dev/null
        sudo firewall-cmd --reload
else
        echo "[INFO] 포트 $V_PORT 이 열려 있지 않습니다. 아무 작업도 수행하지 않습니다."
fi

```

🧩 문제 3: SSH 서비스 제거 후 특정 IP만 허용
✅ 실행 예시
```bash
$ sudo ./problem3.sh
[INFO] 8080 서비스가 열려 있습니다. 제거합니다...
success
[INFO] 192.168.0.10 IP에만 포트 8080 허용 규칙을 추가합니다...
success
```

또는
```bash
$ sudo ./problem3.sh
[INFO] SSH 서비스가 이미 제거되어 있습니다.
[INFO] 포트 8080 허용 규칙만 추가합니다...
success
```

```bash
V_IP="12.168.0.31"
V_PORT="8080"

if [ -n "$(sudo firewall-cmd --list-ports | grep "$V_PORT")" ]; then
        echo "[INFO] $V_PORT 서비스가 열려 있습니다. 제거합니다..."
        sudo firewall-cmd --permanent --remove-port="$V_PORT"/tcp &> /dev/null
        echo "[INFO] $V_IP IP에만 포트 $V_PORT 허용 규칙을 추가합니다..."
        sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="V_IP" port="$V_PORT" accept' &> /dev/null
        sudo firewall-cmd --reload
else
        echo "[INFO] $V_PORT 서비스가 이미 제거되어 있습니다."
        echo "[INFO] 포트 $V_PORT 허용 규칙만 추가합니다..."
        sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="V_IP" port="$V_PORT" accept' &> /dev/null
        sudo firewall-cmd --reload
fi
```