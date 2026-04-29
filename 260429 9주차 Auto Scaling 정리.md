# Auto Scaling

## 개념 정리

### Scaling(스케일링)이란?
애플리케이션 또는 시스템의 성능을 높이기 위해 컴퓨팅 리소스를 확장하거나 축소하는 것. 요청이 많아질 때 처리할 수 있도록 서버를 키우는 방식이다.

---

## Scale-Up vs Scale-Out

### 스케일 업 (Scale-Up, Vertical Scaling)
서버 한 대의 성능을 높이는 방식

| 항목 | t3.nano | t3.2xlarge | 차이 |
|------|---------|------------|------|
| vCPU 수 | 2 | 8 | 4배 |
| 가격 | USD 0.0065 | USD 0.416 | 약 64배 |

> 성능 4배 올리는 데 가격은 64배! 비효율적일 수 있음

**장점:** 단순하고 설정이 쉬움

**단점:**
- 물리적인 한계 존재 (더 이상 업그레이드 불가)
- 재시작 필요할 수 있음
- 하나의 서버에만 의존 → 장애 발생 시 위험

---

### 스케일 아웃 (Scale-Out, Horizontal Scaling)
서버를 여러 대로 늘려서 처리 성능을 확장하는 방식

**장점:** 무중단 확장 가능!

**단점:** 복잡한 아키텍처, Load Balancer(ELB) 등의 추가 구성 필요

---

## Auto Scaling

트래픽 상황에 맞춰 서버 수를 **자동으로** 늘리거나 줄여주는 AWS 서비스

- 트래픽 증가 시 → 인스턴스 자동 증가
- 사용량 감소 시 → 인스턴스 자동 축소
- 리소스 비용 절감 + 고가용성 유지

---

## Auto Scaling 구성 요소

### 1. Launch Template (시작 템플릿)
새 인스턴스를 생성할 때 필요한 설정을 담고 있는 **설계도**

- 어떤 AMI(이미지)로 만들지
- 인스턴스 타입 (t3.micro 등)
- 키 페어, 보안 그룹
- UserData 스크립트 (초기 환경 자동화)

### 2. Auto Scaling Group (ASG)
인스턴스를 묶어서 관리하는 단위

- 최소 / 최대 / 원하는(Desired) 인스턴스 수 설정
- 실제 인스턴스 수를 계속 모니터링하고 자동 조절
- Availability Zone 간 분산 가능

**용량 설정 규칙:**
```
최소 용량 ≤ 원하는 용량(Desired) ≤ 최대 용량
```

### 3. Scaling Policy (스케일링 정책)
언제, 어떻게 서버 수를 조절할지에 대한 규칙

| 정책 | 설명 |
|------|------|
| Target Tracking | 평균 CPU 60% 유지처럼 목표값 설정 |
| Step Scaling | CPU 70~80% → +1대, 80% 이상 → +2대 |
| Scheduled Scaling | 특정 시간에 확장 (예: 매일 오후 6시) |

### 4. CloudWatch
지표를 감시하다가 스케일링 정책을 실행하는 역할

- CPU 사용률, 네트워크 트래픽, 메모리 등 모니터링
- 조건 충족 시 Scaling Policy 트리거
- 실시간 알람 + 로그 기록 가능

---

## 실습 1) Auto Scaling 생성

### 시작 템플릿 생성

| 항목 | 값 |
|------|-----|
| AMI | Amazon Linux 2 |
| 인스턴스 유형 | t3.nano |
| 키페어 | 자신의 PEM 키 |
| 보안그룹 | 자신이 생성한 것 |

> ⚠️ 고급 세부 정보 반드시 클릭!

**UserData 스크립트:**
```bash
#!/bin/bash
yum update -y
yum install -y httpd stress
systemctl enable httpd
systemctl start httpd
echo "Hello from Auto Scaling Instance (Shingu Univ)" > /var/www/html/index.html
```

### ASG 생성 설정
- Auto Scaling 그룹 이름: `sgu-[계정번호]-AutoScaling`
- 시작 템플릿: 위에서 생성한 것 선택
- 가용 영역: ap-northeast-2a/b/c/d 모두 선택
- 원하는 용량: **2**, 최소 용량: **2**, 최대 용량: **2**

> 상태 확인 유예 기간(300초): 인스턴스 부팅 직후 "상태 확인 실패"로 종료되지 않도록 유예시간을 주는 설정

### 결과 확인
ASG 생성 후 EC2 콘솔에서 인스턴스 2대가 자동으로 실행되는 것을 확인할 수 있다.

### Auto Scaling 장애 복구 테스트
인스턴스 1대를 의도적으로 종료하면 → ASG가 감지하고 자동으로 새 인스턴스를 생성해 원하는 용량(Desired)을 유지한다.

---

## 실습 2) CPU 임계치 기반 자동 스케일링

### ASG 용량 수정
- 원하는 용량: **1**, 최소 용량: **1**, 최대 용량: **2**
- 기존 인스턴스 1대가 자동으로 종료되는 것을 확인

### 동적 크기 조정 정책 생성
- `Automatic Scaling` 탭 → `동적 크기 조정 정책 생성`
- 정책 유형: **대상 추적 크기 조정 (Target Tracking)**
- 정책 이름: `Target Tracking Policy`
- 지표 유형: 평균 CPU 사용률
- 대상 값: **20** (CPU 20% 초과 시 스케일 아웃)
- 인스턴스 워밍업: 300초

### 부하 테스트 (stress)
EC2에 SSH(PuTTY)로 접속 후 아래 명령어 실행:

```bash
stress --cpu 2 --timeout 6000
```

CPU 사용률이 임계치(20%)를 초과하면 새 인스턴스가 자동 생성된다.  
CPU가 정상으로 돌아오면 인스턴스 수가 다시 최소값(1대)으로 축소된다.

---

## 실습 3) 특정 시간대에 EC2 자동 증가 (Scheduled Scaling)

EC2 > Auto Scaling Group > **Automatic Scaling 탭** > `예약된 작업 생성`

| 항목 | 값 |
|------|-----|
| 이름 | `scaling_out_test` |
| 원하는 용량 | 2 |
| 반복 | 한 번 |
| 표준 시간대 | Asia/Seoul |
| 특정 시작 시간 | 원하는 날짜/시간 입력 |

설정한 시간이 되면 인스턴스가 자동으로 1대 더 생성되는 것을 확인할 수 있다.
