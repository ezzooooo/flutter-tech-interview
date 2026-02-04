# Flutter 기술 면접 준비

[![Since](https://img.shields.io/badge/since-2026.02-333333.svg?style=flat-square)](https://github.com/ezzooooo/flutter-tech-interview)
[![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B.svg?style=flat-square&logo=flutter)](https://flutter.dev)
[![Dart](https://img.shields.io/badge/Dart-3.0+-0175C2.svg?style=flat-square&logo=dart)](https://dart.dev)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-FF6F6F.svg?style=flat-square)](http://makeapullrequest.com)

> Flutter/Dart 개발자를 위한 기술 면접 준비 자료 모음

## 소개

이 레포지토리는 Flutter 개발자 면접을 준비하는 분들을 위한 기술 면접 자료를 정리한 공간입니다.
각 주제별로 면접에서 자주 나오는 질문들과 심화 개념, 그리고 예상 꼬리 질문까지 다루고 있습니다.

---

## 목차

### Flutter

- #### 상태 관리 (State Management)

  - [Flutter 상태(State)란?](./Flutter/State%20Management/Flutter%20상태.md)
    - Ephemeral State vs App State
    - StatelessWidget vs StatefulWidget
    - setState() 동작 원리
    - Provider, Riverpod, Bloc, GetX 비교
  - [Riverpod 상태관리 패턴](./Flutter/State%20Management/Riverpod%20상태관리.md)
    - Riverpod 핵심 특징
    - Provider 종류별 사용법
    - Riverpod 3.0 주요 변경사항
  - [Riverpod vs Provider 비교](./Flutter/State%20Management/Riverpod%20vs%20Provider.md)
    - 핵심 차이점 상세 비교
    - 선택 가이드

- #### 위젯 (Widget)

  - [ListView와 스크롤 위젯 비교](./Flutter/Widget/ListView와%20스크롤%20위젯.md)
    - ListView vs ListView.builder vs SingleChildScrollView + Column
    - 성능 최적화 팁
    - 무한 스크롤 구현

- #### 기타

  - [상수 클래스 (Constants)](./Flutter/상수%20클래스(Constants).md)
    - 상수 클래스의 정의와 사용 이유
    - const vs final vs static
    - 베스트 프랙티스

### Architecture

- [ViewModel의 역할](./Architecture/ViewModel의%20역할.md)
  - MVVM 패턴과 ViewModel
  - ViewModel의 핵심 역할 5가지
  - Flutter에서 ViewModel 구현 방식

---

## 예정된 주제

### Flutter 심화

- [ ] 위젯 생명주기 (Widget Lifecycle)
- [ ] 렌더링 파이프라인 (Rendering Pipeline)
- [ ] Key의 역할과 종류
- [ ] BuildContext 이해하기
- [ ] Navigator 2.0 / go_router
- [ ] 애니메이션 (Animation)
- [ ] 플랫폼 채널 (Platform Channel)
- [ ] 성능 최적화 (Performance Optimization)

### Dart 언어

- [ ] Dart 기초 문법
- [ ] Null Safety
- [ ] Dart 3.0 새로운 기능 (Records, Patterns, Sealed Classes)
- [ ] 비동기 프로그래밍 (Future, Stream, async/await)
- [ ] Isolate와 동시성
- [ ] 제네릭 (Generics)
- [ ] Extension Methods
- [ ] Mixin

### 아키텍처

- [ ] Clean Architecture
- [ ] Repository 패턴
- [ ] 의존성 주입 (Dependency Injection)
- [ ] 테스트 (Unit, Widget, Integration)

### 네트워크 & 데이터

- [ ] REST API 통신 (Dio)
- [ ] GraphQL
- [ ] 로컬 저장소 (SharedPreferences, Hive, SQLite)
- [ ] 직렬화/역직렬화 (JSON Serialization)

---

## 기여 방법

1. 이 레포지토리를 Fork합니다.
2. 새로운 브랜치를 생성합니다. (`git checkout -b feature/new-topic`)
3. 변경사항을 커밋합니다. (`git commit -m 'Add: 새로운 주제 추가'`)
4. 브랜치에 Push합니다. (`git push origin feature/new-topic`)
5. Pull Request를 생성합니다.

### 커밋 컨벤션

```
날짜-[주제]-내용-상태
예) 2026-02-04 [Flutter] State Management Add
```

---

## 폴더 구조

```
flutter-tech-interview/
├── README.md
├── Flutter/
│   ├── State Management/
│   │   ├── Flutter 상태.md
│   │   ├── Riverpod 상태관리.md
│   │   └── Riverpod vs Provider.md
│   ├── Widget/
│   │   └── ListView와 스크롤 위젯.md
│   └── 상수 클래스(Constants).md
├── Architecture/
│   └── ViewModel의 역할.md
├── Dart/                           # 예정
└── Network/                        # 예정
```

---

## 참고 자료

- [Flutter 공식 문서](https://docs.flutter.dev/)
- [Dart 공식 문서](https://dart.dev/guides)
- [Riverpod 공식 문서](https://riverpod.dev/)
- [flutter_bloc 패키지](https://bloclibrary.dev/)
- [Pub.dev](https://pub.dev/)

---

## 라이선스

이 프로젝트는 MIT 라이선스를 따릅니다.
