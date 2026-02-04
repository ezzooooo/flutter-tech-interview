# Flutter 상태 (State) - 면접 답변 가이드

> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Flutter 3.x, Riverpod 3.0, flutter_bloc 8.x, Dart 3.0+

## 1. 상태(State)의 정의

### 기본 개념

- **상태란**: 앱이 실행되는 동안 변경될 수 있는 데이터
- UI를 빌드하는 데 사용되는 모든 데이터를 포함
- 위젯이 빌드되는 시점에 동기적으로 읽을 수 있어야 함
- 위젯의 생명주기 동안 변경될 수 있음

### 상태의 두 가지 유형

#### Ephemeral State (임시 상태 / UI 상태)

- 단일 위젯 내에서만 필요한 상태
- 다른 위젯에서 접근할 필요가 없는 상태
- 예시: PageView의 현재 페이지, 애니메이션 진행 상태, BottomNavigationBar의 선택된 탭

```dart
class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _selectedIndex = 0;  // Ephemeral State

  @override
  Widget build(BuildContext context) {
    return BottomNavigationBar(
      currentIndex: _selectedIndex,
      onTap: (index) {
        setState(() {
          _selectedIndex = index;
        });
      },
    );
  }
}
```

#### App State (앱 상태 / 공유 상태)

- 앱의 여러 부분에서 공유되는 상태
- 사용자 세션 간에 유지하고 싶은 상태
- 예시: 사용자 로그인 정보, 장바구니, 읽음/안읽음 상태, 알림

---

## 2. StatelessWidget vs StatefulWidget

### StatelessWidget

- 내부 상태를 가지지 않는 위젯
- 한 번 빌드되면 변경되지 않음 (부모가 변경하지 않는 한)
- 생성자에서 받은 데이터만 표시

```dart
class GreetingWidget extends StatelessWidget {
  final String name;

  const GreetingWidget({required this.name});

  @override
  Widget build(BuildContext context) {
    return Text('Hello, $name!');
  }
}
```

### StatefulWidget

- 변경 가능한 상태를 가지는 위젯
- State 객체와 함께 동작
- `setState()` 호출 시 UI 재빌드

```dart
class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

### 언제 어떤 것을 사용하나요?

| 상황                        | 권장 위젯       |
| --------------------------- | --------------- |
| 정적 UI, 변하지 않는 데이터 | StatelessWidget |
| 사용자 입력에 반응          | StatefulWidget  |
| 애니메이션                  | StatefulWidget  |
| 시간에 따라 변하는 데이터   | StatefulWidget  |

---

## 3. setState() 동작 원리

### 동작 과정

1. `setState()` 호출
2. 전달된 콜백 함수 실행 (상태 변경)
3. 위젯을 "dirty"로 표시
4. 다음 프레임에서 `build()` 메서드 재호출
5. 변경된 부분만 UI 업데이트 (Flutter의 효율적인 렌더링)

### 주의사항

```dart
// 잘못된 사용 - 비동기 작업 내에서
setState(() async {  // X - async 사용하지 말 것
  await fetchData();
});

// 올바른 사용
Future<void> _loadData() async {
  final data = await fetchData();
  setState(() {
    _data = data;  // 동기적으로 상태만 변경
  });
}
```

### setState() 성능 고려사항

- `setState()`는 해당 위젯의 전체 `build()` 메서드를 재실행
- 불필요하게 큰 위젯 트리에서 호출하면 성능 저하
- 상태를 가능한 작은 위젯으로 분리하는 것이 좋음

---

## 4. 상태 관리 솔루션 비교

### Provider

- Flutter 팀이 권장하는 상태 관리 솔루션
- InheritedWidget을 기반으로 구현
- 간단하고 직관적

```dart
// 상태 정의
class Counter with ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

// 상태 제공
ChangeNotifierProvider(
  create: (_) => Counter(),
  child: MyApp(),
)

// 상태 사용
Consumer<Counter>(
  builder: (context, counter, child) {
    return Text('${counter.count}');
  },
)

// context.watch - 변경 시 리빌드
Text('${context.watch<Counter>().count}')

// context.read - 한 번만 읽기 (이벤트 핸들러에서 사용)
onPressed: () => context.read<Counter>().increment()

// context.select - 특정 속성만 구독 (성능 최적화)
// name이 변경될 때만 리빌드, age 변경 시에는 리빌드하지 않음
final name = context.select<User, String>((user) => user.name);
```

### Riverpod

- Provider의 개선된 버전 (**Riverpod 3.0** - 2025년 9월 출시)
- 컴파일 타임 안전성 (ProviderNotFoundException 없음)
- 테스트 용이성
- Provider 간 의존성 관리 용이
- **주의**: `StateNotifierProvider`, `StateProvider`, `ChangeNotifierProvider`는 **레거시**로 분리됨

#### Riverpod 3.0 주요 변경 사항

| 변경 사항         | 설명                                                       |
| ----------------- | ---------------------------------------------------------- |
| 자동 재시도       | 실패한 Provider가 자동으로 exponential backoff로 재시도    |
| 화면 밖 일시정지  | 위젯이 보이지 않으면 Provider 자동 일시정지                |
| 레거시 분리       | `StateNotifierProvider` 등은 `legacy.dart`에서 import 필요 |
| `==` 필터링       | 모든 Provider가 `==`로 업데이트 필터링 (기존: `identical`) |
| Ref 단순화        | Ref 서브클래스 제거, 타입 파라미터 제거                    |
| Family 통합       | `FamilyNotifier` 제거 → `Notifier` + 생성자로 통합         |
| ProviderException | 에러가 `ProviderException`으로 래핑됨                      |

```dart
// ✅ Riverpod 3.0 권장 방식 - NotifierProvider
final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;  // 초기값 설정

  void increment() => state++;
  void decrement() => state--;
}

// 비동기 상태는 AsyncNotifierProvider 사용
final userProvider = AsyncNotifierProvider<UserNotifier, User>(
  UserNotifier.new,
);

class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return await fetchUser();
  }

  Future<void> updateName(String name) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() => updateUserName(name));
  }
}

// ✅ Riverpod 3.0 Family 사용법 (FamilyNotifier 제거됨)
final todoProvider = NotifierProvider.family<TodoNotifier, Todo, String>(
  TodoNotifier.new,
);

class TodoNotifier extends Notifier<Todo> {
  TodoNotifier(this.todoId);  // 생성자에서 arg 받음
  final String todoId;

  @override
  Todo build() {
    return Todo(id: todoId, title: '');
  }
}

// 상태 사용
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}

// 코드 생성 방식 (riverpod_generator 사용 시)
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  void increment() => state++;
}

// ⚠️ 레거시 사용 시 (마이그레이션 전까지만)
import 'package:flutter_riverpod/legacy.dart';
// StateNotifierProvider, StateProvider, ChangeNotifierProvider 사용 가능
```

#### Riverpod 3.0 자동 재시도 설정

```dart
// 전역 비활성화
ProviderScope(
  retry: (retryCount, error) => null,  // 재시도 안 함
  child: MyApp(),
)

// 특정 Provider만 비활성화
final myProvider = NotifierProvider<MyNotifier, int>(
  MyNotifier.new,
  retry: (retryCount, error) => null,
);

// 에러 처리 (3.0에서 ProviderException으로 래핑됨)
try {
  await ref.read(myProvider.future);
} on ProviderException catch (e) {
  if (e.exception is NotFoundException) {
    // 원본 에러 처리
  }
}
```

### Bloc (Business Logic Component)

- 이벤트 기반 상태 관리
- 명확한 상태 흐름
- 대규모 앱에 적합
- **Dart 3.0+**: `sealed class` 패턴으로 exhaustive 패턴 매칭 가능

```dart
// ✅ Dart 3.0+ sealed class 패턴 (권장)
// Event 정의
sealed class CounterEvent {}
final class Increment extends CounterEvent {}
final class Decrement extends CounterEvent {}
final class Reset extends CounterEvent {}

// State 정의 - sealed class로 모든 상태 표현
sealed class CounterState {
  final int count;
  const CounterState(this.count);
}

final class CounterInitial extends CounterState {
  const CounterInitial() : super(0);
}

final class CounterUpdated extends CounterState {
  const CounterUpdated(super.count);
}

// Bloc 정의
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterInitial()) {
    on<Increment>((event, emit) {
      emit(CounterUpdated(state.count + 1));
    });
    on<Decrement>((event, emit) {
      emit(CounterUpdated(state.count - 1));
    });
    on<Reset>((event, emit) {
      emit(const CounterInitial());
    });
  }
}

// 사용 - switch expression으로 exhaustive 패턴 매칭
BlocBuilder<CounterBloc, CounterState>(
  builder: (context, state) {
    return switch (state) {
      CounterInitial() => const Text('Press button to start'),
      CounterUpdated(:final count) => Text('Count: $count'),
    };
  },
)

// 비동기 상태 관리 예시
sealed class UserState {}
final class UserInitial extends UserState {}
final class UserLoading extends UserState {}
final class UserLoaded extends UserState {
  final User user;
  UserLoaded(this.user);
}
final class UserError extends UserState {
  final String message;
  UserError(this.message);
}
```

### GetX

- 상태 관리, 라우팅, 의존성 주입 통합
- 최소한의 보일러플레이트
- 반응형 상태 관리
- Context 없이 어디서든 접근 가능
- 변경된 위젯만 선택적으로 리빌드

```dart
// Controller 정의
class CounterController extends GetxController {
  var count = 0.obs;  // .obs로 반응형 변수 생성

  void increment() => count++;
  void decrement() => count--;

  // 생명주기 훅
  @override
  void onInit() {
    super.onInit();
    // 초기화 로직
  }

  @override
  void onClose() {
    // 정리 로직
    super.onClose();
  }
}

// 의존성 주입 (Bindings 활용)
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut(() => CounterController());
  }
}

// 사용 방법 1: Obx (권장 - 자동 리빌드)
Obx(() => Text('${controller.count}'))

// 사용 방법 2: GetX 위젯
GetX<CounterController>(
  init: CounterController(),
  builder: (controller) {
    return Text('${controller.count}');
  },
)

// 사용 방법 3: GetBuilder (수동 업데이트)
GetBuilder<CounterController>(
  builder: (controller) {
    return Text('${controller.count.value}');
  },
)

// 어디서든 컨트롤러 접근
final controller = Get.find<CounterController>();
```

### 상태 관리 솔루션 비교표

| 특성               | Provider        | Riverpod  | Bloc      | GetX      |
| ------------------ | --------------- | --------- | --------- | --------- |
| 학습 곡선          | 낮음            | 중간      | 높음      | 낮음      |
| 보일러플레이트     | 적음            | 적음      | 많음      | 매우 적음 |
| 테스트 용이성      | 좋음            | 매우 좋음 | 매우 좋음 | 보통      |
| 대규모 앱 적합성   | 보통            | 좋음      | 매우 좋음 | 보통      |
| 공식 지원          | Flutter 팀 권장 | 커뮤니티  | 커뮤니티  | 커뮤니티  |
| 컴파일 타임 안전성 | ❌              | ✅        | ✅        | ❌        |

### 각 솔루션의 장단점 정리

#### Provider

**장점:**

- Flutter 공식 권장, 낮은 학습 곡선
- InheritedWidget 기반으로 Flutter와 자연스러운 통합

**단점:**

- `ProviderNotFoundException` 런타임 에러 발생 가능
- Provider 간 복잡한 의존성 관리 어려움
- 동일 타입 여러 Provider 관리 불편

#### Riverpod

**장점:**

- 컴파일 타임 안전성 (런타임 에러 없음)
- BuildContext 없이 어디서든 접근 가능
- Provider 간 의존성 자동 관리
- 3.0: 자동 재시도, 화면 밖 일시정지로 안정성/성능 향상

**단점:**

- 코드 생성 의존 시 빌드 시간 증가
- Provider에 비해 높은 러닝커브
- 버전 업데이트 시 마이그레이션 필요 (2.0→3.0 breaking changes)
- 3.0: `StateNotifierProvider` 등 레거시 분리로 기존 코드 마이그레이션 필요

#### Bloc

**장점:**

- 명확한 단방향 데이터 흐름
- 뛰어난 테스트 용이성
- 대규모 팀/앱에 적합한 구조

**단점:**

- 많은 보일러플레이트 코드
- 간단한 기능에도 Event, State, Bloc 클래스 필요
- 초기 학습 곡선 높음

#### GetX

**장점:**

- 최소한의 코드로 빠른 개발
- 상태 관리 + 라우팅 + DI 통합
- Context 없이 전역 접근 가능

**단점:**

- 전역 상태로 인한 테스트 어려움
- "마법같은" 동작으로 디버깅 어려움
- 과도한 전역 접근은 코드 추적 어려움
- 커뮤니티 논란 (안티패턴 우려)

---

## 5. InheritedWidget

### 개념

- Flutter의 기본 상태 공유 메커니즘
- 위젯 트리 아래로 데이터를 효율적으로 전달
- Provider, Bloc 등 대부분의 상태 관리 솔루션의 기반

### 구현 예시

```dart
class CounterInherited extends InheritedWidget {
  final int count;
  final Function() increment;

  const CounterInherited({
    required this.count,
    required this.increment,
    required Widget child,
  }) : super(child: child);

  static CounterInherited of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<CounterInherited>()!;
  }

  @override
  bool updateShouldNotify(CounterInherited oldWidget) {
    return count != oldWidget.count;
  }
}
```

### updateShouldNotify의 역할

- 상속받은 위젯이 업데이트될 때 호출
- `true` 반환 시: 의존하는 위젯들 재빌드
- `false` 반환 시: 재빌드하지 않음
- 성능 최적화에 중요

---

## 6. 상태 끌어올리기 (Lifting State Up)

### 개념

- 여러 위젯에서 공유해야 하는 상태를 공통 부모로 이동
- React의 패턴과 유사
- 단순한 상태 공유에 효과적

### 예시

```dart
// 상태를 부모에서 관리
class ParentWidget extends StatefulWidget {
  @override
  _ParentWidgetState createState() => _ParentWidgetState();
}

class _ParentWidgetState extends State<ParentWidget> {
  bool _isActive = false;

  void _handleToggle(bool newValue) {
    setState(() {
      _isActive = newValue;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ToggleButton(
          isActive: _isActive,
          onToggle: _handleToggle,
        ),
        DisplayWidget(isActive: _isActive),
      ],
    );
  }
}

// 자식 위젯은 콜백을 통해 부모에 알림
class ToggleButton extends StatelessWidget {
  final bool isActive;
  final ValueChanged<bool> onToggle;

  const ToggleButton({
    required this.isActive,
    required this.onToggle,
  });

  @override
  Widget build(BuildContext context) {
    return Switch(
      value: isActive,
      onChanged: onToggle,
    );
  }
}
```

---

## 7. 면접에서 자주 나오는 질문들

### Q1: "setState()를 호출하면 정확히 어떤 일이 일어나나요?"

**답변**: setState()를 호출하면 먼저 전달된 콜백 함수가 동기적으로 실행되어 상태가 변경됩니다. 그 후 해당 State 객체가 "dirty"로 표시되고, Flutter 프레임워크가 다음 프레임에서 build() 메서드를 다시 호출합니다. Flutter는 이전 위젯 트리와 새 위젯 트리를 비교하여 실제로 변경된 부분만 효율적으로 업데이트합니다.

### Q2: "Provider와 Bloc의 차이점은 무엇인가요?"

**답변**: Provider는 간단하고 직관적인 상태 관리 솔루션으로, ChangeNotifier를 통해 상태 변경을 알립니다. 반면 Bloc은 이벤트 기반 아키텍처로, 모든 상태 변경이 Event를 통해 이루어지며 더 엄격한 단방향 데이터 흐름을 강제합니다. Provider는 작은~중간 규모 앱에, Bloc은 복잡한 비즈니스 로직이 있는 대규모 앱에 더 적합합니다.

### Q3: "언제 StatefulWidget 대신 다른 상태 관리 솔루션을 사용해야 하나요?"

**답변**:

- 단일 위젯 내에서만 사용되는 UI 상태: StatefulWidget으로 충분
- 여러 위젯에서 공유되는 상태: Provider/Riverpod 등 고려
- 복잡한 비즈니스 로직: Bloc 고려
- 상태가 앱 전체에서 접근 필요: 전역 상태 관리 솔루션 필요

### Q4: "Flutter에서 상태 관리의 best practice는 무엇인가요?"

**답변**:

1. 상태를 가능한 로컬하게 유지 (필요한 곳에서만 관리)
2. 불변성 유지 (상태 객체는 불변으로 설계)
3. 관심사 분리 (UI와 비즈니스 로직 분리)
4. 적절한 도구 선택 (앱 규모와 복잡도에 맞게)
5. 테스트 가능한 구조 유지

### Q5: "context.watch(), context.read(), context.select()의 차이는?"

**답변**:

- `context.watch()`: Provider의 값이 변경될 때마다 위젯을 재빌드합니다. build() 메서드 내에서 UI에 표시할 값에 사용합니다.
- `context.read()`: 값을 한 번만 읽고 변경을 구독하지 않습니다. 버튼 클릭 등의 이벤트 핸들러에서 사용합니다.
- `context.select()`: 객체의 특정 속성만 구독합니다. 예를 들어 User 객체에서 name만 select하면, age가 변경되어도 리빌드하지 않아 성능 최적화에 유용합니다.

### Q6: "Riverpod 3.0의 주요 변경 사항은?"

**답변**: Riverpod 3.0(2025년 9월)에서 다음과 같은 주요 변경이 있었습니다:

1. **레거시 분리**: `StateNotifierProvider`, `StateProvider`, `ChangeNotifierProvider`는 `legacy.dart`로 이동. 새 프로젝트에서는 `NotifierProvider` 사용 권장
2. **자동 재시도**: 실패한 Provider가 기본적으로 exponential backoff로 자동 재시도
3. **화면 밖 일시정지**: 위젯이 보이지 않으면 Provider가 자동으로 일시정지되어 리소스 절약
4. **Family 통합**: `FamilyNotifier` 제거, 일반 `Notifier`에서 생성자로 argument 전달
5. **Ref 단순화**: Ref 서브클래스 제거, 타입 파라미터 제거로 API 단순화
6. **ProviderException**: 에러가 `ProviderException`으로 래핑되어 더 나은 에러 핸들링 가능

### Q7: "Bloc에서 sealed class를 사용하는 이점은?"

**답변**: Dart 3.0의 sealed class를 사용하면:

1. **Exhaustive 패턴 매칭**: switch문에서 모든 상태를 처리하지 않으면 컴파일 에러 발생
2. **타입 안전성**: 정의된 상태만 사용 가능
3. **freezed 불필요**: 코드 생성 없이 비슷한 효과 달성
4. **가독성 향상**: 상태 계층 구조가 명확해짐

---

## 8. 심화 개념

### 상태 불변성 (Immutability)

```dart
// 나쁜 예 - 가변 상태
class UserState {
  String name;
  int age;
}

// 좋은 예 - 불변 상태
class UserState {
  final String name;
  final int age;

  const UserState({required this.name, required this.age});

  UserState copyWith({String? name, int? age}) {
    return UserState(
      name: name ?? this.name,
      age: age ?? this.age,
    );
  }
}
```

### ValueNotifier와 ValueListenableBuilder

```dart
// 간단한 단일 값 상태 관리
final counter = ValueNotifier<int>(0);

ValueListenableBuilder<int>(
  valueListenable: counter,
  builder: (context, value, child) {
    return Text('$value');
  },
)

// 값 변경
counter.value++;
```

### flutter_hooks

React의 Hooks 패턴을 Flutter에 도입한 라이브러리로, StatefulWidget의 보일러플레이트를 줄여줍니다.

```dart
// flutter_hooks 사용 예시
class CounterWidget extends HookWidget {
  @override
  Widget build(BuildContext context) {
    // useState - 상태 관리
    final counter = useState(0);

    // useMemoized - 비싼 연산 캐싱
    final expensiveValue = useMemoized(() => computeExpensiveValue());

    // useEffect - 사이드 이펙트 처리
    useEffect(() {
      print('Counter changed: ${counter.value}');
      return () => print('Cleanup');  // dispose 시 실행
    }, [counter.value]);

    // useTextEditingController - 컨트롤러 자동 관리
    final textController = useTextEditingController();

    // useAnimationController - 애니메이션 컨트롤러
    final animController = useAnimationController(
      duration: const Duration(seconds: 1),
    );

    return Column(
      children: [
        Text('Count: ${counter.value}'),
        ElevatedButton(
          onPressed: () => counter.value++,
          child: const Text('Increment'),
        ),
        TextField(controller: textController),
      ],
    );
  }
}

// useFuture와 useMemoized 조합 (비동기 데이터)
class UserWidget extends HookWidget {
  @override
  Widget build(BuildContext context) {
    // Future 재생성 방지
    final userFuture = useMemoized(() => fetchUser());
    final snapshot = useFuture(userFuture);

    if (snapshot.connectionState == ConnectionState.waiting) {
      return const CircularProgressIndicator();
    }
    return Text('Hello, ${snapshot.data?.name}');
  }
}
```

**flutter_hooks 주요 훅:**
| 훅 | 설명 |
|---|------|
| `useState` | 상태 변수 생성 및 관리 |
| `useMemoized` | 비싼 연산 결과 캐싱 |
| `useEffect` | 사이드 이펙트 및 cleanup 처리 |
| `useCallback` | 콜백 함수 메모이제이션 |
| `useRef` | 리빌드 없이 값 저장 |
| `useFuture` | Future 상태 구독 |
| `useStream` | Stream 상태 구독 |
| `useAnimationController` | 애니메이션 컨트롤러 자동 관리 |
| `useTextEditingController` | 텍스트 컨트롤러 자동 관리 |

### 비동기 상태 관리

```dart
// FutureBuilder
FutureBuilder<User>(
  future: fetchUser(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return CircularProgressIndicator();
    }
    if (snapshot.hasError) {
      return Text('Error: ${snapshot.error}');
    }
    return Text('Hello, ${snapshot.data!.name}');
  },
)

// StreamBuilder
StreamBuilder<int>(
  stream: counterStream,
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text('${snapshot.data}');
    }
    return CircularProgressIndicator();
  },
)
```

---

## 9. 상태 관리 선택 가이드

### 프로젝트 규모별 권장 솔루션

| 프로젝트 규모  | 권장 솔루션     | 이유                         |
| -------------- | --------------- | ---------------------------- |
| 소규모 (1-2명) | Provider / GetX | 빠른 개발, 낮은 학습 곡선    |
| 중규모 (3-5명) | Riverpod        | 타입 안전성, 테스트 용이성   |
| 대규모 (6명+)  | Bloc            | 명확한 아키텍처, 일관된 패턴 |

### 선택 기준 체크리스트

```
□ 팀원들의 학습 시간이 충분한가?
  - 충분함 → Bloc, Riverpod
  - 부족함 → Provider, GetX

□ 테스트 커버리지가 중요한가?
  - 매우 중요 → Bloc, Riverpod
  - 보통 → Provider
  - 낮음 → GetX

□ 앱의 복잡도는?
  - 복잡한 비즈니스 로직 → Bloc
  - 중간 복잡도 → Riverpod
  - 간단한 CRUD → Provider, GetX

□ 런타임 에러 방지가 중요한가?
  - 매우 중요 → Riverpod (컴파일 타임 안전성)
  - 보통 → 다른 솔루션도 가능
```

---

## 10. 참고 자료

- [Flutter 공식 문서 - State management](https://docs.flutter.dev/data-and-backend/state-mgmt)
- [Provider 패키지](https://pub.dev/packages/provider)
- [Riverpod 공식 문서](https://riverpod.dev/)
- [Riverpod 3.0 마이그레이션 가이드](https://riverpod.dev/docs/3.0_migration)
- [Riverpod What's New](https://riverpod.dev/docs/whats_new)
- [flutter_bloc 패키지](https://pub.dev/packages/flutter_bloc)
- [GetX 패키지](https://pub.dev/packages/get)
- [flutter_hooks 패키지](https://pub.dev/packages/flutter_hooks)
- [Dart 3.0 Sealed Classes](https://dart.dev/language/class-modifiers#sealed)
