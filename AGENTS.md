# AGENTS.md

AI 에이전트가 이 프로젝트에서 작업할 때 참고해야 할 학습 내용과 주의사항을 기록합니다.

---

## 실수 기록 및 학습 내용

### 2026-02-04: Cursor CLI 존재 여부 오판

**상황:**
- "Agent CLI와 웹 배포.md" 파일 검토 중 Cursor CLI가 별도의 Agent CLI 도구가 아니라고 잘못 판단
- Cursor는 IDE 내장 AI 기능만 제공한다고 오해하여 문서에서 Cursor CLI를 제거함

**실제 사실:**
- Cursor CLI는 실제로 존재하는 터미널 기반 Agent CLI 도구
- https://www.cursor.com/cli 에서 공식 제공
- `curl https://cursor.com/install -fsS | bash` 명령어로 설치 가능
- 다양한 AI 모델 지원 (Claude, GPT, Gemini, Grok 등)
- Cursor 구독 필요 (유료)

**교훈:**
- 특정 도구나 기능의 존재 여부를 판단할 때, 확신이 없으면 반드시 공식 웹사이트나 문서를 검색하여 확인할 것
- 사용자의 원본 정보를 함부로 삭제하거나 수정하기 전에 검증 필요
