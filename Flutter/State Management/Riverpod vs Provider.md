# Riverpod vs Provider 비교 - 면접 답변 가이드

> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Provider 6.x, Riverpod 3.0, Flutter 3.x, Dart 3.0+

## 핵심 질문

> **"Riverpod 상태관리와 Provider 상태관리의 차이는 무엇인가요?"**

---

## 1. 한눈에 보는 비교표

| 특성 | Provider | Riverpod |
|------|----------|----------|
| **BuildContext 의존성** | 필요 | 불필요 |
| **컴파일 타임 안전성** | ❌ (런타임 에러) | ✅ (컴파일 에러) |
| **동일 타입 여러 Provider** | 어려움 | 쉬움 |
| **Provider 간 의존성** | 수동 관리 | 자동 관리 |
| **테스트 용이성** | 보통 | 매우 좋음 |
| **학습 곡선** | 낮음 | 중간 |
| **공식 지원** | Flutter 팀 권장 | 커뮤니티 |
| **InheritedWidget 기반** | ✅ | ❌ |
| **코드 생성** | 없음 | 선택적 지원 |

---

## 2. Provider란?

### 개요

Provider는 Flutter 팀이 공식으로 권장하는 상태 관리 솔루션으로, **InheritedWidget**을 기반으로 구현되었습니다. 간단하고 직관적인 API로 Flutter 개발자들에게 널리 사용됩니다.

### 기본 사용법

```dart
// 1. 상태 클래스 정의
class Counter with ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();  // UI 업데이트 트리거
  }
}

// 2. Provider 등록
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => Counter(),
      child: const MyApp(),
    ),
  );
}

// 3. 상태 사용
class CounterWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // watch: 값 변경 시 리빌드
    final count = context.watch<Counter>().count;
    
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          // read: 한 번만 읽기
          onPressed: () => context.read<Counter>().increment(),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### Provider의 장점

1. **Flutter 공식 권장**: 안정성과 지속적인 지원 보장
2. **낮은 학습 곡선**: InheritedWidget 기반으로 Flutter 개념과 일치
3. **최소한의 보일러플레이트**: 간단한 설정으로 빠르게 시작
4. **풍부한 생태계**: 많은 예제와 커뮤니티 자료

---

## 3. Riverpod이란?

### 개요

Riverpod은 Provider의 저자가 Provider의 한계를 극복하기 위해 **처음부터 새롭게 설계**한 상태 관리 프레임워크입니다. InheritedWidget에 의존하지 않아 더 유연하고 강력한 기능을 제공합니다.

### 기본 사용법

```dart
// 1. Provider 정의 (전역)
final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;  // 초기값
  
  void increment() => state++;
}

// 2. ProviderScope 설정
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

// 3. 상태 사용
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // watch: 값 변경 시 리빌드
    final count = ref.watch(counterProvider);
    
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          // read: 한 번만 읽기
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### Riverpod의 장점

1. **컴파일 타임 안전성**: 존재하지 않는 Provider 참조 시 컴파일 에러
2. **BuildContext 불필요**: 어디서든 Provider 접근 가능
3. **자동 의존성 관리**: Provider 간 의존성 자동 추적
4. **뛰어난 테스트 용이성**: 쉬운 Provider 오버라이드

---

## 4. 핵심 차이점 상세 비교

### 4.1 BuildContext 의존성

**Provider:**
```dart
class MyService {
  // ❌ BuildContext 없이는 Provider 접근 불가
  void doSomething(BuildContext context) {
    final counter = context.read<Counter>();
    counter.increment();
  }
}

// 서비스 레이어에서 사용하려면 context를 전달해야 함
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => MyService().doSomething(context),
      child: const Text('Do Something'),
    );
  }
}
```

**Riverpod:**
```dart
class MyService {
  final Ref ref;
  MyService(this.ref);
  
  // ✅ BuildContext 없이도 Provider 접근 가능
  void doSomething() {
    ref.read(counterProvider.notifier).increment();
  }
}

// Provider로 서비스 정의
final myServiceProvider = Provider((ref) => MyService(ref));

// 어디서든 사용
ref.read(myServiceProvider).doSomething();
```

### 4.2 컴파일 타임 안전성

**Provider:**
```dart
// ❌ 런타임 에러 - Provider를 찾을 수 없음
// ProviderNotFoundException: Could not find the correct Provider<MyNotifier>
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // MyNotifier가 등록되지 않았어도 컴파일은 성공
    // 실행 시점에 에러 발생!
    final notifier = context.watch<MyNotifier>();
    return Text('${notifier.value}');
  }
}
```

**Riverpod:**
```dart
// ✅ 컴파일 에러 - 존재하지 않는 Provider
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // nonExistentProvider가 정의되지 않았으면 컴파일 에러!
    final value = ref.watch(nonExistentProvider);  // ❌ 컴파일 에러
    return Text('$value');
  }
}
```

### 4.3 동일 타입 여러 Provider

**Provider:**
```dart
// ❌ 동일 타입 구분 어려움
// 어떤 Counter가 반환될지 불명확
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = context.watch<Counter>();  // 어떤 Counter?
    return Text('${counter.count}');
  }
}

// 해결책: 별도의 타입 정의 필요
class FirstCounter extends Counter {}
class SecondCounter extends Counter {}
```

**Riverpod:**
```dart
// ✅ 동일 타입도 쉽게 구분
final firstCounterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

final secondCounterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

// 명확하게 구분하여 사용
final first = ref.watch(firstCounterProvider);
final second = ref.watch(secondCounterProvider);
```

### 4.4 Provider 간 의존성 관리

**Provider:**
```dart
// Provider에서 다른 Provider에 의존하려면
// ProxyProvider 또는 수동 관리 필요
MultiProvider(
  providers: [
    Provider<ApiClient>(create: (_) => ApiClient()),
    // ApiClient에 의존하는 Repository
    ProxyProvider<ApiClient, UserRepository>(
      update: (_, api, __) => UserRepository(api),
    ),
    // UserRepository에 의존하는 Service
    ProxyProvider<UserRepository, UserService>(
      update: (_, repo, __) => UserService(repo),
    ),
  ],
  child: MyApp(),
)
```

**Riverpod:**
```dart
// 의존성이 자동으로 추적되고 관리됨
final apiClientProvider = Provider((ref) => ApiClient());

final userRepositoryProvider = Provider((ref) {
  final api = ref.watch(apiClientProvider);  // 자동 의존성 추적
  return UserRepository(api);
});

final userServiceProvider = Provider((ref) {
  final repo = ref.watch(userRepositoryProvider);  // 자동 의존성 추적
  return UserService(repo);
});

// apiClientProvider 변경 시 → userRepositoryProvider 재계산
// → userServiceProvider 재계산 (자동!)
```

### 4.5 테스트 용이성

**Provider:**
```dart
testWidgets('counter test', (tester) async {
  await tester.pumpWidget(
    // 테스트용 Mock 주입
    ChangeNotifierProvider<Counter>.value(
      value: MockCounter(),
      child: const MyApp(),
    ),
  );
});

// MultiProvider 사용 시 더 복잡해짐
```

**Riverpod:**
```dart
testWidgets('counter test', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        // 간단한 오버라이드
        counterProvider.overrideWith(() => MockCounterNotifier()),
        userRepositoryProvider.overrideWithValue(MockUserRepository()),
      ],
      child: const MyApp(),
    ),
  );
});

// 유닛 테스트도 쉬움
test('counter increments', () {
  final container = ProviderContainer();
  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);
  container.dispose();
});
```

---

## 5. 상태 업데이트 방식 비교

### Provider (ChangeNotifier)

```dart
class TodoList with ChangeNotifier {
  final List<Todo> _todos = [];
  List<Todo> get todos => List.unmodifiable(_todos);
  
  void add(Todo todo) {
    _todos.add(todo);
    notifyListeners();  // 수동으로 알림
  }
  
  void remove(String id) {
    _todos.removeWhere((t) => t.id == id);
    notifyListeners();  // 수동으로 알림
  }
  
  void toggle(String id) {
    final index = _todos.indexWhere((t) => t.id == id);
    _todos[index] = _todos[index].copyWith(
      completed: !_todos[index].completed,
    );
    notifyListeners();  // 수동으로 알림
  }
}
```

### Riverpod (Notifier)

```dart
final todoListProvider = NotifierProvider<TodoListNotifier, List<Todo>>(
  TodoListNotifier.new,
);

class TodoListNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];  // 초기값
  
  void add(Todo todo) {
    state = [...state, todo];  // 불변 업데이트, 자동 알림
  }
  
  void remove(String id) {
    state = state.where((t) => t.id != id).toList();
  }
  
  void toggle(String id) {
    state = state.map((t) {
      if (t.id == id) {
        return t.copyWith(completed: !t.completed);
      }
      return t;
    }).toList();
  }
}
```

---

## 6. 비동기 처리 비교

### Provider

```dart
class UserNotifier with ChangeNotifier {
  User? _user;
  bool _isLoading = false;
  String? _error;
  
  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;
  
  Future<void> fetchUser() async {
    _isLoading = true;
    _error = null;
    notifyListeners();
    
    try {
      _user = await api.getUser();
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}

// 사용
Consumer<UserNotifier>(
  builder: (context, notifier, child) {
    if (notifier.isLoading) {
      return const CircularProgressIndicator();
    }
    if (notifier.error != null) {
      return Text('Error: ${notifier.error}');
    }
    return Text('Hello, ${notifier.user?.name}');
  },
)
```

### Riverpod

```dart
final userProvider = AsyncNotifierProvider<UserNotifier, User>(
  UserNotifier.new,
);

class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return await api.getUser();  // 자동으로 로딩/에러 상태 관리
  }
  
  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => api.getUser());
  }
}

// 사용 - AsyncValue로 깔끔한 처리
Consumer(
  builder: (context, ref, child) {
    final userAsync = ref.watch(userProvider);
    
    return userAsync.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
      data: (user) => Text('Hello, ${user.name}'),
    );
  },
)
```

---

## 7. 실전 선택 가이드

### Provider를 선택해야 하는 경우

- 소규모 프로젝트 또는 간단한 상태 관리
- Flutter 초보자 또는 팀의 학습 시간이 제한적인 경우
- 공식 지원과 안정성이 중요한 경우
- 기존 InheritedWidget 기반 코드와 통합 필요 시

### Riverpod을 선택해야 하는 경우

- 중~대규모 프로젝트
- 복잡한 Provider 의존성 관리가 필요한 경우
- 테스트 커버리지가 중요한 경우
- BuildContext 없이 상태 접근이 필요한 경우 (서비스 레이어 등)
- 컴파일 타임 안전성이 중요한 경우
- 동일 타입의 여러 Provider가 필요한 경우

---

## 8. 면접 예상 질문

### Q1: "Provider에서 Riverpod으로 마이그레이션하는 이유는?"

**답변**: 주요 이유는 세 가지입니다:
1. **컴파일 타임 안전성**: Provider는 런타임에 ProviderNotFoundException이 발생할 수 있지만, Riverpod은 컴파일 시점에 에러를 잡습니다.
2. **의존성 관리**: Riverpod은 ref.watch를 통해 Provider 간 의존성을 자동으로 추적하고 관리합니다. Provider에서는 ProxyProvider나 수동 관리가 필요합니다.
3. **테스트 용이성**: Riverpod은 ProviderScope의 overrides로 쉽게 Mock을 주입할 수 있어 테스트 작성이 훨씬 간편합니다.

### Q2: "두 라이브러리의 내부 구현 차이는?"

**답변**: 
- **Provider**: Flutter의 InheritedWidget을 기반으로 구현되어 위젯 트리에 의존합니다. BuildContext를 통해 상위 위젯 트리에서 Provider를 찾습니다.
- **Riverpod**: InheritedWidget을 사용하지 않고 자체적인 상태 관리 시스템을 구현했습니다. 전역 Provider 정의와 ProviderContainer를 통해 상태를 관리하므로 위젯 트리에 독립적입니다.

### Q3: "Provider의 context.watch와 Riverpod의 ref.watch의 차이는?"

**답변**: 
- **context.watch**: BuildContext가 필요하며, build 메서드 내에서만 사용 가능합니다. InheritedWidget의 dependOnInheritedWidgetOfExactType을 내부적으로 사용합니다.
- **ref.watch**: BuildContext 불필요하며, ConsumerWidget의 build 메서드나 Provider 정의 내에서 사용합니다. Riverpod 자체의 의존성 추적 시스템을 사용합니다.

```dart
// Provider
Widget build(BuildContext context) {
  final value = context.watch<MyNotifier>().value;  // context 필요
}

// Riverpod
Widget build(BuildContext context, WidgetRef ref) {
  final value = ref.watch(myProvider);  // context 불필요
}
```

### Q4: "기존 Provider 프로젝트를 Riverpod으로 마이그레이션하는 방법은?"

**답변**: 점진적 마이그레이션을 권장합니다:

1. **의존성 추가**: pubspec.yaml에 flutter_riverpod 추가
2. **ProviderScope 설정**: 앱 루트에 ProviderScope 추가 (기존 Provider와 공존 가능)
3. **새로운 기능은 Riverpod으로**: 새로 작성하는 코드는 Riverpod 사용
4. **점진적 변환**: 시간이 허락할 때 기존 ChangeNotifier를 Notifier로 변환

```dart
// 공존 가능
void main() {
  runApp(
    ProviderScope(  // Riverpod
      child: MultiProvider(  // Provider
        providers: [
          ChangeNotifierProvider(create: (_) => LegacyCounter()),
        ],
        child: const MyApp(),
      ),
    ),
  );
}
```

### Q5: "ChangeNotifier와 Notifier의 차이점은?"

**답변**:
- **ChangeNotifier** (Provider): `notifyListeners()`를 수동으로 호출해야 하며, 가변 상태를 직접 수정합니다.
- **Notifier** (Riverpod): `state`에 새 값을 할당하면 자동으로 구독자에게 알림이 갑니다. 불변 상태 패턴을 권장합니다.

```dart
// ChangeNotifier - 수동 알림
class Counter with ChangeNotifier {
  int _count = 0;
  void increment() {
    _count++;
    notifyListeners();  // 수동!
  }
}

// Notifier - 자동 알림
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;
  void increment() => state++;  // 자동!
}
```

### Q6: "두 라이브러리의 성능 차이는?"

**답변**: 일반적인 사용에서 성능 차이는 미미합니다. 하지만 특정 상황에서 Riverpod이 유리합니다:

1. **선택적 리빌드**: Riverpod의 `ref.watch(provider.select(...))`로 더 세밀한 구독이 가능
2. **autoDispose**: Riverpod은 자동 상태 정리로 메모리 관리가 용이
3. **의존성 그래프**: Riverpod은 의존성 그래프를 효율적으로 관리하여 불필요한 재계산 방지

Provider의 `context.select()`도 유사한 최적화를 제공하지만, Riverpod이 더 일관된 API를 제공합니다.

---

## 9. 마이그레이션 매핑 테이블

| Provider | Riverpod |
|----------|----------|
| `ChangeNotifierProvider` | `NotifierProvider` |
| `StateNotifierProvider` | `NotifierProvider` (레거시) |
| `FutureProvider` | `FutureProvider` |
| `StreamProvider` | `StreamProvider` |
| `Provider` | `Provider` |
| `ProxyProvider` | `ref.watch` (자동) |
| `Consumer` | `Consumer` / `ConsumerWidget` |
| `context.watch<T>()` | `ref.watch(provider)` |
| `context.read<T>()` | `ref.read(provider)` |
| `context.select<T, R>()` | `ref.watch(provider.select())` |

---

## 10. 참고 자료

- [Provider 공식 문서](https://pub.dev/packages/provider)
- [Riverpod 공식 문서](https://riverpod.dev/)
- [Riverpod vs Provider 비교 (공식)](https://riverpod.dev/docs/from_provider/provider_vs_riverpod)
- [Flutter 상태 관리 가이드](https://docs.flutter.dev/data-and-backend/state-mgmt)
