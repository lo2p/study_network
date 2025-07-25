# Shell 조건문 및 텍스트 처리 실습 문제

## 실습 환경 설정

### 1. 실습 디렉토리 생성 및 이동

```bash
mkdir ~/shell_practice
cd ~/shell_practice
```

### 2. 실습용 데이터 파일 생성

#### students.txt 파일 생성

```bash
cat > students.txt << EOF
김철수:수학:85:영어:92:과학:78
이영희:수학:95:영어:88:과학:91
박민수:수학:76:영어:79:과학:82
최지원:수학:88:영어:95:과학:89
정우진:수학:92:영어:76:과학:94
김지현:수학:83:영어:91:과학:87
이준호:수학:79:영어:84:과학:76
박서연:수학:97:영어:93:과학:96
한도윤:수학:81:영어:77:과학:83
송민재:수학:86:영어:89:과학:91
EOF
```

#### server_logs.txt 파일 생성

```bash
cat > server_logs.txt << EOF
2024-01-15 10:30:15 INFO User login successful: user001
2024-01-15 10:31:22 ERROR Database connection failed
2024-01-15 10:32:05 INFO User login successful: user002
2024-01-15 10:33:18 WARNING Memory usage high: 85%
2024-01-15 10:34:25 ERROR Authentication failed: user003
2024-01-15 10:35:10 INFO User logout: user001
2024-01-15 10:36:33 ERROR Network timeout occurred
2024-01-15 10:37:45 INFO System backup started
2024-01-15 10:38:52 WARNING Disk space low: 90%
2024-01-15 10:39:15 ERROR Database connection failed
2024-01-15 10:40:28 INFO User login successful: user004
2024-01-15 10:41:35 ERROR Authentication failed: user005
EOF
```

#### sales_data.txt 파일 생성

```bash
cat > sales_data.txt << EOF
2024-01,서울,노트북,1500000
2024-01,부산,스마트폰,800000
2024-01,대구,태블릿,600000
2024-01,서울,스마트폰,850000
2024-02,부산,노트북,1450000
2024-02,서울,태블릿,620000
2024-02,대구,스마트폰,780000
2024-02,서울,노트북,1520000
2024-03,부산,태블릿,590000
2024-03,대구,노트북,1480000
2024-03,서울,스마트폰,820000
2024-03,부산,스마트폰,790000
EOF
```

#### words.txt 파일 생성

```bash
cat > words.txt << EOF
apple
banana
Apple
cherry
BANANA
date
Cherry
elderberry
Apple
fig
banana
CHERRY
EOF
```

---

## 실습 문제

### 문제 1: 학생 성적 분석기 

파일: grade_analyzer.sh

요구사항:

- students.txt 파일을 분석하여 다음 기능을 수행하는 스크립트 작성
- 사용자로부터 과목명을 입력받아 해당 과목의 통계 정보 출력
- 조건문을 사용하여
-  입력 검증 및 등급 분류 수행

구현해야 할 기능:

1. 사용자가 입력한 과목이 유효한지 검사 (수학, 영어, 과학)
2. 해당 과목의 모든 점수를 추출하여 정렬된 목록 출력
3. 최고점, 최저점, 평균점수 계산
4. 90점 이상(A), 80점 이상(B), 70점 이상(C), 그 외(D) 등급별 학생 수 출력
5. 평균이 85점 이상이면 "우수", 75점 이상이면 "양호", 그 외는 "보통" 출력

힌트:

- cut, grep, sort, wc 명령어 활용
- 파이프라인으로 명령어 연결
- 조건문으로 점수 범위 검사

```bash
#!/bin/bash

# variables
V_FILE="students.txt" 
V_AGRADE=0
V_BGRADE=0
V_CGRADE=0
v_DGRADE=0

# function
countgrade() {
    if [ $1 -ge 90 ]; then
        ((++V_AGRADE))
    elif [ $1 -ge 80 ]; then
        ((++V_BGRADE))
    elif [ $1 -ge 70 ]; then
        ((++V_CGRADE))
    else
        ((++v_DGRADE))
    fi
}

read -p "Subject (math, english, science): " V_SUBJECT

V_SUBJECT=$(echo "$V_SUBJECT" | tr 'A-Z' 'a-z')

# user input 확인
if [ "$V_SUBJECT" != "math" ] && [ "$V_SUBJECT" != "english" ] && [ "$V_SUBJECT" != "science" ]; then
    echo "$V_SUBJECT 는 없는 과목입니다."
    return 1
fi

# column 선택
if [ "$V_SUBJECT" = "math" ]; then
    V_COLUMN=3
elif [ "$V_SUBJECT" = "english" ]; then
    V_COLUMN=5
else
    V_COLUMN=7
fi

# 최고점, 최저점, 평균점수 계산
V_SCORES=$(cut -d: -f"$V_COLUMN" "$V_FILE" | sort -n)

V_HIGHEST=$(echo "$V_SCORES" | tail -n 1)
V_LOWEST=$(echo "$V_SCORES" | head -n 1)

V_SUM_STR=$(echo "$V_SCORES" | tr '\n' '+' | head -c -1)
V_SUM=$((V_SUM_STR))
V_COUNT=$(echo "$V_SCORES" | wc -l)
V_AVERAGE=$(("$V_SUM" / "$V_COUNT"))

echo "$V_SCORES"; echo;
echo "Max: $V_HIGHEST"; echo "Min: $V_LOWEST"; echo "Average: $V_AVERAGE"; echo;

# 90점 이상(A), 80점 이상(B), 70점 이상(C), 그 외(D) 등급별 학생 수 출력
countgrade $(echo $V_SCORES | cut -d" " -f"1")
countgrade $(echo $V_SCORES | cut -d" " -f"2")
countgrade $(echo $V_SCORES | cut -d" " -f"3")
countgrade $(echo $V_SCORES | cut -d" " -f"4")
countgrade $(echo $V_SCORES | cut -d" " -f"5")
countgrade $(echo $V_SCORES | cut -d" " -f"6")
countgrade $(echo $V_SCORES | cut -d" " -f"7")
countgrade $(echo $V_SCORES | cut -d" " -f"8")
countgrade $(echo $V_SCORES | cut -d" " -f"9")
countgrade $(echo $V_SCORES | cut -d" " -f"10")

echo "A: $V_AGRADE"; echo "B: $V_BGRADE"; echo "C: $V_CGRADE"; echo "D: $v_DGRADE"; echo;

# 평균이 85점 이상이면 "우수", 75점 이상이면 "양호", 그 외는 "보통" 출력
if [ "$V_AVERAGE" -ge 85 ]; then
    echo "Average: $V_AVERAGE"; echo "우수"
elif [ "$V_AVERAGE" -ge 75 ]; then
    echo "Average: $V_AVERAGE"; echo "양호"
else
    echo "Average: $V_AVERAGE"; echo "보통"
fi
```

---

### 문제 2: 서버 로그 모니터링 도구

파일: log_monitor.sh

요구사항:

- server_logs.txt 파일을 분석하여 로그 수준별 통계 및 문제 상황 감지
- 조건문을 사용하여 경고 수준 결정

구현해야 할 기능:

1. 전체 로그 라인 수 출력
2. ERROR, WARNING, INFO 각각의 개수 계산 및 출력
3. ERROR 로그만 별도 파일(errors.log)로 저장
4. 가장 많이 발생한 ERROR 유형 찾기 (중복 제거 후 개수 확인)
5. ERROR 비율이 30% 이상이면 "위험", 20% 이상이면 "주의", 그 외는 "정상" 출력
6. 마지막 5개 로그 항목을 시간 역순으로 출력

힌트:

- grep, wc, uniq, sort, tail 명령어 활용
- 리다이렉션으로 파일 저장
- 수치 계산을 위한 조건문 사용

```bash
#!/bin/bash

V_FILE="server_logs.txt"

# 전체 로그 라인 수
V_LINES=$(wc -l < "$V_FILE")
echo "전체 로그 라인: $V_LINES"

# 로그 수준별 개수
V_ERROR_COUNT=$(grep -c "ERROR" "$V_FILE")
V_WARNING_COUNT=$(grep -c "WARNING" "$V_FILE")
V_INFO_COUNT=$(grep -c "INFO" "$V_FILE")

echo "ERROR: $V_ERROR_COUNT"
echo "WARNING: $V_WARNING_COUNT"
echo "INFO: $V_INFO_COUNT"

# ERROR 로그 저장
V_ERROR_LOG=$(grep "ERROR" "$V_FILE")
echo "$V_ERROR_LOG" > errors.log

# 가장 많이 발생한 ERROR 유형
echo "가장 많이 발생한 ERROR 유형": 
echo "$V_ERROR_LOG" | cut -d" " -f4- | sort | uniq -c | sort -nr | head -n 1

# ERROR 비율이 30% 이상이면 "위험", 20% 이상이면 "주의", 그 외는 "정상" 출력
V_PERCENT=$(( V_ERROR_COUNT * 100 / V_LINES ))

if [ "$V_PERCENT" -ge 30 ]; then
    echo "위험"
elif [ "$V_PERCENT" -ge 20 ]; then
  echo "주의"
else
  echo "정상"
fi

# 마지막 5개 로그 항목을 시간 역순으로 출력
echo "$V_ERROR_LOG" | sort -k1,1r -k2,2r | head -n 5
```

---
# --- 🛑 STOP HERE ---
### 문제 3: 판매 데이터 분석 시스템

파일: sales_analyzer.sh

요구사항:

- sales_data.txt 파일을 분석하여 매출 데이터 통계 생성
- 사용자 입력에 따른 다양한 분석 결과 제공

구현해야 할 기능:

1. 사용자로부터 분석 타입 입력받기 (월별/지역별/제품별)
2. 입력값이 유효하지 않으면 사용법 출력 후 종료
3. 선택한 분석 타입에 따라 다음 수행:

- 월별: 각 월의 총 매출액을 높은 순으로 정렬하여 출력
- 지역별: 각 지역의 총 매출액과 평균 매출액 출력
- 제품별: 각 제품의 판매 횟수와 총 매출액 출력

5. 전체 매출액이 1000만원 이상이면 "목표 달성", 800만원 이상이면 "양호", 그 외는 "노력 필요" 출력
    

힌트:

- cut, sort, uniq, grep 명령어 조합
- 필드 분리를 위한 -d 옵션 활용
- 조건문으로 분석 타입 분기 처리

---

### 문제 4: 단어 빈도 분석기

파일: word_frequency.sh

요구사항:

- words.txt 파일의 단어들을 분석하여 빈도수 계산
- 대소문자 처리 옵션 제공

구현해야 할 기능:

1. 사용자로부터 대소문자 구분 여부 입력받기 (y/n)
2. 입력값 검증 (y 또는 n이 아니면 재입력 요구)
3. 대소문자 구분하지 않는 경우:
    

- 모든 단어를 소문자로 변환
- 중복 제거 후 빈도수와 함께 출력
- 빈도수 기준 내림차순 정렬

5. 대소문자 구분하는 경우:


- 원본 그대로 중복 제거 후 빈도수 계산
    

7. 총 고유 단어 개수 출력
8. 가장 빈도가 높은 단어가 3회 이상 나타나면 "높은 중복도", 2회면 "보통 중복도", 1회면 "낮은 중복도" 출력
    

힌트:

- tr, sort, uniq, wc 명령어 활용
- 대소문자 변환을 위한 tr A-Z a-z
- 조건문으로 사용자 선택 처리

---
