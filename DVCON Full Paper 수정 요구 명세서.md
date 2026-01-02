DVCON Full Paper 수정 요구 명세서
1. 전체 방향
항목요구사항작성 스타일나열식(bullet points) → 문장식(prose)으로 변경, 사람이 작성한 것처럼 자연스럽게분량불필요한 내용 축소, 핵심에 집중표절 방지다른 논문 표현과 차별화, 독자적 서술기밀 보호회사 내부 정보, 구체적 기술 스택 노출 최소화

2. 섹션별 수정 사항
Section III. System Architecture
항목현재변경기술 스택"MongoDB", "OracleDB" 명시"pattern database", "specification database" 등 일반화된 표현MCP 설명"Custom MCP servers for MongoDB access, OracleDB queries""Custom MCP servers for database access and file system operations"추가텍스트만 존재시스템 구조 그림(Figure) 추가

Section IV. System Prompt Engineering
항목현재변경분량~2 페이지0.75~1 페이지로 축소상세도구체적 prompt snippet, command 예시 포함설계 원칙과 개념 수준만 서술
제거할 내용:

상세 prompt excerpt (코드 블록)
Category별 세부 instruction
구체적 command 예시 (cat, grep, find 등)
Database query 예시 (MongoDB, OracleDB)

유지할 내용:

4가지 설계 원칙 (Domain Specificity, Structured Output, Safety Constraints, Context Preservation)
Agent별 역할 요약 (1-2문장씩)
Output JSON 구조 (간략히)


Section V. Implementation and Case Study Validation
항목현재변경비용 추정"$100/hour", "$694,000 annual savings"전체 제거표현비용 절감 금액 명시시간 절감(hours)만 표현
제거할 문장:

"At average engineer cost $100/hour: ~$694,000 annual savings"
"translating to approximately $694,000 in annual savings"


Section VI. Discussion
항목현재변경구조A. Key Insights / B. Comparison with Related Work / C. Practical Deployment / D. Limitations and Future WorkA. Key Insights / B. Deployment Considerations and Future Directions비교 섹션"vs. AutoCodeRover", "vs. HDLdebugger", "vs. LogLLM"전체 제거 (Section II에서 이미 다룸)파일럿 언급"3-month pilot deployment"제거작성 스타일나열식 (bullet points)문장식 (prose)

Section VII. Conclusion & Abstract
항목현재변경비용 언급"$694K annual savings"제거수치 표현"88.7% time reduction" (단정적)"preliminary case studies show potential time reduction" (보수적)

3. 추가 작업
시스템 구조 그림 (Figure 1)
포함 요소:

5개 Agent 흐름 (Error Analyzer → Data Collector → Decision Maker → Auto Executor → Notification)
외부 연동: Pattern Database, Specification Database, File System
Engineer 피드백 루프

스타일:

간결한 블록 다이어그램
화살표로 데이터 흐름 표시
흑백 호환 가능하게


4. 작성 스타일 가이드
현재 (나열식)변경 (문장식)**Challenges:** 1. Pattern database... 2. Prompt maintenance... 3. Trust building...Deploying this system requires careful consideration of several factors. The initial construction of the pattern database demands significant domain expertise, as each pattern must be manually curated. Additionally, system prompts require ongoing maintenance as tools evolve.

5. 수정 우선순위

Section IV 축소 (prompt 상세 내용 제거)
Section VI 재구성 (비교 제거, 통합, 문장식 변환)
Section V 비용 관련 제거
Section III DB명 일반화 + 그림 추가
Abstract, Conclusion 비용/수치 표현 수정
전체 문장 다듬기, 표절 검토


6. 최종 확인 체크리스트

 MongoDB, OracleDB 직접 언급 제거
 Prompt 상세 코드/예시 제거
 비용($) 관련 내용 전체 제거
 Related work 비교 섹션 제거
 3-month pilot 언급 제거
 나열식 → 문장식 변환
 시스템 구조 그림 추가
 표절 우려 표현 검토 및 수정
