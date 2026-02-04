# ListView와 스크롤 위젯 비교 - 면접 답변 가이드

> **최종 업데이트**: 2026년 2월  
> **적용 버전**: Flutter 3.x, Dart 3.0+

## 핵심 질문

> **"ListView, ListView.builder, SingleChildScrollView + Column 3가지의 차이점은 무엇인가요?"**

---

## 1. 한눈에 보는 비교표

| 특성 | ListView | ListView.builder | SingleChildScrollView + Column |
|------|----------|------------------|-------------------------------|
| **렌더링 방식** | 모든 아이템 즉시 생성 | 화면에 보이는 아이템만 생성 (Lazy) | 모든 아이템 즉시 생성 |
| **메모리 효율** | 낮음 | 높음 | 낮음 |
| **적합한 아이템 수** | 소량 (10-20개 이하) | 대량 (수백~수천 개) | 소량 + 복잡한 레이아웃 |
| **스크롤 가능 여부** | 기본 제공 | 기본 제공 | 기본 제공 |
| **itemCount 필요** | ❌ | ✅ (권장) | ❌ |
| **무한 스크롤** | ❌ | ✅ | ❌ |

---

## 2. ListView (기본)

### 특징
- `children` 속성에 위젯 리스트를 직접 전달
- **모든 자식 위젯을 즉시 생성** (Eager Loading)
- 간단한 정적 리스트에 적합

### 코드 예시

```dart
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    ListTile(title: Text('Item 3')),
    // 모든 아이템이 즉시 빌드됨
  ],
)
```

### 언제 사용하나요?

- 아이템 수가 적고 고정된 경우 (10-20개 이하)
- 정적인 설정 화면, 메뉴 리스트
- 아이템 생성 비용이 낮을 때

### 주의사항

```dart
// ❌ 나쁜 예 - 많은 아이템에 ListView 사용
ListView(
  children: List.generate(1000, (index) => 
    ExpensiveWidget(index: index)  // 1000개 모두 즉시 생성!
  ),
)

// ✅ 좋은 예 - ListView.builder 사용
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) => ExpensiveWidget(index: index),
)
```

---

## 3. ListView.builder (권장)

### 특징
- **Lazy Loading**: 화면에 보이는 아이템만 생성
- 스크롤 시 화면 밖으로 나간 아이템은 메모리에서 해제
- 대량의 데이터 처리에 최적화
- **무한 스크롤 구현 가능**

### 코드 예시

```dart
ListView.builder(
  itemCount: items.length,  // null이면 무한 리스트
  itemBuilder: (BuildContext context, int index) {
    return ListTile(
      title: Text('Item $index'),
      subtitle: Text(items[index].description),
    );
  },
)
```

### 무한 스크롤 구현

```dart
ListView.builder(
  itemCount: null,  // 무한 리스트
  itemBuilder: (context, index) {
    // 끝에 도달하면 더 많은 데이터 로드
    if (index >= items.length - 5) {
      _loadMoreItems();
    }
    
    if (index >= items.length) {
      return const Center(child: CircularProgressIndicator());
    }
    
    return ListTile(title: Text(items[index]));
  },
)
```

### 내부 동작 원리

```
┌─────────────────────────────────────┐
│         화면 (Viewport)              │
│  ┌─────────────────────────────┐    │
│  │ Item 5  ← 빌드됨             │    │
│  │ Item 6  ← 빌드됨             │    │
│  │ Item 7  ← 빌드됨             │    │
│  │ Item 8  ← 빌드됨             │    │
│  │ Item 9  ← 빌드됨             │    │
│  └─────────────────────────────┘    │
│                                      │
│  Item 0~4: 해제됨 (화면 위)          │
│  Item 10+: 아직 생성 안 됨           │
└─────────────────────────────────────┘
```

### cacheExtent 설정

```dart
ListView.builder(
  // 화면 밖으로 몇 픽셀까지 미리 빌드할지 설정
  cacheExtent: 100.0,  // 기본값: 250.0
  itemCount: 1000,
  itemBuilder: (context, index) => MyWidget(index),
)
```

---

## 4. SingleChildScrollView + Column

### 특징
- Column 자체는 스크롤 불가 → SingleChildScrollView로 감싸서 스크롤 활성화
- **모든 자식 위젯을 즉시 생성**
- 다양한 위젯 타입을 자유롭게 배치 가능
- 복잡한 레이아웃에 유연함

### 코드 예시

```dart
SingleChildScrollView(
  child: Column(
    children: [
      // 다양한 위젯 타입 자유롭게 배치
      HeaderWidget(),
      const SizedBox(height: 16),
      ImageBanner(),
      const SizedBox(height: 16),
      ...items.map((item) => ItemCard(item: item)),
      FooterWidget(),
    ],
  ),
)
```

### 언제 사용하나요?

- **폼(Form) 화면**: 여러 입력 필드가 있는 화면
- **상세 페이지**: 헤더, 이미지, 본문, 관련 항목 등 다양한 구성
- **프로필 화면**: 사용자 정보, 통계, 설정 등 혼합 레이아웃
- 아이템 수가 적고 레이아웃이 복잡한 경우

### 실전 예시: 상품 상세 페이지

```dart
SingleChildScrollView(
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      // 상품 이미지 캐러셀
      ProductImageCarousel(images: product.images),
      
      // 상품 정보
      Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(product.name, style: Theme.of(context).textTheme.headlineMedium),
            const SizedBox(height: 8),
            Text('\$${product.price}', style: priceStyle),
            const SizedBox(height: 16),
            Text(product.description),
          ],
        ),
      ),
      
      // 리뷰 섹션 (아이템 수가 적다면)
      ReviewSection(reviews: product.reviews.take(5).toList()),
      
      // 관련 상품
      RelatedProducts(products: relatedProducts),
    ],
  ),
)
```

---

## 5. 기타 ListView 변형

### ListView.separated

아이템 사이에 구분선이나 여백을 추가할 때 사용

```dart
ListView.separated(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
  separatorBuilder: (context, index) {
    return const Divider(height: 1);
  },
)
```

### ListView.custom

SliverChildDelegate를 직접 제공하여 완전한 커스터마이징

```dart
ListView.custom(
  childrenDelegate: SliverChildBuilderDelegate(
    (context, index) => MyWidget(index),
    childCount: 100,
    // 아이템 재사용 로직 커스터마이징 가능
  ),
)
```

---

## 6. 성능 최적화 팁

### const 생성자 활용

```dart
ListView.builder(
  itemCount: 100,
  itemBuilder: (context, index) {
    return const ListTile(  // const 사용
      leading: Icon(Icons.star),
      title: Text('Static Item'),
    );
  },
)
```

### itemExtent 지정 (고정 높이일 때)

```dart
ListView.builder(
  itemCount: 1000,
  itemExtent: 56.0,  // 각 아이템의 높이가 고정되어 있을 때
  itemBuilder: (context, index) {
    return ListTile(title: Text('Item $index'));
  },
)
```

- `itemExtent`를 지정하면 Flutter가 스크롤 위치 계산을 최적화
- 모든 아이템을 측정할 필요 없이 정확한 스크롤 위치 계산 가능

### prototypeItem 사용 (Flutter 3.0+)

```dart
ListView.builder(
  itemCount: 1000,
  prototypeItem: const ListTile(title: Text('Prototype')),
  itemBuilder: (context, index) {
    return ListTile(title: Text('Item $index'));
  },
)
```

---

## 7. 면접 예상 질문

### Q1: "왜 ListView.builder가 ListView보다 성능이 좋은가요?"

**답변**: ListView.builder는 **Lazy Loading** 방식을 사용하여 화면에 보이는 아이템만 빌드합니다. 예를 들어 1000개의 아이템이 있어도 화면에 10개만 보인다면 10개만 메모리에 생성됩니다. 스크롤하면 화면 밖으로 나간 아이템은 dispose되고, 새로 보이는 아이템만 생성됩니다. 반면 ListView는 모든 children을 즉시 생성하므로 메모리 사용량이 높고 초기 빌드 시간이 오래 걸립니다.

### Q2: "무한 스크롤은 어떻게 구현하나요?"

**답변**: ListView.builder에서 `itemCount`를 null로 설정하거나, 현재 데이터 길이보다 1 큰 값으로 설정합니다. itemBuilder에서 마지막 인덱스에 도달하면 추가 데이터를 로드하는 함수를 호출하고, 로딩 인디케이터를 표시합니다. 또는 `ScrollController`를 사용하여 스크롤 위치가 끝에 가까워지면 데이터를 로드할 수 있습니다.

### Q3: "SingleChildScrollView와 ListView의 차이점은?"

**답변**: 
- **SingleChildScrollView**: 단일 자식 위젯을 스크롤 가능하게 만듭니다. 보통 Column과 함께 사용하며, 모든 자식을 즉시 렌더링합니다. 복잡하고 다양한 위젯 조합이 필요한 화면에 적합합니다.
- **ListView**: 여러 아이템을 효율적으로 스크롤하기 위해 설계되었습니다. 특히 ListView.builder는 대량의 동일한 형태 아이템에 최적화되어 있습니다.

### Q4: "ListView에서 스크롤 위치를 저장하고 복원하려면?"

**답변**:
```dart
// ScrollController 사용
final _scrollController = ScrollController();

// 위치 저장
final position = _scrollController.position.pixels;

// 위치 복원
_scrollController.jumpTo(savedPosition);
// 또는 애니메이션과 함께
_scrollController.animateTo(
  savedPosition,
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeOut,
);
```

또는 `PageStorageKey`를 사용하여 자동으로 스크롤 위치를 저장할 수 있습니다.

### Q5: "ListView 안에 ListView를 넣으면 어떤 문제가 발생하나요?"

**답변**: 중첩된 ListView는 스크롤 충돌 문제가 발생합니다. 내부 ListView가 스크롤되어야 할지, 외부 ListView가 스크롤되어야 할지 Flutter가 판단하기 어렵습니다.

**해결 방법:**
```dart
// 1. shrinkWrap + NeverScrollableScrollPhysics 사용
ListView(
  children: [
    ListView.builder(
      shrinkWrap: true,  // 내용물 크기에 맞춤
      physics: const NeverScrollableScrollPhysics(),  // 스크롤 비활성화
      itemCount: items.length,
      itemBuilder: (context, index) => ListTile(title: Text(items[index])),
    ),
  ],
)

// 2. CustomScrollView + Sliver 사용 (권장)
CustomScrollView(
  slivers: [
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text(items[index])),
        childCount: items.length,
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text(otherItems[index])),
        childCount: otherItems.length,
      ),
    ),
  ],
)
```

### Q6: "itemExtent와 prototypeItem의 차이는?"

**답변**:
- **itemExtent**: 픽셀 단위로 정확한 높이를 지정합니다. 모든 아이템이 동일한 고정 높이일 때 사용합니다.
- **prototypeItem**: 프로토타입 위젯을 제공하면 Flutter가 해당 위젯을 한 번 빌드하여 높이를 측정합니다. 아이템 높이가 동적이지만 모두 같은 구조일 때 유용합니다.

둘 다 스크롤 성능을 향상시키지만, prototypeItem은 더 유연하고 itemExtent는 더 명시적입니다.

---

## 8. 선택 가이드

```
아이템 수가 많은가? (100개 이상)
├── Yes → ListView.builder 사용
└── No
    ├── 아이템이 모두 같은 형태인가?
    │   ├── Yes → ListView 또는 ListView.builder
    │   └── No → SingleChildScrollView + Column
    └── 복잡한 레이아웃이 필요한가?
        ├── Yes → SingleChildScrollView + Column
        └── No → ListView

구분선이 필요한가?
├── Yes → ListView.separated
└── No → ListView.builder

여러 스크롤 영역을 조합해야 하는가?
├── Yes → CustomScrollView + Slivers
└── No → 위 옵션 중 선택
```

---

## 9. 참고 자료

- [Flutter 공식 문서 - ListView](https://api.flutter.dev/flutter/widgets/ListView-class.html)
- [Flutter 공식 문서 - SingleChildScrollView](https://api.flutter.dev/flutter/widgets/SingleChildScrollView-class.html)
- [Flutter 공식 문서 - Slivers](https://docs.flutter.dev/ui/layout/scrolling/slivers)
