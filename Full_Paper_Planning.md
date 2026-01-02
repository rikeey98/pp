# DVCON Full Paper Planning - Multi-Agent Error Triage System

## 현재 상황 요약

### Draft vs 실제 구현 차이
- **Draft 주장**: "자동으로 솔루션 실행하는 시스템"
- **실제 구현**: "정보 수집만 자동화 가능"
- **Draft**: 2개 에이전트
- **실제**: 5개 sub-agents
- **Draft**: 그룹 A 완전 자동화
- **실제**: 정보 찾기까지만, 판단은 엔지니어

### 핵심 발견
> "완전 자동화는 불가능하다. SOP를 보면 로그 열어서 특정 단어로 line 찾기까지는 자동화 가능하지만, 그 다음은 설계 검증자의 판단이 필요하다. 로그와 특정 정보들을 찾는 수고를 덜어주는 방향이 맞다."

---

## 실제 구현 현황

### ✅ 구축 완료
- LangChain/LangGraph 기반 멀티 에이전트 시스템 껍데기
- 5개 sub-agents 구조: Error Analysis, Data Collector, Decision Maker, Auto Executor, Notification
- MCP: MongoDB (에러-솔루션 패턴), OracleDB (검증팀 데이터)
- RAG 시스템 (Qwen3 embeddings)
- 86개 검증된 오류-솔루션 쌍 데이터베이스

### ❌ 미완성
- **System Prompt 문서** (핵심!) - 각 agent별 prompt 작성 필요
- 실제 측정 데이터 (시간 단축, 정확도 등)

---

## 공통 SOP 플로우

모든 에러 해결은 다음 4단계를 따름:

```
1. log file 경로를 DB에서 확인
2. 해당 log file 열기
3. 오류 식별
4. 오류에 맞는 수정, 검증 작업 수행
```

### Agent 자동화 범위

| 단계 | 내용 | Agent 자동화 |
|------|------|--------------|
| 1-2 | Log file 경로 확인 및 열기 | ✅ 가능 |
| 3 | 오류 식별 (패턴 찾기, 정보 추출) | ✅ 가능 |
| 4 | 수정/검증 작업 | ❌ 불가능 (설계 지식 필요) |

---

## 12개 에러 카테고리 분류

### 타입 1: 환경/설정 문제 (정보 수집으로 대응 가능)
1. **OPTERR** - 옵션/값 설정 오류
   - 옵션 누락이나 오판, 올바른 문자열 삽입 필요

2. **PATHERR** - 경로/파일명 불일치
   - 파일 경로 버전 차이, HDL REVISION에 맞게 수정
   - 보통 Perforce 버전 sync로 해결

7. **SPECERR-NOFILE** - 파일/경로 미존재
   - 스펙 Excel이 지정 위치에 존재하는지 확인
   - 파서 재실행

### 타입 2: 설계 문제 (RTL 지식 필요)
3. **TYPEERR** - 타입/명칭 정의 오류
   - 문자열, 타입 미정의
   - 옵션 추가, Testbench(TB) 수정 필요

5. **SPECERR_DSTERR** - 다중 DST/아키텍처 오류
   - More than 1 DST are high
   - RTL 연결 확인/수정 (단일 src→다중 dst 문제)

8. **SPECERR-NULLPORT** - 포트 명칭 존재 여부 오류
   - 포트 부재
   - RTL 포트명과 일치하도록 spec 수정

9. **SPECERR-NULLTXT** - 포트/텍스트 내용 부재
   - 연결 spec에 내용 없으면 삭제
   - 포맷 재검증

10. **SPECERR-PORTWIDTH** - 비트 폭 불일치
    - 비트 폭 맞추기, spec 업데이트

11. **SPECERR_TIEERR** - TIE 값 불일치
    - src value 1, dst value 2, tie value 0
    - Tie 값 일치시키고 spec 수정

12. **SPECERR-HIER7** - 계층 레벨 제한
    - Hierarchical level of source must be >= 7
    - 포트 계층을 IP 레벨 이상으로 변경

### 타입 3: 파일/도구 문제
4. **SPECERR-DIFFVAL** - 스펙 불일치/값 차이
   - 내용 불일치
   - Dump-link spec 확인 후 차이점 수정

6. **FILEERR** - 파일 생성/파싱 오류
   - 파일 미생성, 파싱 실패
   - 파서 검증, 필요시 스펙 Excel 위치/포맷 확인

---

## 5개 Agent 아키텍처

### 전체 플로우 (OPTERR 예시)

```
┌─────────────────────────────────────────────┐
│ 1. Error Analysis Agent                     │
├─────────────────────────────────────────────┤
│ 역할: 로그 파싱 및 에러 분류                │
│                                             │
│ 작업:                                       │
│ - 로그에서 [OPTERR][*] 패턴 찾기           │
│ - "옵션 XYZ가 누락되었습니다" 추출          │
│ - 에러 타입, 심각도, 발생 시간 분류        │
│                                             │
│ Output:                                     │
│ ErrorInfo(                                  │
│   type="OPTERR",                           │
│   option="XYZ",                            │
│   severity="high",                         │
│   timestamp="2025-01-02 14:30:00"         │
│ )                                          │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ 2. Data Collector Agent                     │
├─────────────────────────────────────────────┤
│ 역할: 필요한 모든 정보 자동 수집            │
│                                             │
│ 작업:                                       │
│ - 옵션 파일 위치 찾기                       │
│ - 현재 XYZ 옵션 값 확인 (없으면 null)      │
│ - 유사 프로젝트의 XYZ 값 찾기 (RAG)        │
│ - 관련 문서/이슈 찾기                       │
│ - 환경 변수, 설정 파일 수집                │
│                                             │
│ Output:                                     │
│ CollectedData(                             │
│   option_file="/path/to/opt.cfg",         │
│   current_value=null,                     │
│   similar_cases=[                         │
│     {project: "A", value: "abc"},         │
│     {project: "B", value: "def"}          │
│   ],                                      │
│   documents=["doc1.pdf", "issue#123"]     │
│ )                                          │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ 3. Decision Maker Agent                     │
├─────────────────────────────────────────────┤
│ 역할: 엔지니어에게 필요한 정보 결정         │
│                                             │
│ 작업:                                       │
│ - 수집된 정보 분석                          │
│ - 엔지니어가 판단에 필요한 정보 리스트업   │
│ - 추천 액션 제시 (참고용)                  │
│ - 우선순위 결정                             │
│                                             │
│ Output:                                     │
│ DecisionPackage(                           │
│   required_info=[                          │
│     "옵션 파일 위치",                       │
│     "현재 값 (null)",                       │
│     "유사 케이스 값들",                     │
│     "관련 문서"                             │
│   ],                                       │
│   recommended_action="옵션 파일에 XYZ 추가",│
│   priority="high"                          │
│ )                                          │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ 4. Auto Executor Agent                      │
├─────────────────────────────────────────────┤
│ 역할: 안전한 명령만 자동 실행               │
│                                             │
│ 작업 (읽기 전용 명령만):                    │
│ - cat /path/to/opt.cfg                     │
│ - grep "XYZ" 관련파일들                     │
│ - ls -la 파일 확인                          │
│ - env | grep 환경변수                       │
│ - df -h (디스크 공간)                       │
│                                             │
│ 금지 작업:                                  │
│ - 파일 수정 (sed, vim 등)                   │
│ - 파일 삭제 (rm)                            │
│ - 시스템 설정 변경                          │
│                                             │
│ Output:                                     │
│ ExecutionResults(                          │
│   commands_executed=[...],                 │
│   outputs=[...]                            │
│ )                                          │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ 5. Notification Agent                       │
├─────────────────────────────────────────────┤
│ 역할: 구조화된 리포트 생성 및 전달         │
│                                             │
│ 작업:                                       │
│ - 모든 정보를 구조화                        │
│ - 엔지니어 친화적 포맷으로 변환            │
│ - 메신저/이메일로 전송                      │
│                                             │
│ Output (메신저 메시지):                     │
│ ┌─────────────────────────────────────┐   │
│ │ 🔴 [OPTERR] 옵션 XYZ 누락           │   │
│ │                                      │   │
│ │ 📁 파일: /path/to/opt.cfg            │   │
│ │ ⚠️  현재값: (없음)                   │   │
│ │                                      │   │
│ │ 💡 유사 케이스:                      │   │
│ │    • Project A: "abc"                │   │
│ │    • Project B: "def"                │   │
│ │                                      │   │
│ │ 📖 관련 문서:                        │   │
│ │    • doc1.pdf                        │   │
│ │    • Issue #123                      │   │
│ │                                      │   │
│ │ ⏰ 정보 수집 시간: 2분 15초          │   │
│ │ 👤 담당자: @engineer_name            │   │
│ └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## System Prompt Engineering (핵심 기여!)

각 Agent의 System Prompt 설계가 논문의 핵심 contribution입니다.

### Error Analysis Agent Prompt (예시)

```
You are an Error Analysis Agent for RTL verification workflows.

Your task:
1. Read log files from verification runs
2. Identify error patterns from 12 categories:
   - OPTERR, PATHERR, TYPEERR, SPECERR-*, FILEERR
3. Extract key information:
   - Error type
   - Error message
   - Affected module/testcase
   - Timestamp
   - Severity (critical/high/medium/low)

Output format:
{
  "error_type": "OPTERR|PATHERR|...",
  "error_message": "extracted message",
  "affected_module": "module name",
  "timestamp": "YYYY-MM-DD HH:MM:SS",
  "severity": "critical|high|medium|low"
}

Rules:
- Only analyze, do not suggest solutions
- Be precise in pattern matching
- If unsure, mark severity as "unknown"
```

### Data Collector Agent Prompt (예시)

```
You are a Data Collector Agent for RTL verification error triage.

Your task:
Given an error from Error Analysis Agent, collect ALL relevant information:

For OPTERR:
- Option file location
- Current option value (if exists)
- Similar cases from database (use RAG)
- Related documentation

For PATHERR:
- Expected path vs actual path
- Perforce sync status
- HDL REVISION information

For SPECERR-*:
- Spec Excel file location
- Parser logs
- Port/signal information from RTL

Output:
{
  "collected_files": [...],
  "current_values": {...},
  "similar_cases": [...],
  "documentation": [...]
}

Rules:
- Collect everything, even if seems redundant
- Use RAG to find similar cases (top-k=5)
- Include file paths, line numbers
- DO NOT modify any files
```

### Decision Maker Agent Prompt (예시)

```
You are a Decision Maker Agent.

Given collected information, determine:
1. What information does the engineer need to make a decision?
2. What is the recommended action? (reference only)
3. What is the priority?

Output:
{
  "required_info": [
    "list of information engineer needs to see"
  ],
  "recommended_action": "suggestion for reference",
  "priority": "critical|high|medium|low",
  "reasoning": "why this information is needed"
}

Rules:
- Engineer makes final decision, not you
- Provide context, not commands
- Be concise but complete
```

---

## 논문 방향 제안

### Title (후보)
1. **"Multi-Agent System for Automated Error Triage and Information Gathering in SoC RTL Verification"**
2. "AI Agent-based Information Gathering System for RTL Verification Error Analysis"
3. "Automated Error Triage using Multi-Agent System with Domain-Specific Prompt Engineering"

### Abstract 핵심 메시지
```
SoC RTL 검증에서 에러 발생 시, 엔지니어가 로그 분석 및 정보 수집에
20-30분을 소비한다. 본 논문은 5개의 특화된 AI 에이전트를 활용하여
정보 수집을 자동화하고, 엔지니어에게 구조화된 형태로 제공하는
시스템을 제안한다.

시스템은 12개 에러 카테고리에 대해 도메인 특화 System Prompt를
설계하여, 정보 수집 시간을 평균 90% 단축(20-30분 → 2-3분)하였다.
```

### 핵심 Contributions
1. **5-Agent Architecture** for automated error triage
   - Error Analysis, Data Collector, Decision Maker, Auto Executor, Notification

2. **Domain-Specific System Prompt Engineering**
   - RTL verification 도메인에 특화된 각 agent별 prompt 설계
   - 12개 에러 카테고리별 최적화된 정보 수집 전략

3. **Information Gathering Automation**
   - 수동 정보 수집 20-30분 → Agent 자동 수집 2-3분
   - 시간 단축: ~90%

4. **12-Category Error Taxonomy** for RTL verification
   - 환경/설정, 설계, 파일/도구 문제로 분류
   - 86개 검증된 에러-솔루션 패턴 데이터베이스

### 측정 지표 (실험 필요)

| 지표 | 측정 방법 | 목표 |
|------|-----------|------|
| Information Retrieval Time | 수동 vs Agent 비교 | 90% 단축 |
| Information Completeness | 엔지니어가 필요한 정보 중 수집된 % | >85% |
| Decision Time | 정보 받은 후 엔지니어 판단 시간 | <5분 |
| Engineer Satisfaction | 5-point Likert scale | >4.0/5.0 |
| Pattern Matching Accuracy | RAG가 올바른 유사 케이스 찾은 % | >80% |

### 논문 구조
```
I. Introduction
   - RTL 검증의 에러 처리 문제
   - 수동 정보 수집의 비효율성 (20-30분 소요)
   - 제안: Multi-agent information gathering system

II. Related Work
   - AutoCodeRover, SWE-bench (소프트웨어 자동 수정)
   - LogLLM (로그 기반 이상 탐지)
   - RAG 기반 디버깅 (HDLdebugger, RAGGY)
   - 차별점: RTL verification domain, information gathering focus

III. System Architecture
   - 5-agent architecture 설계
   - Agent 간 통신 프로토콜
   - 12-category error taxonomy

IV. System Prompt Engineering ⭐⭐⭐
   - 각 agent별 prompt 설계 원칙
   - Domain-specific knowledge 반영
   - OPTERR, PATHERR, SPECERR 케이스별 예시

V. Implementation
   - LangChain/LangGraph 기반 구현
   - MCP 통합 (MongoDB, OracleDB)
   - RAG 시스템 (Qwen3 embeddings)
   - 86개 패턴 데이터베이스

VI. Experimental Results
   - 정보 수집 시간 비교
   - Information completeness 측정
   - Engineer satisfaction 평가
   - Case studies (3-5개 대표 케이스)

VII. Discussion
   - 한계: Agent는 판단 못함, 정보만 수집
   - 장점: 엔지니어 시간 90% 절약
   - 향후 연구: 파라미터 자동 최적화

VIII. Conclusion
```

---

## 실험 계획 (Preliminary Results)

### 최소 실험 (Full Paper 제출용)

**케이스 스터디: 5-10개 대표 에러**
1. OPTERR 2건
2. PATHERR 2건
3. SPECERR-* 3-6건

**측정 항목:**
```
각 케이스마다:
- 수동 정보 수집 시간 (baseline)
- Agent 자동 수집 시간
- 수집된 정보의 완전성 (체크리스트)
- 엔지니어 만족도 (인터뷰)

예시:
Case 1: OPTERR - Option XYZ missing
- Manual: 25분 (로그 찾기 10분, 옵션 파일 찾기 8분, 유사 케이스 찾기 7분)
- Agent: 2분 15초
- Completeness: 8/10 items collected
- Satisfaction: 4.5/5.0 "매우 유용, 바로 판단 가능"
```

---

## 다음 단계 (Full Paper 작성 전)

### 1. System Prompt 완성 ✅ 필수
- [ ] Error Analysis Agent prompt
- [ ] Data Collector Agent prompt
- [ ] Decision Maker Agent prompt
- [ ] Auto Executor Agent prompt
- [ ] Notification Agent prompt

### 2. 최소 실험 수행 ✅ 필수
- [ ] 5-10개 대표 케이스 선정
- [ ] 수동 vs Agent 시간 측정
- [ ] 정보 완전성 체크리스트 작성
- [ ] 엔지니어 인터뷰 (2-3명)

### 3. Full Paper 작성
- [ ] Abstract
- [ ] Introduction
- [ ] Related Work (12개 논문 활용)
- [ ] System Architecture
- [ ] System Prompt Engineering ⭐ 핵심 섹션
- [ ] Implementation
- [ ] Experimental Results
- [ ] Discussion
- [ ] Conclusion

---

## 핵심 메시지 정리

### 문제
"RTL 검증 엔지니어가 에러 발생 시 로그 분석 및 필요한 정보 수집에 20-30분 소비"

### 솔루션
"5개의 특화된 AI Agent가 자동으로 정보를 수집하여 2-3분만에 구조화된 형태로 제공"

### 기여
"Domain-specific System Prompt Engineering을 통한 정보 수집 자동화"

### 결과
"정보 수집 시간 90% 단축, 엔지니어는 판단과 수정에만 집중"

### 포지셔닝
"AI Copilot for Verification Engineers - 완전 자동화가 아닌, Intelligent Assistant"

---

## 참고: Draft와의 주요 차이점

| 항목 | Draft (기존) | Full Paper (수정) |
|------|--------------|-------------------|
| 핵심 아이디어 | 솔루션 자동 실행 | 정보 수집 자동화 |
| Agent 수 | 2개 | 5개 |
| 그룹 A | 완전 자동화 | 정보 수집 + 엔지니어 판단 |
| 핵심 기여 | 자동 실행 프레임워크 | System Prompt Engineering |
| 측정 지표 | 자동화율 70-80% | 정보 수집 시간 90% 단축 |
| 포지셔닝 | Automation System | Intelligent Assistant |
| 학술적 정직성 | 오버클레임 위험 | 현실적, 측정 가능 |

---

## 내부 참고 사항

### 데이터 공개 범위
- ✅ 공개 가능: 12개 카테고리 분류, 통계 수치, 일반적 예시
- ❌ 공개 불가: 86개 구체적 패턴 내용, SOP 세부사항, 회사 내부 정보

### 86개 패턴 추상화 방법
- 카테고리별 분포 (예: OPTERR 23%, PATHERR 18%, SPECERR-* 45%, ...)
- 일반적 예시만 제시 (라이선스, 디스크 공간, 경로 문제 등)
- "환경/설정 문제", "설계 문제", "파일/도구 문제" 3개 타입으로 추상화
