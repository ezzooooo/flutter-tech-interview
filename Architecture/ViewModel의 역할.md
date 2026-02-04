# ViewModel의 역할 - 면접 답변 가이드

> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Flutter 3.x, Dart 3.0+

## 핵심 질문

> **"ViewModel의 역할은 무엇인가요?"**

---

## 1. ViewModel이란?

### 정의

ViewModel은 **MVVM(Model-View-ViewModel) 아키텍처 패턴**에서 View와 Model 사이의 중재자 역할을 하는 컴포넌트입니다. UI 로직과 상태를 관리하며, View가 필요로 하는 데이터를 Model에서 가져와 가공하여 제공합니다.

### MVVM 패턴 구조

```
┌─────────────────────────────────────────────────────────────┐
│                         MVVM 패턴                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────┐      ┌──────────────┐      ┌─────────┐       │
│   │  View   │ ←──→ │  ViewModel   │ ←──→ │  Model  │       │
│   │  (UI)   │      │ (UI Logic)   │      │ (Data)  │       │
│   └─────────┘      └──────────────┘      └─────────┘       │
│                                                             │
│   • 사용자 입력       • 상태 관리          • 비즈니스 로직    │
│   • UI 렌더링         • 데이터 변환        • 데이터 저장소    │
│   • 이벤트 처리       • UI 로직            • API 통신        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. ViewModel의 핵심 역할

### 2.1 UI 상태 관리 (State Management)

ViewModel은 화면에 표시될 **모든 상태를 관리**합니다.

```dart
// ViewModel의 상태 정의
class ProductListState {
  final List<Product> products;
  final bool isLoading;
  final String? errorMessage;
  final String searchQuery;
  final ProductSortType sortType;
  
  const ProductListState({
    this.products = const [],
    this.isLoading = false,
    this.errorMessage,
    this.searchQuery = '',
    this.sortType = ProductSortType.newest,
  });
  
  ProductListState copyWith({
    List<Product>? products,
    bool? isLoading,
    String? errorMessage,
    String? searchQuery,
    ProductSortType? sortType,
  }) {
    return ProductListState(
      products: products ?? this.products,
      isLoading: isLoading ?? this.isLoading,
      errorMessage: errorMessage,
      searchQuery: searchQuery ?? this.searchQuery,
      sortType: sortType ?? this.sortType,
    );
  }
}
```

### 2.2 View와 Model 분리 (Separation of Concerns)

**View는 UI만 담당하고, ViewModel이 로직을 담당**합니다.

```dart
// ❌ 잘못된 예: View에 로직이 포함됨
class ProductListScreen extends StatefulWidget {
  @override
  _ProductListScreenState createState() => _ProductListScreenState();
}

class _ProductListScreenState extends State<ProductListScreen> {
  List<Product> products = [];
  bool isLoading = false;
  
  @override
  void initState() {
    super.initState();
    _loadProducts();  // UI 코드에 데이터 로딩 로직
  }
  
  Future<void> _loadProducts() async {
    setState(() => isLoading = true);
    try {
      final response = await http.get(Uri.parse('api/products'));
      final data = jsonDecode(response.body);
      setState(() {
        products = data.map((e) => Product.fromJson(e)).toList();
      });
    } catch (e) {
      // 에러 처리
    } finally {
      setState(() => isLoading = false);
    }
  }
  
  @override
  Widget build(BuildContext context) { ... }
}

// ✅ 올바른 예: ViewModel이 로직 담당
class ProductListViewModel extends Notifier<ProductListState> {
  @override
  ProductListState build() => const ProductListState();
  
  Future<void> loadProducts() async {
    state = state.copyWith(isLoading: true);
    try {
      final products = await ref.read(productRepositoryProvider).getProducts();
      state = state.copyWith(products: products, isLoading: false);
    } catch (e) {
      state = state.copyWith(errorMessage: e.toString(), isLoading: false);
    }
  }
}

// View는 UI만 담당
class ProductListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(productListViewModelProvider);
    
    if (state.isLoading) return const LoadingWidget();
    if (state.errorMessage != null) return ErrorWidget(state.errorMessage!);
    return ProductGrid(products: state.products);
  }
}
```

### 2.3 데이터 변환 (Data Transformation)

Model의 데이터를 **View가 사용하기 쉬운 형태로 변환**합니다.

```dart
class UserProfileViewModel extends Notifier<UserProfileState> {
  @override
  UserProfileState build() => const UserProfileState();
  
  // Model 데이터를 View용 데이터로 변환
  String get formattedJoinDate {
    final user = state.user;
    if (user == null) return '';
    
    final date = user.createdAt;
    return '${date.year}년 ${date.month}월 ${date.day}일 가입';
  }
  
  String get displayName {
    final user = state.user;
    if (user == null) return '게스트';
    return user.nickname.isNotEmpty ? user.nickname : user.email.split('@').first;
  }
  
  // 금액 포맷팅
  String formatPrice(int price) {
    final formatter = NumberFormat('#,###', 'ko_KR');
    return '${formatter.format(price)}원';
  }
  
  // 상태에 따른 UI 문자열
  String get orderStatusText {
    return switch (state.order?.status) {
      OrderStatus.pending => '주문 대기 중',
      OrderStatus.confirmed => '주문 확인됨',
      OrderStatus.shipping => '배송 중',
      OrderStatus.delivered => '배송 완료',
      OrderStatus.cancelled => '주문 취소됨',
      null => '',
    };
  }
}
```

### 2.4 사용자 이벤트 처리 (Event Handling)

View에서 발생하는 **사용자 액션을 처리**합니다.

```dart
class LoginViewModel extends Notifier<LoginState> {
  @override
  LoginState build() => const LoginState();
  
  // 입력 이벤트 처리
  void onEmailChanged(String email) {
    state = state.copyWith(
      email: email,
      emailError: _validateEmail(email),
    );
  }
  
  void onPasswordChanged(String password) {
    state = state.copyWith(
      password: password,
      passwordError: _validatePassword(password),
    );
  }
  
  // 폼 유효성 검증
  String? _validateEmail(String email) {
    if (email.isEmpty) return '이메일을 입력해주세요';
    if (!email.contains('@')) return '올바른 이메일 형식이 아닙니다';
    return null;
  }
  
  String? _validatePassword(String password) {
    if (password.isEmpty) return '비밀번호를 입력해주세요';
    if (password.length < 8) return '비밀번호는 8자 이상이어야 합니다';
    return null;
  }
  
  bool get isFormValid =>
      state.emailError == null &&
      state.passwordError == null &&
      state.email.isNotEmpty &&
      state.password.isNotEmpty;
  
  // 로그인 버튼 클릭 처리
  Future<void> onLoginPressed() async {
    if (!isFormValid) return;
    
    state = state.copyWith(isLoading: true);
    try {
      await ref.read(authRepositoryProvider).login(
        email: state.email,
        password: state.password,
      );
      // 성공 처리
    } catch (e) {
      state = state.copyWith(error: e.toString());
    } finally {
      state = state.copyWith(isLoading: false);
    }
  }
}
```

### 2.5 비즈니스 로직 위임 (Business Logic Delegation)

복잡한 비즈니스 로직은 **Repository나 UseCase에 위임**합니다.

```dart
class CartViewModel extends Notifier<CartState> {
  @override
  CartState build() {
    _loadCart();
    return const CartState();
  }
  
  CartRepository get _cartRepository => ref.read(cartRepositoryProvider);
  
  Future<void> _loadCart() async {
    state = state.copyWith(isLoading: true);
    try {
      final items = await _cartRepository.getCartItems();
      state = state.copyWith(items: items, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }
  
  // ViewModel은 UI 관련 로직만 처리
  // 실제 장바구니 로직은 Repository에 위임
  Future<void> addToCart(Product product, int quantity) async {
    try {
      await _cartRepository.addItem(product.id, quantity);
      await _loadCart();  // 상태 갱신
    } catch (e) {
      state = state.copyWith(error: e.toString());
    }
  }
  
  // 계산 로직은 ViewModel에서 처리 (UI 관련)
  int get totalItems => state.items.fold(0, (sum, item) => sum + item.quantity);
  
  int get totalPrice => state.items.fold(
    0,
    (sum, item) => sum + (item.product.price * item.quantity),
  );
  
  bool get canCheckout => state.items.isNotEmpty && !state.isLoading;
}
```

---

## 3. Flutter에서 ViewModel 구현 방식

### 3.1 Riverpod + Notifier (권장)

```dart
// Provider 정의
final todoListViewModelProvider =
    NotifierProvider<TodoListViewModel, TodoListState>(
  TodoListViewModel.new,
);

// ViewModel 구현
class TodoListViewModel extends Notifier<TodoListState> {
  @override
  TodoListState build() {
    _loadTodos();
    return const TodoListState();
  }
  
  Future<void> _loadTodos() async {
    state = state.copyWith(isLoading: true);
    try {
      final todos = await ref.read(todoRepositoryProvider).getAll();
      state = state.copyWith(todos: todos, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }
  
  Future<void> addTodo(String title) async {
    final todo = Todo(id: uuid.v4(), title: title, completed: false);
    await ref.read(todoRepositoryProvider).add(todo);
    state = state.copyWith(todos: [...state.todos, todo]);
  }
  
  Future<void> toggleTodo(String id) async {
    final updatedTodos = state.todos.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(completed: !todo.completed);
      }
      return todo;
    }).toList();
    
    state = state.copyWith(todos: updatedTodos);
    await ref.read(todoRepositoryProvider).update(
      updatedTodos.firstWhere((t) => t.id == id),
    );
  }
  
  // Computed values
  List<Todo> get completedTodos =>
      state.todos.where((t) => t.completed).toList();
  
  List<Todo> get incompleteTodos =>
      state.todos.where((t) => !t.completed).toList();
  
  int get completionPercentage {
    if (state.todos.isEmpty) return 0;
    return (completedTodos.length / state.todos.length * 100).round();
  }
}

// View 사용
class TodoListScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(todoListViewModelProvider);
    final viewModel = ref.read(todoListViewModelProvider.notifier);
    
    return Scaffold(
      appBar: AppBar(
        title: Text('완료율: ${viewModel.completionPercentage}%'),
      ),
      body: state.isLoading
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: state.todos.length,
              itemBuilder: (context, index) {
                final todo = state.todos[index];
                return CheckboxListTile(
                  value: todo.completed,
                  title: Text(todo.title),
                  onChanged: (_) => viewModel.toggleTodo(todo.id),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context, viewModel),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### 3.2 Bloc (이벤트 기반)

```dart
// Event 정의
sealed class TodoEvent {}
final class LoadTodos extends TodoEvent {}
final class AddTodo extends TodoEvent {
  final String title;
  AddTodo(this.title);
}
final class ToggleTodo extends TodoEvent {
  final String id;
  ToggleTodo(this.id);
}

// State 정의
sealed class TodoState {}
final class TodoInitial extends TodoState {}
final class TodoLoading extends TodoState {}
final class TodoLoaded extends TodoState {
  final List<Todo> todos;
  TodoLoaded(this.todos);
}
final class TodoError extends TodoState {
  final String message;
  TodoError(this.message);
}

// Bloc (ViewModel 역할)
class TodoBloc extends Bloc<TodoEvent, TodoState> {
  final TodoRepository _repository;
  
  TodoBloc(this._repository) : super(TodoInitial()) {
    on<LoadTodos>(_onLoadTodos);
    on<AddTodo>(_onAddTodo);
    on<ToggleTodo>(_onToggleTodo);
  }
  
  Future<void> _onLoadTodos(LoadTodos event, Emitter<TodoState> emit) async {
    emit(TodoLoading());
    try {
      final todos = await _repository.getAll();
      emit(TodoLoaded(todos));
    } catch (e) {
      emit(TodoError(e.toString()));
    }
  }
  
  Future<void> _onAddTodo(AddTodo event, Emitter<TodoState> emit) async {
    if (state is! TodoLoaded) return;
    
    final currentTodos = (state as TodoLoaded).todos;
    final todo = Todo(id: uuid.v4(), title: event.title, completed: false);
    
    await _repository.add(todo);
    emit(TodoLoaded([...currentTodos, todo]));
  }
  
  Future<void> _onToggleTodo(ToggleTodo event, Emitter<TodoState> emit) async {
    if (state is! TodoLoaded) return;
    
    final currentTodos = (state as TodoLoaded).todos;
    final updatedTodos = currentTodos.map((todo) {
      if (todo.id == event.id) {
        return todo.copyWith(completed: !todo.completed);
      }
      return todo;
    }).toList();
    
    emit(TodoLoaded(updatedTodos));
  }
}
```

### 3.3 Provider + ChangeNotifier

```dart
class TodoViewModel extends ChangeNotifier {
  final TodoRepository _repository;
  
  TodoViewModel(this._repository) {
    loadTodos();
  }
  
  List<Todo> _todos = [];
  bool _isLoading = false;
  String? _error;
  
  List<Todo> get todos => _todos;
  bool get isLoading => _isLoading;
  String? get error => _error;
  
  Future<void> loadTodos() async {
    _isLoading = true;
    _error = null;
    notifyListeners();
    
    try {
      _todos = await _repository.getAll();
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
  
  Future<void> addTodo(String title) async {
    final todo = Todo(id: uuid.v4(), title: title, completed: false);
    await _repository.add(todo);
    _todos = [..._todos, todo];
    notifyListeners();
  }
}
```

---

## 4. ViewModel vs 다른 패턴

### 4.1 ViewModel vs Controller

| 특성 | ViewModel | Controller |
|------|-----------|------------|
| **패턴** | MVVM | MVC |
| **데이터 바인딩** | 양방향 (자동) | 단방향 (수동) |
| **View 참조** | 없음 (View 불인지) | 있음 (View 직접 제어) |
| **주요 역할** | UI 상태 & 로직 | View 업데이트 제어 |

### 4.2 ViewModel vs Presenter

| 특성 | ViewModel | Presenter |
|------|-----------|-----------|
| **패턴** | MVVM | MVP |
| **View와의 관계** | 느슨한 결합 | 1:1 관계 |
| **View 참조** | 없음 | 인터페이스로 참조 |
| **테스트 용이성** | 매우 좋음 | 좋음 |

---

## 5. ViewModel 베스트 프랙티스

### 5.1 단일 책임 원칙

```dart
// ❌ 여러 책임을 가진 ViewModel
class BadViewModel extends Notifier<AppState> {
  void loadUsers() { ... }
  void loadProducts() { ... }
  void loadOrders() { ... }
  void processPayment() { ... }  // 너무 많은 책임!
}

// ✅ 단일 책임을 가진 ViewModel
class UserListViewModel extends Notifier<UserListState> { ... }
class ProductListViewModel extends Notifier<ProductListState> { ... }
class OrderViewModel extends Notifier<OrderState> { ... }
class PaymentViewModel extends Notifier<PaymentState> { ... }
```

### 5.2 불변 상태 사용

```dart
// ❌ 가변 상태
class MutableState {
  List<Todo> todos = [];  // 직접 수정 가능!
  
  void addTodo(Todo todo) {
    todos.add(todo);  // 상태 직접 변경
  }
}

// ✅ 불변 상태
class ImmutableState {
  final List<Todo> todos;
  
  const ImmutableState({this.todos = const []});
  
  ImmutableState copyWith({List<Todo>? todos}) {
    return ImmutableState(todos: todos ?? this.todos);
  }
}

// 사용
state = state.copyWith(todos: [...state.todos, newTodo]);
```

### 5.3 비즈니스 로직 분리

```dart
// ❌ ViewModel에 비즈니스 로직 포함
class OrderViewModel extends Notifier<OrderState> {
  Future<void> placeOrder() async {
    // 재고 확인, 결제 처리, 주문 생성... (너무 복잡!)
    final items = state.cartItems;
    for (var item in items) {
      final stock = await api.checkStock(item.productId);
      if (stock < item.quantity) {
        throw Exception('재고 부족');
      }
    }
    final paymentResult = await paymentGateway.process(...);
    // ...
  }
}

// ✅ UseCase/Repository로 분리
class OrderViewModel extends Notifier<OrderState> {
  Future<void> placeOrder() async {
    state = state.copyWith(isLoading: true);
    try {
      // UseCase에 위임
      await ref.read(placeOrderUseCaseProvider).execute(
        cartItems: state.cartItems,
        paymentMethod: state.paymentMethod,
      );
      state = state.copyWith(isSuccess: true, isLoading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoading: false);
    }
  }
}
```

---

## 6. 면접 예상 질문

### Q1: "ViewModel의 주요 역할은 무엇인가요?"

**답변**: ViewModel의 주요 역할은 5가지입니다:
1. **UI 상태 관리**: 화면에 표시될 모든 상태를 관리합니다.
2. **View와 Model 분리**: UI 코드와 비즈니스 로직을 분리합니다.
3. **데이터 변환**: Model 데이터를 View가 사용하기 쉬운 형태로 변환합니다.
4. **이벤트 처리**: 사용자 액션(클릭, 입력 등)을 처리합니다.
5. **비즈니스 로직 위임**: 복잡한 로직은 Repository나 UseCase에 위임합니다.

### Q2: "ViewModel과 Controller의 차이점은?"

**답변**: 
- **ViewModel (MVVM)**: View를 알지 못합니다. 데이터 바인딩을 통해 상태가 자동으로 UI에 반영됩니다. View와 느슨하게 결합되어 있어 테스트가 용이합니다.
- **Controller (MVC)**: View에 대한 참조를 가지고 직접 View를 업데이트합니다. View와 더 밀접하게 결합되어 있습니다.

Flutter에서는 Riverpod의 Notifier나 Bloc이 ViewModel 역할을 하며, GetX의 Controller는 전통적인 Controller에 가깝습니다.

### Q3: "ViewModel에 비즈니스 로직을 넣어도 되나요?"

**답변**: ViewModel에는 **UI 관련 로직**만 넣는 것이 좋습니다. 예를 들어:
- ✅ 폼 유효성 검증
- ✅ 로딩/에러 상태 관리
- ✅ 데이터 포맷팅 (날짜, 금액 등)
- ✅ 필터링/정렬 UI 로직

복잡한 비즈니스 로직은 **Repository**나 **UseCase**에 위임해야 합니다:
- ❌ 결제 처리 로직
- ❌ 재고 확인 로직
- ❌ 복잡한 계산 로직

### Q4: "Flutter에서 ViewModel을 어떻게 구현하나요?"

**답변**: Flutter에서는 여러 방법으로 ViewModel을 구현할 수 있습니다:

1. **Riverpod Notifier** (권장): 가장 현대적인 방식으로, 컴파일 타임 안전성과 좋은 테스트 용이성을 제공합니다.
2. **Bloc**: 이벤트 기반으로 더 엄격한 단방향 데이터 흐름을 강제합니다.
3. **Provider + ChangeNotifier**: 가장 간단하지만 수동으로 notifyListeners를 호출해야 합니다.

```dart
// Riverpod 예시
class MyViewModel extends Notifier<MyState> {
  @override
  MyState build() => const MyState();
  
  void doSomething() {
    state = state.copyWith(...);  // 자동 UI 업데이트
  }
}
```

### Q5: "ViewModel 테스트는 어떻게 작성하나요?"

**답변**: ViewModel은 View와 분리되어 있어 단위 테스트가 쉽습니다:

```dart
void main() {
  test('TodoViewModel adds todo correctly', () {
    // Arrange
    final container = ProviderContainer(
      overrides: [
        todoRepositoryProvider.overrideWithValue(MockTodoRepository()),
      ],
    );
    
    final viewModel = container.read(todoViewModelProvider.notifier);
    
    // Act
    viewModel.addTodo('New Todo');
    
    // Assert
    final state = container.read(todoViewModelProvider);
    expect(state.todos.length, 1);
    expect(state.todos.first.title, 'New Todo');
    
    container.dispose();
  });
}
```

### Q6: "ViewModel의 상태는 왜 불변이어야 하나요?"

**답변**: 불변 상태를 사용하면:
1. **예측 가능성**: 상태가 언제 어떻게 변경되었는지 추적하기 쉽습니다.
2. **버그 방지**: 의도치 않은 상태 변경을 방지합니다.
3. **성능 최적화**: `==` 비교로 변경 여부를 빠르게 판단할 수 있습니다.
4. **시간 여행 디버깅**: 이전 상태로 돌아가거나 상태 히스토리를 관리할 수 있습니다.

```dart
// 불변 상태 변경
state = state.copyWith(count: state.count + 1);

// 가변 상태 변경 (비권장)
state.count++;  // 추적 어려움, 버그 발생 가능
```

---

## 7. 참고 자료

- [MVVM 패턴 소개](https://docs.microsoft.com/en-us/dotnet/architecture/maui/mvvm)
- [Flutter Architecture Blueprints](https://github.com/nicksrandall/flutter-architecture-blueprints)
- [Riverpod 공식 문서](https://riverpod.dev/)
- [flutter_bloc 아키텍처](https://bloclibrary.dev/#/architecture)
