# Riverpod 상태관리 패턴 - 면접 답변 가이드

> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Riverpod 3.0, Flutter 3.x, Dart 3.0+

## 핵심 질문

> **"Riverpod 상태관리 패턴의 특징은 무엇인가요?"**

---

## 1. Riverpod이란?

### 정의

Riverpod은 Flutter를 위한 **반응형 상태 관리 및 의존성 주입 프레임워크**입니다. Provider 패키지의 저자(Remi Rousselet)가 Provider의 한계를 극복하기 위해 완전히 새롭게 설계한 솔루션입니다.

> **이름의 유래**: "Riverpod"는 "Provider"의 애너그램(anagram)입니다.

### Provider vs Riverpod 핵심 차이

```dart
// Provider - BuildContext 필요
final value = context.watch<MyNotifier>().value;

// Riverpod - BuildContext 불필요
final value = ref.watch(myProvider);
```

---

## 2. Riverpod의 핵심 특징

### 2.1 컴파일 타임 안전성 (Compile-time Safety)

**Provider의 문제점:**
```dart
// Provider - 런타임 에러 발생 가능
// ProviderNotFoundException: Could not find the correct Provider<Counter>
final counter = context.watch<Counter>();  // Counter가 등록 안 되어 있으면 런타임 에러!
```

**Riverpod의 해결:**
```dart
// Riverpod - 컴파일 타임에 에러 검출
final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

// Provider가 존재하지 않으면 컴파일 에러!
final count = ref.watch(counterProvider);  // ✅ 안전
final count = ref.watch(nonExistentProvider);  // ❌ 컴파일 에러
```

### 2.2 BuildContext 독립성

```dart
// ✅ 어디서든 Provider 접근 가능
class ApiService {
  final Ref ref;
  ApiService(this.ref);
  
  Future<User> getUser() async {
    // 서비스 레이어에서도 다른 Provider 접근 가능
    final token = ref.read(authTokenProvider);
    return await api.fetchUser(token);
  }
}

// Provider로 서비스 정의
final apiServiceProvider = Provider((ref) => ApiService(ref));
```

### 2.3 Provider 간 의존성 자동 관리

```dart
// 인증 토큰 Provider
final authTokenProvider = StateProvider<String?>((ref) => null);

// 사용자 정보 Provider - authToken에 의존
final userProvider = FutureProvider<User>((ref) async {
  final token = ref.watch(authTokenProvider);  // 의존성 자동 추적
  
  if (token == null) {
    throw Exception('Not authenticated');
  }
  
  return await fetchUser(token);
});

// authTokenProvider가 변경되면 userProvider 자동 재계산!
```

### 2.4 동일 타입 여러 Provider 지원

```dart
// Provider에서는 동일 타입 구분 어려움
// Riverpod에서는 쉽게 구분
final lightThemeProvider = Provider<ThemeData>((ref) => lightTheme);
final darkThemeProvider = Provider<ThemeData>((ref) => darkTheme);

// 각각 독립적으로 접근
final light = ref.watch(lightThemeProvider);
final dark = ref.watch(darkThemeProvider);
```

### 2.5 테스트 용이성

```dart
// 쉬운 Provider 오버라이드
void main() {
  testWidgets('counter test', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          // 테스트용 Mock Provider
          userRepositoryProvider.overrideWithValue(MockUserRepository()),
          counterProvider.overrideWith(() => MockCounterNotifier()),
        ],
        child: const MyApp(),
      ),
    );
  });
}
```

### 2.6 자동 dispose (Riverpod 3.0)

```dart
// autoDispose: 구독자가 없으면 자동으로 상태 정리
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  // 화면을 벗어나면 자동으로 dispose
  return await fetchUser();
});

// 캐싱이 필요한 경우
final cachedUserProvider = FutureProvider<User>((ref) async {
  // autoDispose 없음 = 캐싱됨
  return await fetchUser();
});
```

---

## 3. Riverpod 3.0 주요 변경사항

### 3.1 레거시 Provider 분리

```dart
// ❌ Riverpod 3.0에서 deprecated
// import 'package:flutter_riverpod/flutter_riverpod.dart';
// final counterProvider = StateProvider<int>((ref) => 0);  // 레거시!

// ✅ 레거시 사용 시 (마이그레이션 전)
import 'package:flutter_riverpod/legacy.dart';
final counterProvider = StateProvider<int>((ref) => 0);

// ✅ 권장: NotifierProvider 사용
final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);
```

### 3.2 자동 재시도 (Auto-retry)

```dart
// 실패한 Provider가 자동으로 exponential backoff로 재시도
final dataProvider = FutureProvider<Data>((ref) async {
  // 네트워크 에러 발생 시 자동 재시도
  return await fetchData();
});

// 재시도 비활성화
final dataProvider = FutureProvider<Data>(
  (ref) async => await fetchData(),
  retry: (retryCount, error) => null,  // 재시도 안 함
);

// 전역 비활성화
ProviderScope(
  retry: (retryCount, error) => null,
  child: MyApp(),
)
```

### 3.3 화면 밖 일시정지

위젯이 화면에 보이지 않으면 Provider가 자동으로 일시정지되어 리소스 절약

### 3.4 Family 통합

```dart
// ❌ Riverpod 2.x - FamilyNotifier 사용
// ✅ Riverpod 3.0 - 일반 Notifier + 생성자

final todoProvider = NotifierProvider.family<TodoNotifier, Todo, String>(
  TodoNotifier.new,
);

class TodoNotifier extends Notifier<Todo> {
  TodoNotifier(this.todoId);  // 생성자에서 argument 받음
  final String todoId;

  @override
  Todo build() {
    return Todo(id: todoId, title: '');
  }
  
  void updateTitle(String title) {
    state = state.copyWith(title: title);
  }
}

// 사용
final todo = ref.watch(todoProvider('todo-123'));
```

### 3.5 Ref 단순화

```dart
// Riverpod 2.x - 여러 Ref 타입 존재
// - Ref, WidgetRef, AutoDisposeRef 등

// Riverpod 3.0 - 단일 Ref 타입
// 타입 파라미터 제거로 API 단순화
```

---

## 4. Provider 종류별 사용법

### 4.1 Provider (읽기 전용)

```dart
// 단순 값 제공
final greetingProvider = Provider<String>((ref) => 'Hello, World!');

// 다른 Provider에 의존
final fullGreetingProvider = Provider<String>((ref) {
  final name = ref.watch(userNameProvider);
  return 'Hello, $name!';
});
```

### 4.2 NotifierProvider (동기 상태)

```dart
final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;  // 초기값
  
  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}

// 사용
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### 4.3 AsyncNotifierProvider (비동기 상태)

```dart
final userProvider = AsyncNotifierProvider<UserNotifier, User>(
  UserNotifier.new,
);

class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    // 초기 데이터 로드
    return await fetchUser();
  }
  
  Future<void> updateName(String name) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final updated = await updateUserName(name);
      return updated;
    });
  }
  
  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => fetchUser());
  }
}

// 사용
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);
    
    return userAsync.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
      data: (user) => Text('Hello, ${user.name}'),
    );
  }
}
```

### 4.4 FutureProvider (단순 비동기)

```dart
// 상태 변경이 필요 없는 단순 비동기 데이터
final configProvider = FutureProvider<AppConfig>((ref) async {
  return await loadAppConfig();
});

// autoDispose로 메모리 관리
final weatherProvider = FutureProvider.autoDispose<Weather>((ref) async {
  return await fetchWeather();
});
```

### 4.5 StreamProvider

```dart
final messagesProvider = StreamProvider<List<Message>>((ref) {
  final userId = ref.watch(userIdProvider);
  return chatRepository.messagesStream(userId);
});

// 사용
Consumer(
  builder: (context, ref, child) {
    final messagesAsync = ref.watch(messagesProvider);
    
    return messagesAsync.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
      data: (messages) => MessageList(messages: messages),
    );
  },
)
```

---

## 5. 핵심 API

### 5.1 ref.watch vs ref.read vs ref.listen

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ watch: 값이 변경되면 위젯 리빌드 (build 메서드에서 사용)
    final count = ref.watch(counterProvider);
    
    // ✅ listen: 값 변경 시 사이드 이펙트 실행 (build 메서드에서 사용)
    ref.listen(counterProvider, (previous, next) {
      if (next > 10) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('Count exceeded 10!')),
        );
      }
    });
    
    return ElevatedButton(
      // ✅ read: 이벤트 핸들러에서 한 번만 읽기
      onPressed: () => ref.read(counterProvider.notifier).increment(),
      child: Text('Count: $count'),
    );
  }
}
```

### 5.2 select로 성능 최적화

```dart
// ❌ 전체 User 객체 변경 시 리빌드
final user = ref.watch(userProvider);

// ✅ name만 변경될 때만 리빌드
final name = ref.watch(userProvider.select((user) => user.name));

// 복합 조건
final isAdult = ref.watch(
  userProvider.select((user) => user.age >= 18),
);
```

### 5.3 invalidate & refresh

```dart
// invalidate: Provider 상태 초기화 (다음 읽기 시 재계산)
ref.invalidate(userProvider);

// refresh: 즉시 재계산하고 새 값 반환
final newUser = ref.refresh(userProvider);
```

---

## 6. 코드 생성 방식 (riverpod_generator)

### 설정

```yaml
# pubspec.yaml
dependencies:
  flutter_riverpod: ^3.0.0
  riverpod_annotation: ^3.0.0

dev_dependencies:
  riverpod_generator: ^3.0.0
  build_runner: ^2.4.0
```

### 사용법

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'providers.g.dart';

// 동기 Provider
@riverpod
String greeting(Ref ref) => 'Hello, World!';

// Notifier
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
}

// 비동기 Notifier
@riverpod
class User extends _$User {
  @override
  Future<UserModel> build() async {
    return await fetchUser();
  }
}

// Family (파라미터 있는 Provider)
@riverpod
Future<Todo> todo(Ref ref, String todoId) async {
  return await fetchTodo(todoId);
}
```

```bash
# 코드 생성
dart run build_runner build
# 또는 watch 모드
dart run build_runner watch
```

---

## 7. 실전 패턴

### 7.1 Repository 패턴

```dart
// Repository 추상화
abstract class UserRepository {
  Future<User> getUser(String id);
  Future<void> updateUser(User user);
}

// 구현체
class UserRepositoryImpl implements UserRepository {
  final ApiClient api;
  UserRepositoryImpl(this.api);
  
  @override
  Future<User> getUser(String id) => api.fetchUser(id);
  
  @override
  Future<void> updateUser(User user) => api.updateUser(user);
}

// Provider 정의
final userRepositoryProvider = Provider<UserRepository>((ref) {
  final api = ref.watch(apiClientProvider);
  return UserRepositoryImpl(api);
});

// 사용
final userProvider = FutureProvider.family<User, String>((ref, userId) async {
  final repository = ref.watch(userRepositoryProvider);
  return await repository.getUser(userId);
});
```

### 7.2 인증 상태 관리

```dart
// 인증 상태
sealed class AuthState {}
final class AuthInitial extends AuthState {}
final class AuthLoading extends AuthState {}
final class Authenticated extends AuthState {
  final User user;
  Authenticated(this.user);
}
final class Unauthenticated extends AuthState {}

// 인증 Notifier
final authProvider = NotifierProvider<AuthNotifier, AuthState>(
  AuthNotifier.new,
);

class AuthNotifier extends Notifier<AuthState> {
  @override
  AuthState build() => AuthInitial();
  
  Future<void> login(String email, String password) async {
    state = AuthLoading();
    try {
      final user = await ref.read(authRepositoryProvider).login(email, password);
      state = Authenticated(user);
    } catch (e) {
      state = Unauthenticated();
      rethrow;
    }
  }
  
  Future<void> logout() async {
    await ref.read(authRepositoryProvider).logout();
    state = Unauthenticated();
  }
}
```

### 7.3 페이지네이션

```dart
final postsProvider = AsyncNotifierProvider<PostsNotifier, List<Post>>(
  PostsNotifier.new,
);

class PostsNotifier extends AsyncNotifier<List<Post>> {
  int _page = 1;
  bool _hasMore = true;
  
  @override
  Future<List<Post>> build() async {
    _page = 1;
    _hasMore = true;
    return await _fetchPosts(1);
  }
  
  Future<void> loadMore() async {
    if (!_hasMore || state is AsyncLoading) return;
    
    _page++;
    final newPosts = await _fetchPosts(_page);
    
    if (newPosts.isEmpty) {
      _hasMore = false;
    } else {
      state = AsyncData([...state.value ?? [], ...newPosts]);
    }
  }
  
  Future<List<Post>> _fetchPosts(int page) async {
    return await ref.read(postRepositoryProvider).getPosts(page: page);
  }
}
```

---

## 8. 면접 예상 질문

### Q1: "Riverpod의 가장 큰 특징은?"

**답변**: Riverpod의 가장 큰 특징은 **컴파일 타임 안전성**입니다. Provider와 달리 존재하지 않는 Provider를 참조하면 런타임이 아닌 컴파일 타임에 에러가 발생합니다. 또한 BuildContext에 의존하지 않아 서비스 레이어나 테스트에서도 쉽게 사용할 수 있고, Provider 간 의존성을 자동으로 추적하여 관리합니다.

### Q2: "ref.watch와 ref.read의 차이점은?"

**답변**: 
- **ref.watch**: Provider의 값을 구독합니다. 값이 변경되면 위젯이 자동으로 리빌드됩니다. build 메서드 내에서 UI에 표시할 값에 사용합니다.
- **ref.read**: 값을 한 번만 읽고 변경을 구독하지 않습니다. 버튼 클릭 같은 이벤트 핸들러에서 사용합니다.

```dart
// build에서는 watch
final count = ref.watch(counterProvider);

// 이벤트 핸들러에서는 read
onPressed: () => ref.read(counterProvider.notifier).increment()
```

### Q3: "NotifierProvider와 StateNotifierProvider의 차이는?"

**답변**: Riverpod 3.0에서 `StateNotifierProvider`는 레거시로 분류되었습니다. 새로운 `NotifierProvider`가 권장됩니다.

**주요 차이점:**
- **NotifierProvider**: Ref를 통해 다른 Provider에 접근, build 메서드에서 초기값 설정
- **StateNotifierProvider**: 생성자에서 초기값 설정, Ref 접근이 제한적

```dart
// ✅ NotifierProvider (권장)
class CounterNotifier extends Notifier<int> {
  @override
  int build() {
    // ref로 다른 Provider 접근 가능
    final initial = ref.watch(initialValueProvider);
    return initial;
  }
}

// ❌ StateNotifierProvider (레거시)
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);  // 생성자에서 초기값
}
```

### Q4: "Provider 간 의존성은 어떻게 관리되나요?"

**답변**: Riverpod은 ref.watch를 통해 **의존성을 자동으로 추적**합니다.

```dart
final authTokenProvider = StateProvider<String?>((ref) => null);

final userProvider = FutureProvider<User>((ref) async {
  final token = ref.watch(authTokenProvider);  // 의존성 자동 추적
  if (token == null) throw Exception('Not authenticated');
  return await fetchUser(token);
});
```

`authTokenProvider`가 변경되면 `userProvider`가 자동으로 재계산됩니다. 이 의존성 그래프는 Riverpod이 내부적으로 관리하며, 순환 의존성이 있으면 경고합니다.

### Q5: "autoDispose는 언제 사용하나요?"

**답변**: `autoDispose`는 **구독자가 없어지면 Provider 상태를 자동으로 정리**합니다.

**사용하는 경우:**
- 화면별 일회성 데이터 (상세 페이지 등)
- 메모리 정리가 중요한 대용량 데이터
- 사용자가 떠나면 의미 없는 실시간 데이터

**사용하지 않는 경우:**
- 캐싱이 필요한 데이터
- 앱 전체에서 공유되는 상태

```dart
// 상세 페이지 - 화면 벗어나면 정리
final productDetailProvider = FutureProvider.autoDispose
    .family<Product, String>((ref, productId) async {
  return await fetchProduct(productId);
});

// 사용자 정보 - 캐싱 (autoDispose 없음)
final currentUserProvider = FutureProvider<User>((ref) async {
  return await fetchCurrentUser();
});
```

### Q6: "Riverpod에서 테스트는 어떻게 작성하나요?"

**답변**: Riverpod은 **ProviderScope의 overrides**를 통해 쉽게 Provider를 Mock으로 교체할 수 있습니다.

```dart
void main() {
  testWidgets('displays user name', (tester) async {
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          // Mock Repository 주입
          userRepositoryProvider.overrideWithValue(MockUserRepository()),
          // Mock Notifier 주입
          authProvider.overrideWith(() => MockAuthNotifier()),
        ],
        child: const MyApp(),
      ),
    );
    
    await tester.pumpAndSettle();
    expect(find.text('John Doe'), findsOneWidget);
  });
}

// 유닛 테스트
test('counter increments', () {
  final container = ProviderContainer(
    overrides: [
      initialValueProvider.overrideWithValue(10),
    ],
  );
  
  expect(container.read(counterProvider), 10);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 11);
  
  container.dispose();
});
```

---

## 9. 마이그레이션 가이드 (2.x → 3.0)

### 주요 변경사항

| 2.x | 3.0 |
|-----|-----|
| `StateProvider` | `NotifierProvider` |
| `StateNotifierProvider` | `NotifierProvider` |
| `ChangeNotifierProvider` | `NotifierProvider` |
| `FamilyNotifier` | `Notifier` + 생성자 |
| 여러 Ref 타입 | 단일 `Ref` 타입 |

### 마이그레이션 예시

```dart
// Before (2.x)
final counterProvider = StateProvider<int>((ref) => 0);

// After (3.0)
final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;
  
  void increment() => state++;
}
```

---

## 10. 참고 자료

- [Riverpod 공식 문서](https://riverpod.dev/)
- [Riverpod 3.0 마이그레이션 가이드](https://riverpod.dev/docs/migration/from_riverpod_2_to_3)
- [riverpod_generator 패키지](https://pub.dev/packages/riverpod_generator)
- [Flutter Riverpod 패키지](https://pub.dev/packages/flutter_riverpod)
