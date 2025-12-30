# Required MCP Servers for AI Agent-based Error Resolution System

## Overview
AI Agent가 검증 환경 오류를 자동으로 해결하기 위해 필요한 MCP(Model Context Protocol) 서버 목록입니다.

---

## Priority 1: 필수 MCP (없으면 작동 불가)

### 1. SSH/Remote Execution MCP ⭐⭐⭐⭐⭐

**역할**: 원격 검증 서버에서 명령어 실행

**필요한 이유**:
- 솔루션 명령어 실행 (환경변수 설정, 스크립트 실행 등)
- 검증 재실행
- 시스템 상태 확인

**주요 기능**:
```
- execute_command(host, command) - 원격 명령 실행
- read_remote_file(host, path) - 원격 파일 읽기
- write_remote_file(host, path, content) - 원격 파일 쓰기
- check_process(host, process_name) - 프로세스 상태 확인
```

**사용 예시**:
```python
# 환경변수 설정
ssh_mcp.execute_command(
    host="rtl-verification-server",
    command="export LM_LICENSE_FILE=/licenses/synopsys"
)

# 검증 재실행
ssh_mcp.execute_command(
    host="rtl-verification-server",
    command="./scripts/run_verification.sh"
)
```

**구현 방법**:
- Python: `paramiko` 라이브러리 사용
- 또는 기존 SSH MCP 서버 활용

**Agent 기능 매핑**:
- ✅ Execute (솔루션 실행)
- ✅ Detect (로그 읽기)
- ✅ Evaluate (시스템 상태 확인)

---

### 2. Filesystem MCP ⭐⭐⭐⭐⭐

**역할**: 로컬/원격 파일 시스템 작업

**필요한 이유**:
- 검증 로그 파일 읽기
- 스냅샷 생성/복원
- 디스크 공간 확인
- 임시 파일 관리

**주요 기능**:
```
- read_file(path) - 파일 읽기
- write_file(path, content) - 파일 쓰기
- list_directory(path) - 디렉토리 목록
- check_disk_space(path) - 디스크 공간 확인
- create_snapshot(path) - 백업 생성
- restore_snapshot(snapshot_id) - 백업 복원
```

**사용 예시**:
```python
# 로그 읽기
log_content = fs_mcp.read_file(
    path="/var/log/verification/latest.log"
)

# 스냅샷 생성
snapshot_id = fs_mcp.create_snapshot(
    path="/etc/environment"
)

# 디스크 공간 확인
disk_info = fs_mcp.check_disk_space("/var")
```

**구현 방법**:
- 표준 MCP: `@modelcontextprotocol/server-filesystem`
- 또는 Python `os`, `shutil` 모듈로 직접 구현

**Agent 기능 매핑**:
- ✅ Detect (로그 읽기)
- ✅ Evaluate (디스크 공간 확인)
- ✅ Execute (스냅샷 관리)

---

### 3. MongoDB MCP ✅ (이미 보유)

**역할**: 패턴, 솔루션, 실행 결과 저장/조회

**필요한 이유**:
- 86개 오류-솔루션 패턴 저장
- 실행 이력 기록
- 성능 메트릭 저장
- RAG용 벡터 임베딩 저장

**주요 기능**:
```
- insert(collection, document) - 문서 삽입
- query(collection, filter) - 문서 조회
- update(collection, filter, update) - 문서 업데이트
- aggregate(collection, pipeline) - 집계 쿼리
```

**사용 예시**:
```python
# 패턴 조회
patterns = mongo_mcp.query(
    collection="error_patterns",
    filter={"category": "environment"}
)

# 실행 결과 저장
mongo_mcp.insert(
    collection="execution_logs",
    document={
        "error": error_info,
        "solution": solution,
        "result": "success",
        "timestamp": datetime.now()
    }
)
```

**Agent 기능 매핑**:
- ✅ Match (패턴 검색)
- ✅ Monitor (결과 저장)

---

### 4. OracleDB MCP ✅ (이미 보유)

**역할**: 검증팀 데이터 조회

**필요한 이유**:
- 검증팀이 관리하는 데이터 접근
- 검증 작업 정보 조회
- 히스토리 데이터 분석

**주요 기능**:
```
- query(sql) - SQL 쿼리 실행
- fetch_verification_tasks() - 검증 작업 목록
- get_error_history() - 오류 히스토리 조회
```

**사용 예시**:
```python
# 검증 작업 조회
tasks = oracle_mcp.query(
    sql="SELECT * FROM verification_tasks WHERE status='failed'"
)
```

**Agent 기능 매핑**:
- ✅ Detect (검증 작업 상태 확인)
- ✅ Monitor (히스토리 분석)

---

## Priority 2: 매우 중요 (자동화 완성도 향상)

### 5. Messenger API MCP ⭐⭐⭐⭐

**역할**: 팀 메신저로 승인 요청 및 알림

**필요한 이유**:
- Human-in-the-loop 승인 요청
- 실행 결과 실시간 알림
- 에러 발생 시 즉시 알림

**주요 기능**:
```
- send_message(channel, message) - 메시지 전송
- send_approval_request(message, buttons) - 승인 요청
- wait_for_response(message_id, timeout) - 응답 대기
- send_alert(severity, message) - 긴급 알림
```

**사용 예시**:
```python
# 승인 요청
message_id = messenger_mcp.send_approval_request(
    channel="verification-team",
    message="Approval needed for solution ENV001",
    buttons=[
        {"label": "Approve", "action": "approve"},
        {"label": "Reject", "action": "reject"}
    ]
)

# 응답 대기
response = messenger_mcp.wait_for_response(
    message_id=message_id,
    timeout=3600  # 1시간
)
```

**구현 방법**:
- 회사 메신저 API에 맞춰 직접 구현
- Slack: `slack_sdk` 사용
- MS Teams: `pymsteams` 사용

**Agent 기능 매핑**:
- ✅ Approve (승인 요청)
- ✅ Monitor (알림)

---

### 6. Email MCP ⭐⭐⭐

**역할**: 이메일 알림 및 리포트

**필요한 이유**:
- 승인 요청 백업 (메신저 놓쳤을 경우)
- 일일/주간 리포트 발송
- 중요 오류 알림

**주요 기능**:
```
- send_email(to, subject, body) - 이메일 전송
- send_html_email(to, subject, html) - HTML 이메일
- send_report(recipients, report_type) - 리포트 발송
```

**사용 예시**:
```python
# 승인 요청 이메일
email_mcp.send_email(
    to="engineer@company.com",
    subject="[Urgent] Approval Required - Solution ENV001",
    body=format_approval_email(solution)
)

# 일일 리포트
email_mcp.send_report(
    recipients=["team@company.com"],
    report_type="daily",
    data=generate_daily_stats()
)
```

**구현 방법**:
- Python `smtplib` 사용
- 회사 메일 서버 SMTP 설정

**Agent 기능 매핑**:
- ✅ Approve (승인 요청)
- ✅ Monitor (리포트)

---

## Priority 3: 선택 사항 (UX 및 편의성 향상)

### 7. Web Dashboard API MCP ⭐⭐

**역할**: 웹 대시보드 데이터 제공 및 승인 처리

**필요한 이유**:
- 실시간 대시보드 업데이트
- 웹 UI를 통한 승인 처리
- 성능 메트릭 시각화

**주요 기능**:
```
- update_metrics(data) - 메트릭 업데이트
- get_pending_approvals() - 대기 중인 승인 목록
- process_approval(approval_id, status) - 승인 처리
- get_recent_executions(limit) - 최근 실행 목록
```

**사용 예시**:
```python
# 대시보드 메트릭 업데이트
dashboard_mcp.update_metrics({
    "success_rate": 0.94,
    "automation_rate": 0.82,
    "total_resolved": 156,
    "avg_time_saved": 63  # minutes
})

# 승인 상태 확인
pending = dashboard_mcp.get_pending_approvals()
```

**구현 방법**:
- 회사 웹사이트 REST API 연동
- FastAPI 또는 Flask 백엔드

**Agent 기능 매핑**:
- ✅ Approve (웹 승인)
- ✅ Monitor (대시보드)

---

### 8. Logging/Monitoring MCP ⭐

**역할**: 중앙 집중식 로깅 및 모니터링

**필요한 이유**:
- 구조화된 로그 관리
- 에러 추적
- 성능 모니터링
- 디버깅 지원

**주요 기능**:
```
- log(level, message, metadata) - 로그 기록
- log_error(error, context) - 에러 로그
- log_performance(metric, value) - 성능 로그
- query_logs(filter, timerange) - 로그 조회
```

**사용 예시**:
```python
# 구조화된 로그
logging_mcp.log(
    level="info",
    message="Solution executed successfully",
    metadata={
        "solution_id": "ENV001",
        "duration": 5.2,
        "error_id": "ERR_LIC_001"
    }
)

# 에러 로그
logging_mcp.log_error(
    error=exception,
    context={
        "phase": "execution",
        "solution": solution
    }
)
```

**구현 방법**:
- ELK Stack (Elasticsearch, Logstash, Kibana)
- 또는 Python `logging` + 파일 저장

**Agent 기능 매핑**:
- ✅ 모든 기능 (디버깅 및 추적)

---

## MCP 우선순위 요약

### 🔥 즉시 필요 (Phase 1 - 1주일)
```
1. SSH MCP              - 원격 실행
2. Filesystem MCP       - 로그 읽기
3. MongoDB MCP (보유)   - 데이터 저장
```
→ 이것만으로 기본 프로토타입 가능

### ⭐ 자동화 완성 (Phase 2 - 2주일)
```
4. Messenger MCP        - 승인 요청
5. Email MCP            - 알림 및 리포트
```
→ Human-in-the-loop 완성

### 👍 UX 향상 (Phase 3 - 3주일)
```
6. Dashboard API MCP    - 웹 대시보드
7. Logging MCP          - 중앙 로깅
```
→ 프로덕션 준비 완료

---

## Agent 기능별 MCP 매핑

| Agent 기능 | 필요한 MCP |
|-----------|-----------|
| **1. Detect** (오류 감지) | Filesystem, SSH, OracleDB |
| **2. Match** (솔루션 검색) | MongoDB |
| **3. Evaluate** (평가) | SSH, Filesystem |
| **4. Approve** (승인) | Messenger, Email, Dashboard |
| **5. Execute** (실행) | SSH, Filesystem |
| **6. Monitor** (모니터링) | MongoDB, Dashboard, Logging |

---

## 최소 구성 (프로토타입)

```python
# 1주일 안에 시작하려면 이것만:

class MinimalAgent:
    def __init__(self):
        # 필수만
        self.ssh = SSH_MCP()          # 원격 실행
        self.fs = Filesystem_MCP()    # 로그 읽기
        self.db = MongoDB_MCP()       # 이미 있음

        # 임시 대체
        self.approve = print          # 콘솔 출력
        self.notify = print           # 콘솔 출력
```

→ 이것만으로 기본 동작 가능!

---

## 구현 순서 권장

### Week 1: 핵심 MCP
1. SSH MCP 설정 및 테스트
2. Filesystem MCP 연동
3. MongoDB 스키마 설계

### Week 2: 알림 추가
4. Messenger MCP 구현
5. Email MCP 구현

### Week 3: 대시보드
6. Dashboard API MCP
7. Logging MCP (선택)

---

## 참고 자료

- MCP 공식 문서: https://modelcontextprotocol.io
- MCP 서버 예제: https://github.com/modelcontextprotocol/servers
- Paramiko (SSH): https://www.paramiko.org
- pymongo: https://pymongo.readthedocs.io

---

## 작성 일자
2025-01-01

## 업데이트 이력
- 2025-01-01: 초안 작성
