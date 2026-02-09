# MVVM 패턴 - 면접 답변 가이드

> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Flutter 3.x, Dart 3.0+

## 핵심 질문

> **"MVVM 패턴이란 무엇인가요?"**  
> **"MVVM 패턴의 특징과 장점은 무엇인가요?"**  
> **"MVVM 패턴과 StatefulWidget의 차이점을 설명하세요."**

---

## 1. MVVM 패턴이란?

### 정의

MVVM(Model-View-ViewModel)은 UI 개발에서 **비즈니스 로직과 프레젠테이션 로직을 UI로부터 분리**하기 위한 아키텍처 패턴입니다. Microsoft의 Ken Cooper와 Ted Peters가 2005년에 발표했으며, WPF와 Silverlight 개발에서 시작되어 현재는 모바일, 웹 등 다양한 플랫폼에서 사용됩니다.

### 구성 요소

```
┌─────────────────────────────────────────────────────────────────┐
│                         MVVM 패턴                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────┐       ┌───────────────┐       ┌──────────┐       │
│   │   View   │ ←───→ │  ViewModel    │ ←───→ │  Model   │       │
│   │          │       │               │       │          │       │
│   └──────────┘       └───────────────┘       └──────────┘       │
│                                                                 │
│   • UI 렌더링          • UI 상태 관리          • 데이터 구조           │
│   • 사용자 입력 전달     • 데이터 변환            • 비즈니스 로직         │
│   • 상태 구독          • 이벤트 처리            • 데이터 소스 접근       │
│   • 화면 표시          • Model과 통신          • Repository         │
│                                                                 │
│   ──────── 데이터 바인딩 ────────   ──── 직접 참조 ────               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Model (모델)

- 앱의 **데이터 구조와 비즈니스 로직**을 담당
- 데이터 저장, 네트워크 통신, 데이터 가공 등 처리
- View나 ViewModel을 알지 못함 (독립적)

```dart
// 데이터 모델
class User {
  final String id;
  final String name;
  final String email;
  final DateTime createdAt;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
  });

  User copyWith({String? name, String? email}) {
    return User(
      id: id,
      name: name ?? this.name,
      email: email ?? this.email,
      createdAt: createdAt,
    );
  }

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      createdAt: DateTime.parse(json['createdAt'] as String),
    );
  }
}

// Repository (Model 레이어)
class UserRepository {
  final ApiClient _api;
  final LocalStorage _storage;

  UserRepository(this._api, this._storage);

  Future<User> getUser(String id) async {
    try {
      return await _api.fetchUser(id);
    } catch (e) {
      // 캐시된 데이터 반환
      return await _storage.getCachedUser(id);
    }
  }

  Future<void> updateUser(User user) async {
    await _api.updateUser(user);
    await _storage.cacheUser(user);
  }
}
```

#### View (뷰)

- **UI 렌더링**만 담당
- ViewModel의 상태를 구독하여 화면에 표시
- 사용자 입력을 ViewModel에 전달
- 로직을 포함하지 않음

```dart
// ✅ 올바른 View - UI만 담당
class UserProfileScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(userProfileViewModelProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('프로필')),
      body: switch (state) {
        UserProfileLoading() => const Center(
            child: CircularProgressIndicator(),
          ),
        UserProfileError(:final message) => Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text('오류: $message'),
                ElevatedButton(
                  onPressed: () => ref
                      .read(userProfileViewModelProvider.notifier)
                      .loadProfile(),
                  child: const Text('다시 시도'),
                ),
              ],
            ),
          ),
        UserProfileLoaded(:final user) => _buildProfile(context, ref, user),
      },
    );
  }

  Widget _buildProfile(BuildContext context, WidgetRef ref, User user) {
    return Column(
      children: [
        CircleAvatar(child: Text(user.name[0])),
        Text(user.name),
        Text(user.email),
        ElevatedButton(
          onPressed: () => ref
              .read(userProfileViewModelProvider.notifier)
              .logout(),
          child: const Text('로그아웃'),
        ),
      ],
    );
  }
}
```

#### ViewModel (뷰모델)

- View와 Model 사이의 **중재자**
- UI 상태를 관리하고 데이터를 변환
- View는 ViewModel을 알지만, ViewModel은 View를 알지 못함

```dart
// 상태 정의
sealed class UserProfileState {}
final class UserProfileLoading extends UserProfileState {}
final class UserProfileLoaded extends UserProfileState {
  final User user;
  UserProfileLoaded(this.user);
}
final class UserProfileError extends UserProfileState {
  final String message;
  UserProfileError(this.message);
}

// ViewModel
final userProfileViewModelProvider =
    NotifierProvider<UserProfileViewModel, UserProfileState>(
  UserProfileViewModel.new,
);

class UserProfileViewModel extends Notifier<UserProfileState> {
  @override
  UserProfileState build() {
    loadProfile();
    return UserProfileLoading();
  }

  Future<void> loadProfile() async {
    state = UserProfileLoading();
    try {
      final user = await ref.read(userRepositoryProvider).getUser('current');
      state = UserProfileLoaded(user);
    } catch (e) {
      state = UserProfileError(e.toString());
    }
  }

  Future<void> updateName(String name) async {
    final currentState = state;
    if (currentState is! UserProfileLoaded) return;

    try {
      final updated = currentState.user.copyWith(name: name);
      await ref.read(userRepositoryProvider).updateUser(updated);
      state = UserProfileLoaded(updated);
    } catch (e) {
      state = UserProfileError(e.toString());
    }
  }

  Future<void> logout() async {
    await ref.read(authRepositoryProvider).logout();
  }
}
```

---

## 2. MVVM 패턴의 특징

### 2.1 데이터 바인딩 (Data Binding)

View와 ViewModel 사이에 **데이터 바인딩**을 통해 상태 변경이 자동으로 UI에 반영됩니다.

```dart
// Flutter에서의 데이터 바인딩
// ViewModel의 state가 변경되면 → ref.watch가 감지 → View 자동 리빌드

class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ✅ 데이터 바인딩: state 변경 시 자동으로 이 위젯 리빌드
    final count = ref.watch(counterViewModelProvider);

    return Text('Count: $count');
  }
}
```

### 2.2 관심사의 분리 (Separation of Concerns)

각 레이어가 **하나의 역할**만 담당합니다.

```dart
// ❌ 관심사가 분리되지 않은 코드
class BadScreen extends StatefulWidget {
  @override
  _BadScreenState createState() => _BadScreenState();
}

class _BadScreenState extends State<BadScreen> {
  List<Product> products = [];
  bool isLoading = false;

  @override
  void initState() {
    super.initState();
    _fetchProducts();  // UI 코드에 네트워크 로직 혼재
  }

  Future<void> _fetchProducts() async {
    setState(() => isLoading = true);
    final response = await http.get(Uri.parse('https://api.example.com/products'));
    final data = jsonDecode(response.body) as List;
    setState(() {
      products = data.map((e) => Product.fromJson(e)).toList();
      isLoading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    // UI 렌더링 + 비즈니스 로직 + 데이터 처리가 한 파일에 혼재
    final discounted = products.where((p) => p.price > 10000).toList();
    // ...
  }
}

// ✅ MVVM으로 관심사 분리

// Model: 데이터 처리
class ProductRepository {
  Future<List<Product>> getProducts() async {
    final response = await _api.get('/products');
    return response.map((e) => Product.fromJson(e)).toList();
  }
}

// ViewModel: UI 로직
class ProductListViewModel extends Notifier<ProductListState> {
  @override
  ProductListState build() {
    _loadProducts();
    return const ProductListState();
  }

  Future<void> _loadProducts() async {
    state = state.copyWith(isLoading: true);
    try {
      final products = await ref.read(productRepositoryProvider).getProducts();
      state = state.copyWith(products: products, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }

  List<Product> get discountedProducts =>
      state.products.where((p) => p.price > 10000).toList();
}

// View: UI만 담당
class ProductListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(productListViewModelProvider);
    if (state.isLoading) return const CircularProgressIndicator();
    return ProductGrid(products: state.products);
  }
}
```

### 2.3 단방향 데이터 흐름

```
사용자 액션 → View → ViewModel → Model
                                    ↓
UI 업데이트 ← View ← ViewModel ← 결과
```

```dart
// 사용자가 "좋아요" 버튼 클릭
// 1. View: 이벤트 전달
ElevatedButton(
  onPressed: () => ref.read(postViewModelProvider.notifier).toggleLike(postId),
  child: const Icon(Icons.favorite),
)

// 2. ViewModel: 로직 처리 후 상태 업데이트
Future<void> toggleLike(String postId) async {
  final post = state.posts.firstWhere((p) => p.id == postId);
  final updated = post.copyWith(isLiked: !post.isLiked);

  // 낙관적 업데이트 (UI 먼저 변경)
  _updatePostInState(updated);

  try {
    await ref.read(postRepositoryProvider).toggleLike(postId);
  } catch (e) {
    // 실패 시 롤백
    _updatePostInState(post);
  }
}

// 3. View: 상태 변경 감지 → 자동 리빌드
```

### 2.4 ViewModel은 View를 모름

ViewModel은 **View에 대한 참조를 가지지 않습니다**. 상태만 변경하면 데이터 바인딩을 통해 View가 알아서 업데이트됩니다.

```dart
// ViewModel은 "어떤 화면에서 사용되는지" 알 필요 없음
class SearchViewModel extends Notifier<SearchState> {
  @override
  SearchState build() => const SearchState();

  void onQueryChanged(String query) {
    state = state.copyWith(query: query);
    _debounceSearch(query);
  }

  Future<void> _debounceSearch(String query) async {
    await Future.delayed(const Duration(milliseconds: 300));
    if (state.query != query) return; // 새 입력이 있으면 무시

    state = state.copyWith(isSearching: true);
    try {
      final results = await ref.read(searchRepositoryProvider).search(query);
      state = state.copyWith(results: results, isSearching: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isSearching: false);
    }
  }
}

// 같은 ViewModel을 다른 화면에서 재사용 가능
class SearchPage extends ConsumerWidget { /* ... */ }
class SearchDialog extends ConsumerWidget { /* ... */ }
class SearchBottomSheet extends ConsumerWidget { /* ... */ }
```

---

## 3. MVVM 패턴의 장점

### 3.1 테스트 용이성

ViewModel은 View에 독립적이므로 **UI 없이 단위 테스트** 가능합니다.

```dart
void main() {
  group('UserProfileViewModel', () {
    late ProviderContainer container;
    late MockUserRepository mockRepo;

    setUp(() {
      mockRepo = MockUserRepository();
      container = ProviderContainer(
        overrides: [
          userRepositoryProvider.overrideWithValue(mockRepo),
        ],
      );
    });

    tearDown(() => container.dispose());

    test('loadProfile 성공 시 UserProfileLoaded 상태', () async {
      // Arrange
      final testUser = User(
        id: '1',
        name: 'Test',
        email: 'test@test.com',
        createdAt: DateTime.now(),
      );
      when(mockRepo.getUser(any)).thenAnswer((_) async => testUser);

      // Act
      await container.read(userProfileViewModelProvider.notifier).loadProfile();

      // Assert
      final state = container.read(userProfileViewModelProvider);
      expect(state, isA<UserProfileLoaded>());
      expect((state as UserProfileLoaded).user.name, 'Test');
    });

    test('loadProfile 실패 시 UserProfileError 상태', () async {
      when(mockRepo.getUser(any)).thenThrow(Exception('Network error'));

      await container.read(userProfileViewModelProvider.notifier).loadProfile();

      final state = container.read(userProfileViewModelProvider);
      expect(state, isA<UserProfileError>());
    });
  });
}
```

### 3.2 코드 재사용성

```dart
// 하나의 ViewModel을 여러 화면에서 재사용
// 모바일 화면
class MobileCartScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(cartViewModelProvider);
    return MobileCartLayout(state: state);
  }
}

// 태블릿 화면
class TabletCartScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(cartViewModelProvider);  // 같은 ViewModel!
    return TabletCartLayout(state: state);
  }
}
```

### 3.3 유지보수성

- UI 변경 시 ViewModel 수정 불필요
- 비즈니스 로직 변경 시 View 수정 불필요
- 각 레이어를 독립적으로 리팩토링 가능

### 3.4 팀 협업 용이

- UI 개발자는 View만, 로직 개발자는 ViewModel/Model만 작업 가능
- 명확한 인터페이스로 병렬 작업 가능

---

## 4. MVVM vs 다른 아키텍처 패턴

### 4.1 비교표

| 특성                   | MVVM             | MVC               | MVP                    |
| ---------------------- | ---------------- | ----------------- | ---------------------- |
| **데이터 바인딩**      | ✅ 자동          | ❌ 수동           | ❌ 수동                |
| **View ↔ 로직 결합도** | 느슨함           | 밀접함            | 중간                   |
| **View의 로직 참조**   | ViewModel (간접) | Controller (직접) | Presenter (인터페이스) |
| **로직의 View 참조**   | ❌ 없음          | ✅ 있음           | ✅ 인터페이스로        |
| **테스트 용이성**      | 매우 좋음        | 보통              | 좋음                   |
| **복잡도**             | 중간             | 낮음              | 중간                   |
| **Flutter 적합성**     | 매우 좋음        | 보통              | 보통                   |

### 4.2 MVC (Model-View-Controller)

```
┌──────┐     ┌────────────┐     ┌───────┐
│ View │ ──→ │ Controller │ ──→ │ Model │
│      │ ←── │            │ ←── │       │
└──────┘     └────────────┘     └───────┘

Controller가 View를 직접 업데이트
```

### 4.3 MVP (Model-View-Presenter)

```
┌──────┐     ┌───────────┐     ┌───────┐
│ View │ ←─→ │ Presenter │ ──→ │ Model │
│(I/F) │     │           │ ←── │       │
└──────┘     └───────────┘     └───────┘

View와 Presenter가 인터페이스로 1:1 관계
```

### 4.4 MVVM

```
┌──────┐     ┌───────────┐     ┌───────┐
│ View │ ──→ │ ViewModel │ ──→ │ Model │
│      │ ←── │           │ ←── │       │
└──────┘     └───────────┘     └───────┘
     데이터 바인딩        직접 참조

ViewModel은 View를 모름 (데이터 바인딩으로 자동 연결)
```

---

## 5. MVVM 패턴과 StatefulWidget의 차이점

### 5.1 핵심 비교표

| 특성              | MVVM 패턴                   | StatefulWidget         |
| ----------------- | --------------------------- | ---------------------- |
| **상태 위치**     | ViewModel (외부)            | State 객체 (위젯 내부) |
| **로직 위치**     | ViewModel/Model로 분리      | State 클래스에 혼재    |
| **테스트**        | ViewModel 단독 테스트 가능  | 위젯 테스트 필요       |
| **상태 공유**     | 여러 위젯에서 쉽게 공유     | 부모→자식 전달 필요    |
| **코드 재사용**   | ViewModel 재사용 가능       | 위젯 단위로만 재사용   |
| **생명주기 관리** | ViewModel이 독립적으로 관리 | 위젯 생명주기에 종속   |
| **적합한 규모**   | 중~대규모 앱                | 소규모 또는 단순 UI    |

### 5.2 동일 기능 구현 비교

#### StatefulWidget 방식

```dart
// ❌ 모든 것이 하나의 위젯에 집중
class TodoScreen extends StatefulWidget {
  @override
  _TodoScreenState createState() => _TodoScreenState();
}

class _TodoScreenState extends State<TodoScreen> {
  List<Todo> _todos = [];
  bool _isLoading = false;
  String? _error;
  final _controller = TextEditingController();

  @override
  void initState() {
    super.initState();
    _loadTodos();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _loadTodos() async {
    setState(() => _isLoading = true);
    try {
      final response = await http.get(Uri.parse('https://api.example.com/todos'));
      final data = jsonDecode(response.body) as List;
      setState(() {
        _todos = data.map((e) => Todo.fromJson(e)).toList();
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  Future<void> _addTodo() async {
    if (_controller.text.isEmpty) return;
    final todo = Todo(
      id: DateTime.now().toString(),
      title: _controller.text,
      completed: false,
    );
    setState(() => _todos = [..._todos, todo]);
    _controller.clear();

    try {
      await http.post(
        Uri.parse('https://api.example.com/todos'),
        body: jsonEncode(todo.toJson()),
      );
    } catch (e) {
      setState(() => _todos = _todos.where((t) => t.id != todo.id).toList());
    }
  }

  Future<void> _toggleTodo(String id) async {
    setState(() {
      _todos = _todos.map((t) {
        if (t.id == id) return t.copyWith(completed: !t.completed);
        return t;
      }).toList();
    });
  }

  int get _completedCount => _todos.where((t) => t.completed).length;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('할 일 ($_completedCount/${_todos.length})')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _error != null
              ? Center(child: Text('오류: $_error'))
              : ListView.builder(
                  itemCount: _todos.length,
                  itemBuilder: (context, index) {
                    final todo = _todos[index];
                    return CheckboxListTile(
                      value: todo.completed,
                      title: Text(todo.title),
                      onChanged: (_) => _toggleTodo(todo.id),
                    );
                  },
                ),
      bottomSheet: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Expanded(child: TextField(controller: _controller)),
            IconButton(onPressed: _addTodo, icon: const Icon(Icons.add)),
          ],
        ),
      ),
    );
  }
}
```

#### MVVM 방식

```dart
// ✅ 관심사가 명확히 분리됨

// --- Model ---
class TodoRepository {
  final ApiClient _api;
  TodoRepository(this._api);

  Future<List<Todo>> getTodos() => _api.fetchTodos();
  Future<void> addTodo(Todo todo) => _api.createTodo(todo);
  Future<void> updateTodo(Todo todo) => _api.updateTodo(todo);
}

// --- ViewModel ---
class TodoState {
  final List<Todo> todos;
  final bool isLoading;
  final String? error;

  const TodoState({
    this.todos = const [],
    this.isLoading = false,
    this.error,
  });

  TodoState copyWith({
    List<Todo>? todos,
    bool? isLoading,
    String? error,
  }) {
    return TodoState(
      todos: todos ?? this.todos,
      isLoading: isLoading ?? this.isLoading,
      error: error,
    );
  }
}

final todoViewModelProvider = NotifierProvider<TodoViewModel, TodoState>(
  TodoViewModel.new,
);

class TodoViewModel extends Notifier<TodoState> {
  @override
  TodoState build() {
    _loadTodos();
    return const TodoState();
  }

  TodoRepository get _repo => ref.read(todoRepositoryProvider);

  Future<void> _loadTodos() async {
    state = state.copyWith(isLoading: true, error: null);
    try {
      final todos = await _repo.getTodos();
      state = state.copyWith(todos: todos, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }

  Future<void> addTodo(String title) async {
    final todo = Todo(
      id: DateTime.now().toString(),
      title: title,
      completed: false,
    );
    state = state.copyWith(todos: [...state.todos, todo]);
    try {
      await _repo.addTodo(todo);
    } catch (e) {
      state = state.copyWith(
        todos: state.todos.where((t) => t.id != todo.id).toList(),
      );
    }
  }

  void toggleTodo(String id) {
    state = state.copyWith(
      todos: state.todos.map((t) {
        if (t.id == id) return t.copyWith(completed: !t.completed);
        return t;
      }).toList(),
    );
  }

  int get completedCount => state.todos.where((t) => t.completed).length;
  int get totalCount => state.todos.length;
}

// --- View ---
class TodoScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(todoViewModelProvider);
    final viewModel = ref.read(todoViewModelProvider.notifier);

    return Scaffold(
      appBar: AppBar(
        title: Text('할 일 (${viewModel.completedCount}/${viewModel.totalCount})'),
      ),
      body: state.isLoading
          ? const Center(child: CircularProgressIndicator())
          : state.error != null
              ? Center(child: Text('오류: ${state.error}'))
              : TodoList(
                  todos: state.todos,
                  onToggle: viewModel.toggleTodo,
                ),
      bottomSheet: TodoInput(onSubmit: viewModel.addTodo),
    );
  }
}
```

### 5.3 핵심 차이 정리

```
StatefulWidget:
┌─────────────────────────────┐
│ TodoScreen (StatefulWidget) │
│                             │
│  • UI 렌더링                │
│  • 상태 변수 (_todos 등)    │   ← 모든 것이 한 곳에
│  • 네트워크 호출            │
│  • 데이터 가공              │
│  • 이벤트 처리              │
└─────────────────────────────┘

MVVM:
┌────────────┐  ┌───────────────┐  ┌───────────────┐
│ TodoScreen │  │ TodoViewModel │  │ TodoRepository│
│            │  │               │  │               │
│ • UI만     │→ │ • 상태 관리   │→ │ • API 통신    │
│ • 표시만   │← │ • 이벤트 처리 │← │ • 데이터 캐싱 │
│            │  │ • 데이터 변환 │  │               │
└────────────┘  └───────────────┘  └───────────────┘
```

---

## 6. Flutter에서 MVVM 구현 방법

### 6.1 Riverpod + Notifier (권장)

```dart
// ViewModel
final myViewModelProvider = NotifierProvider<MyViewModel, MyState>(
  MyViewModel.new,
);

class MyViewModel extends Notifier<MyState> {
  @override
  MyState build() => const MyState();
  // ...
}

// View
class MyScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(myViewModelProvider);
    // ...
  }
}
```

### 6.2 Bloc (이벤트 기반 MVVM)

```dart
// ViewModel 역할을 Bloc이 담당
class MyBloc extends Bloc<MyEvent, MyState> {
  MyBloc() : super(MyInitial()) {
    on<LoadData>(_onLoadData);
  }
}

// View
BlocBuilder<MyBloc, MyState>(
  builder: (context, state) {
    // ...
  },
)
```

### 6.3 Provider + ChangeNotifier

```dart
// ViewModel
class MyViewModel extends ChangeNotifier {
  MyState _state = const MyState();
  MyState get state => _state;

  void updateState(MyState newState) {
    _state = newState;
    notifyListeners();
  }
}

// View
Consumer<MyViewModel>(
  builder: (context, viewModel, child) {
    // ...
  },
)
```

---

## 7. 면접 예상 질문

### Q1: "MVVM 패턴이란 무엇인가요?"

**답변**: MVVM은 Model-View-ViewModel의 약자로, UI 개발에서 비즈니스 로직과 프레젠테이션 로직을 UI로부터 분리하기 위한 아키텍처 패턴입니다. Model은 데이터와 비즈니스 로직을, View는 UI 렌더링을, ViewModel은 View와 Model 사이에서 상태 관리와 데이터 변환을 담당합니다. 핵심은 데이터 바인딩을 통해 ViewModel의 상태 변경이 자동으로 View에 반영된다는 것이며, ViewModel은 View를 알지 못하는 느슨한 결합 구조입니다.

#### 꼬리질문: "MVVM에서 데이터 바인딩이란 무엇인가요?"

**답변**: 데이터 바인딩은 ViewModel의 상태와 View의 UI를 자동으로 연결하는 메커니즘입니다. ViewModel에서 상태가 변경되면 View가 이를 자동으로 감지하여 UI를 업데이트합니다. Flutter에서는 Riverpod의 `ref.watch()`나 Bloc의 `BlocBuilder`가 이 역할을 합니다. 개발자가 명시적으로 "UI를 업데이트해라"라고 호출하지 않아도, 상태 변경만으로 UI가 반응적으로 갱신됩니다.

#### 꼬리질문: "MVVM에서 ViewModel은 View를 왜 알면 안 되나요?"

**답변**: ViewModel이 View를 알면 두 가지 문제가 생깁니다. 첫째, **결합도가 높아져** 특정 View에 종속되어 재사용이 불가능합니다. 둘째, **테스트가 어려워집니다**. ViewModel이 View를 모르면, View 없이 순수 로직 테스트가 가능하고, 동일한 ViewModel을 모바일/태블릿/웹 등 다양한 View에서 재사용할 수 있습니다.

### Q2: "MVVM 패턴의 특징과 장점은 무엇인가요?"

**답변**: MVVM의 주요 특징은 네 가지입니다:

1. **데이터 바인딩**: 상태 변경이 자동으로 UI에 반영
2. **관심사의 분리**: View는 UI, ViewModel은 로직, Model은 데이터만 담당
3. **단방향 데이터 흐름**: 사용자 입력 → ViewModel 처리 → 상태 변경 → View 업데이트
4. **느슨한 결합**: ViewModel이 View를 알지 못함

장점으로는 **테스트 용이성** (ViewModel 단위 테스트 가능), **코드 재사용성** (같은 ViewModel을 여러 View에서 사용), **유지보수성** (각 레이어 독립 수정), **팀 협업 용이** (UI/로직 병렬 개발) 등이 있습니다.

#### 꼬리질문: "MVVM 패턴의 단점은 무엇인가요?"

**답변**: MVVM의 단점도 있습니다:

1. **과도한 보일러플레이트**: 간단한 화면에도 State, ViewModel, Repository 등 여러 클래스를 만들어야 합니다
2. **학습 곡선**: 패턴 이해와 데이터 바인딩 개념에 익숙해지는 데 시간이 필요합니다
3. **과설계 위험**: 소규모 앱에서는 불필요한 복잡성을 추가할 수 있습니다
4. **디버깅 어려움**: 데이터 흐름이 여러 레이어를 거치므로 문제 추적이 복잡할 수 있습니다

#### 꼬리질문: "Flutter에서 MVVM이 특히 잘 맞는 이유는?"

**답변**: Flutter의 **선언적 UI** 방식이 MVVM의 데이터 바인딩과 자연스럽게 맞기 때문입니다. Flutter는 `build()` 메서드에서 현재 상태에 따라 UI를 선언하고, 상태가 변경되면 자동으로 리빌드합니다. 이것이 바로 MVVM의 데이터 바인딩 개념과 일치합니다. Riverpod의 `ref.watch()`나 Bloc의 `BlocBuilder`가 이 바인딩을 자연스럽게 구현해줍니다.

### Q3: "MVVM 패턴과 StatefulWidget의 차이점을 설명하세요."

**답변**: 가장 큰 차이는 **상태와 로직의 위치**입니다.

StatefulWidget은 상태와 UI 로직, 비즈니스 로직이 모두 하나의 State 클래스에 모여 있습니다. 간단한 UI에는 편리하지만, 복잡해지면 유지보수가 어려워집니다.

MVVM은 View(UI), ViewModel(상태/로직), Model(데이터)로 명확히 분리됩니다. ViewModel은 독립적으로 테스트 가능하고, 여러 View에서 재사용할 수 있습니다. 상태 공유도 쉽습니다.

실무에서는 단순한 UI 상태(탭 선택, 애니메이션 등)는 StatefulWidget을, 비즈니스 로직이 포함된 복잡한 상태는 MVVM 패턴을 사용합니다.

#### 꼬리질문: "그러면 StatefulWidget은 언제 사용하나요?"

**답변**: StatefulWidget은 다음 경우에 적합합니다:

1. **Ephemeral State(임시 상태)**: 현재 탭 인덱스, 텍스트 필드 포커스, 애니메이션 진행률 등 해당 위젯에서만 필요한 상태
2. **컨트롤러 관리**: `TextEditingController`, `AnimationController`, `ScrollController` 등의 생명주기 관리
3. **initState/dispose 필요 시**: 리소스 초기화/정리가 필요한 경우

핵심은 **"이 상태가 다른 위젯에서도 필요한가?"**를 기준으로 판단하는 것입니다. 해당 위젯에서만 필요하면 StatefulWidget, 여러 위젯에서 공유되면 MVVM을 사용합니다.

#### 꼬리질문: "StatefulWidget에서 MVVM으로 리팩토링해야 하는 시점은?"

**답변**: 다음 신호가 보이면 리팩토링을 고려해야 합니다:

1. **State 클래스가 200줄 이상** 되어 관리가 어려울 때
2. **여러 화면에서 같은 로직을 복사-붙여넣기** 할 때
3. **API 호출이나 복잡한 비즈니스 로직**이 위젯에 직접 포함될 때
4. **테스트를 작성하기 어려울 때** (Widget 테스트 대신 단위 테스트가 필요할 때)
5. **여러 위젯이 같은 상태를 공유**해야 할 때

#### 꼬리질문: "MVVM에서 Ephemeral State는 어디서 관리하나요?"

**답변**: Ephemeral State(임시 상태)는 여전히 **StatefulWidget에서 관리**합니다. MVVM에서도 모든 상태를 ViewModel에 넣을 필요는 없습니다.

```dart
// ViewModel: 비즈니스 상태 관리
class ProductViewModel extends Notifier<ProductState> { /* ... */ }

// View: Ephemeral State는 StatefulWidget에서 관리
class ProductScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<ProductScreen> createState() => _ProductScreenState();
}

class _ProductScreenState extends ConsumerState<ProductScreen> {
  final _scrollController = ScrollController();  // Ephemeral State
  bool _isExpanded = false;                       // Ephemeral State

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final state = ref.watch(productViewModelProvider);  // 비즈니스 상태
    // ...
  }
}
```

---

## 8. 참고 자료

- [MVVM 패턴 소개 (Microsoft)](https://docs.microsoft.com/en-us/dotnet/architecture/maui/mvvm)
- [Flutter Architecture Samples](https://fluttersamples.com/)
- [Riverpod 공식 문서](https://riverpod.dev/)
- [flutter_bloc 아키텍처](https://bloclibrary.dev/#/architecture)
- [Clean Architecture with Flutter](https://resocoder.com/flutter-clean-architecture-tdd/)
