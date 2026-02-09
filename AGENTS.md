# AGENTS.md

AI 에이전트가 이 프로젝트에서 작업할 때 참고해야 할 가이드라인입니다.

---

## 프로젝트 개요

Flutter/Dart 개발자를 위한 **기술 면접 준비 자료** 레포지토리입니다.
각 주제별로 면접에서 자주 나오는 질문들과 심화 개념, 예상 꼬리 질문까지 다루고 있습니다.

### 폴더 구조

```
flutter-tech-interview/
├── README.md                          # 프로젝트 소개 및 목차
├── AGENTS.md                          # AI 에이전트 가이드라인 (이 파일)
├── Flutter/
│   ├── State Management/              # 상태 관리
│   │   ├── Flutter 상태.md            # State 기본 개념, 상태 관리 솔루션 비교
│   │   ├── Riverpod 상태관리.md       # Riverpod 핵심 특징, Provider 종류, 3.0 변경사항
│   │   └── Riverpod vs Provider.md    # 두 라이브러리 상세 비교
│   ├── Widget/
│   │   └── ListView와 스크롤 위젯.md  # ListView, ListView.builder, SingleChildScrollView 비교
│   ├── 상수 클래스(Constants).md      # const/final/static, 상수 클래스 패턴
│   ├── 위젯 트리와 렌더링 파이프라인.md  # 3가지 트리, 렌더링 5단계
│   └── BuildContext.md                # BuildContext 정의, of 패턴, 주의사항
├── Architecture/
│   ├── ViewModel의 역할.md            # MVVM 패턴, ViewModel 핵심 역할 5가지
│   └── MVVM 패턴.md                   # MVVM 정의, 특징/장점, StatefulWidget 비교
├── Lesson/
│   └── Agent CLI와 웹 배포.md         # Agent CLI 도구들, Flutter/Next.js 웹 배포
└── images/                            # 이미지 리소스
```

---

## 문서 작성 규칙

### 파일 형식

- 모든 문서는 **Markdown (.md)** 형식으로 작성
- 파일명은 **한글**로 주제를 명확히 표현 (예: `Flutter 상태.md`, `ViewModel의 역할.md`)
- 파일명에 영문 기술 용어가 포함될 수 있음 (예: `Riverpod vs Provider.md`, `ListView와 스크롤 위젯.md`)

### 문서 구조 (면접 답변 가이드)

각 면접 주제 문서는 다음 구조를 따릅니다:

1. **제목**: `# [주제명] - 면접 답변 가이드`
2. **메타 정보**: 최종 업데이트 날짜, 적용 버전 (blockquote 형식)
3. **핵심 질문**: 면접에서 물어볼 핵심 질문 (blockquote 강조)
4. **개념 설명**: 정의, 비교표, 다이어그램 포함
5. **코드 예시**: Dart/Flutter 코드 블록 (✅ 올바른 예, ❌ 잘못된 예 패턴 활용)
6. **면접 예상 질문**: `Q1~Qn` 형식으로 질문-답변 쌍, **꼬리질문 포함** (아래 규칙 참조)
7. **참고 자료**: 공식 문서 및 관련 링크

### 면접 예상 질문 및 꼬리질문 작성 규칙

면접 예상 질문 섹션은 실제 면접 상황을 시뮬레이션하여 **꼬리질문(Follow-up Questions)까지 상세하게 작성**합니다:

- 각 핵심 질문(Q1, Q2 ...)에 대해 **상세한 답변**을 작성
- 각 답변 뒤에 면접관이 이어서 물어볼 수 있는 **꼬리질문 2~4개**를 추가
- 꼬리질문은 `#### 꼬리질문: "질문 내용"` 형식으로 작성
- 꼬리질문의 답변도 핵심 질문과 동일한 수준으로 **구체적이고 실용적으로** 작성
- 가능하면 꼬리질문 답변에도 **코드 예시** 포함
- 꼬리질문은 다음 방향으로 구성:
  - **심화 질문**: 개념의 더 깊은 이해를 묻는 질문
  - **비교 질문**: 유사 개념과의 차이를 묻는 질문
  - **실무 질문**: 실제 개발에서의 활용을 묻는 질문
  - **문제 해결 질문**: 해당 개념에서 자주 발생하는 문제와 해결법

### 메타 정보 형식

```markdown
> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Flutter 3.x, Dart 3.0+
```

### 코드 예시 규칙

- 모든 코드는 Dart 코드 블록 사용
- **좋은 예시**에는 `// ✅` 주석, **나쁜 예시**에는 `// ❌` 주석 사용
- 실무에서 바로 활용 가능한 수준의 예시 코드 작성
- 코드에 한글 주석으로 핵심 포인트 설명

### 비교표 규칙

- 주요 개념 비교 시 **Markdown 테이블** 사용
- 지원 여부 표시: ✅ (지원), ❌ (미지원)
- 간결하고 한눈에 비교 가능하도록 작성

### 언어 규칙

- 본문은 **한국어**로 작성
- 기술 용어는 원문(영어) 유지 (예: BuildContext, InheritedWidget, setState)
- 필요시 영어 용어 뒤에 한글 설명 병기 (예: "Ephemeral State (임시 상태)")

---

## 기술적 맥락

### 다루는 기술 버전

- **Flutter**: 3.x
- **Dart**: 3.0+ (sealed class, records, patterns 등 최신 기능 반영)
- **Riverpod**: 3.0 (2025년 9월 출시, 레거시 분리, 자동 재시도 등)
- **flutter_bloc**: 8.x
- **Provider**: 6.x

### 현재 작성된 주제

- [x] Flutter 상태(State) 기본 개념 및 상태 관리 솔루션 비교 (Provider, Riverpod, Bloc, GetX)
- [x] Riverpod 상태관리 패턴 (핵심 특징, Provider 종류, 3.0 변경사항)
- [x] Riverpod vs Provider 비교 (핵심 차이점, 마이그레이션 가이드)
- [x] ListView와 스크롤 위젯 비교 (ListView, ListView.builder, SingleChildScrollView)
- [x] 상수 클래스 (Constants) (const/final/static, 베스트 프랙티스)
- [x] 위젯 트리와 렌더링 파이프라인 (3가지 트리, 렌더링 5단계, 최적화)
- [x] BuildContext (정의, of 패턴, 주의사항)
- [x] ViewModel의 역할 (MVVM 패턴, Flutter 구현 방식)
- [x] MVVM 패턴 (정의, 특징/장점, StatefulWidget 비교)
- [x] Agent CLI와 웹 배포 (Agent CLI 도구들, Flutter/Next.js 배포)

### 예정된 주제 (README.md 참조)

- Flutter 심화: 위젯 생명주기, 렌더링 파이프라인, Key, BuildContext, Navigator 2.0, 애니메이션 등
- Dart 언어: Null Safety, Dart 3.0 기능, 비동기 프로그래밍, Isolate, 제네릭 등
- 아키텍처: Clean Architecture, Repository 패턴, DI, 테스트
- 네트워크 & 데이터: REST API, GraphQL, 로컬 저장소, JSON 직렬화

---

## 커밋 컨벤션

```
날짜-[주제]-내용-상태
예) 2026-02-04 [Flutter] State Management Add
```

---

## 주의사항

### 문서 수정 시 유의할 점

- **문서 작성/수정 완료 후** 반드시 README.md도 함께 업데이트할 것
  - 새로운 주제 추가 시: 목차에 링크 추가, 폴더 구조 섹션 업데이트, 예정된 주제에서 체크 표시
  - 기존 주제 수정 시: 목차의 하위 항목이 변경되었다면 반영
- **기존 문서 수정 시** 상단의 `최종 업데이트` 날짜를 갱신
- 기술 버전이 업데이트되면 해당 내용을 문서에 반영하고 버전 정보 수정
- Riverpod 3.0 이전의 레거시 API(`StateNotifierProvider`, `StateProvider` 등)는 명시적으로 레거시임을 표시
- 면접 예상 질문은 실제 면접에서 나올 수 있는 수준으로 작성하고, 답변은 구체적이고 실용적으로 작성

### 사용자 원본 데이터 보호

- 사용자가 작성한 원본 정보를 함부로 삭제하거나 수정하지 않기
- 특정 도구나 기능의 존재 여부를 판단할 때, 확신이 없으면 반드시 공식 웹사이트나 문서를 검색하여 확인

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
