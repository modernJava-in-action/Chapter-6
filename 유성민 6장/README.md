## 스트림으로 데이터 수집
  이번에는 스트림의 최종 연산중 하나인 *collect*에 대해서 알아보도록 하겠다. *collect*는 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있고 이 방법들은 *Collector* 인터페이스에 정의되어 있다.

  우선 컬렉터를 활용하는 예를 하나 살펴보도록 하자. 어떤 트랜잭션 리스트를 액면 통화로 그룹화한다고 가정하자. 스트림을 사용하지 않고 코드를 구현한다면 다음과 같이 *for loop*을 돌면서 리스트의 각 요소를 *Map*에 *key*로 나눠지도록 구현해야 한다:
```java
  private static void groupImperatively() {
    Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
    for (Transaction transaction : transactions) {
      Currency currency = transaction.getCurrency();
      List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
      if (transactionsForCurrency == null) {
        transactionsForCurrency = new ArrayList<>();
        transactionsByCurrencies.put(currency, transactionsForCurrency);
      }
      transactionsForCurrency.add(transaction);
    }

    System.out.println(transactionsByCurrencies);
  }
```
  그렇게 어려운 코드는 아니지만 아주 간단한 작업을 위해 너무 장황한 코드가 사용된다는 점은 부정할 수 없다. 

  스트림과 컬렉터를 적절히 사용하면 가독성과 간결성을 향상시킨 코드를 작성하는것이 가능하다:
```java
  private static void groupFunctionally() {
    Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream()
        .collect(groupingBy(Transaction::getCurrency));
    System.out.println(transactionsByCurrencies);
  }
```
  위 코드는 '통화별로 트랜잭션 리스트를 그룹화하시오'를 문장 그대로 옮겨놓은 것처럼 이해하기 쉽다.

### 컬렉터란 무엇인가?
  위 예제처럼 함수형 프로그래밍에서는 '무엇'을 원하는지 직접 명시할 수 있어서 어던 방법으로 이를 얻을지 신결 쓸 필요가 없다. *multilevel*로 그룹하를 수행할 때 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 두드러진다. 명령형 코드는 다중 루프를 사용해야해서 가독성이 크게 떨어지지만 함수형 프로그래밍에서는 필요한 컬렉터를 쉽게 추가할 수 있다.

#### 고급 리듀싱 기능을 수행하는 컬렉터
  함수형 API의 또 다른 장점으로 높은 수준의 조합성과 재사용성을 꼽을 수 있다. *collect*로 결과를 수집하는 과정을 간단하고 유연하게 정의할 수 있다는 것이 장점이다. 스트림에 *collect*를 호출하면 스트림의 요소에 내부적으로 **리듀싱 연산**이 일어난다. 명령형 프로그래밍에서 우리가 직접 구현해야 했던 작업이 자동으로 수행되는 것이다.

#### 미리 정의된 컬렉터
  위에서 살펴본 *groupingBy* 처럼 *Collectors* 클래스에는 미리 정의된 컬렉터들(팩토리 메서드)이 조재한다. *Collectors*에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다:
  - 스트림 요소를 하나의 값으로 리듀싱하고 요약
  - 요소 그룹화
  - 요소 분할

### 리듀싱과 요약
  아래 *Dish* 클래스를 통해서 다양한 컬렉터들을 살펴보자:
```java
public class Dish {

  private final String name;
  private final boolean vegetarian;
  private final int calories;
  private final Type type;

  public Dish(String name, boolean vegetarian, int calories, Type type) {
    this.name = name;
    this.vegetarian = vegetarian;
    this.calories = calories;
    this.type = type;
  }

  public String getName() {
    return name;
  }

  public boolean isVegetarian() {
    return vegetarian;
  }

  public int getCalories() {
    return calories;
  }

  public Type getType() {
    return type;
  }

  @Override
  public String toString() {
    return name;
  }

  public enum Type {
    MEAT,
    FISH,
    OTHER
  }

  public static final List<Dish> menu = asList(
      new Dish("pork", false, 800, Dish.Type.MEAT),
      new Dish("beef", false, 700, Dish.Type.MEAT),
      new Dish("chicken", false, 400, Dish.Type.MEAT),
      new Dish("french fries", true, 530, Dish.Type.OTHER),
      new Dish("rice", true, 350, Dish.Type.OTHER),
      new Dish("season fruit", true, 120, Dish.Type.OTHER),
      new Dish("pizza", true, 550, Dish.Type.OTHER),
      new Dish("prawns", false, 400, Dish.Type.FISH),
      new Dish("salmon", false, 450, Dish.Type.FISH)
  );

  public static final Map<String, List<String>> dishTags = new HashMap<>();
  static {
    dishTags.put("pork", asList("greasy", "salty"));
    dishTags.put("beef", asList("salty", "roasted"));
    dishTags.put("chicken", asList("fried", "crisp"));
    dishTags.put("french fries", asList("greasy", "fried"));
    dishTags.put("rice", asList("light", "natural"));
    dishTags.put("season fruit", asList("fresh", "natural"));
    dishTags.put("pizza", asList("tasty", "salty"));
    dishTags.put("prawns", asList("tasty", "roasted"));
    dishTags.put("salmon", asList("delicious", "fresh"));
  }
}
```
  
  첫 번째 예제로 *counting()*이라는 팩토리 메서드가 반환하는 컬렉터로 메뉴에서 요리 수를 계산해보자:
```java
  private static long howManyDishes() {
    return menu.stream().collect(counting());
  }
```

#### 스트림 값에서 최댓값과 최솟값 검색
  메뉴에서 칼로리가 가장 높은 요리를 찾으려면 *Collectors.maxBy*를 사용하면 된다. 반대로 최솟값을 구하고 싶다면 *Collectors.minBy*를 사용하면 된다. 두 컬렉터는 스트림의 요소를 비교하는 데 사용할 *Comparator*를 인수로 받는다. 다음은 칼로리로 요리를 비교하는 *Comparator*를 구현한 다음에 *Collectors.maxBy*로 전달하는 코드다.
```java
    Comparator<Dish> dishCaloriesComparator = Comparator.comparing(Dish::getCalories);
    
    Optional<Dish> mostCaloricDish = Dish.menu.stream()
      .collect(Collectors.maxBy(dishCaloriesComparator));
```

#### 요약 연산
  Collectors 클래스는 *Collectors.summingIng*라는 요약 팩토리 메서드를 제공한다. *summingIng*는 객체를 int로 매핑하는 함수를 인수로 받아서 객체를 int로 매핑한 컬렉터를 반환한다. 그리고 *summingInt*가 컬렉트 메서드로 전달되면 요약 작업을 수행한다. 아래는 메뉴 리스트의 총 칼로리를 계산하는 코드이다:
```java
  int totalCalories = menu.stream()
                          .collect(summingInt(Dish::getCalories));
```
  *summingLong, summingDouble, averagingInt, averagingLong, averagingDouble* 등의 다양한 요약 연산 기능을 제공한다. 그리고 이렇게 다양한 요약 연산을 하나에 결합시켜놓은 *summarizingInt*라는 컬렉터도 제공한다. 아래 코드를 실행후:
```java
  IntSummaryStatistics menuStatistics = menu.stream()
                                            .collect(summarizingInt(Dish::getCalories));
```
  *menuStatistics*를 출력해보면:
```
  IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}
```
  *count, sum, min, average, max* 연산을 한번에 수행할 수 있다.

#### 문자열 연결
  컬렉터에 *joining* 패토리 메서드를 이용하면 모든 문자열을 하나의 문자열로 연결해서 반환한다. 다음은 메뉴의 모든 요리명을 연결하는 코드다:
```java
  String shortMenu = menu.stream()
                         .map(Dish::getName)
                         .collect(joining());
```
  만약 Dish 클래스가 요리명을 반환하는 toString 메서드를 구현하고 있다면 아래와 같이 map을 생략해도된다:
```java
  String shortMenu = menu.stream()
                         .collect(joining());
```
  위 코드의 출력값은 모든 스트링이 연결되서 나오기 때문에 가독성이 떨어진다. 가독성 증가를 위해서 joining 메서드에 구분자를 추가해줄수 있다.
```java
  String shortMenu = menu.stream()
                         .map(Dish::getName)
                         .collect(joining(", "));
```

### 그룹화 (Grouping)
  스트림을 활용해서 데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산을 살펴보자. 이번에는 메뉴를 고기를 포함하는 그룹, 생선을 포함하는 그룹, 나머지 그룹으로 나눠보자. 이때는 *Collectors.groupingBy* 팩토리 메서드를 활용하면된다.
```java
  private static Map<Dish.Type, List<Dish>> groupDishesByType() {
    return menu.stream().collect(groupingBy(Dish::getType));
  }
```
  스트림의 각 요리에서 Dish.Type과 일치하는 요리를 추출하는 함수를 groupingBy 메서드로 전달했다. 이 함수를 기준으로 스트림이 그룹화되므로 이를 **분류 함수 (classification function)** 라고 부른다.
  
  단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 람다 표현식으로 로직을 구현할 수 있다. 아래는 칼로리로 메뉴를 그룹화하는 코드이다:
```java
  enum CaloricLevel { DIET, NORMAL, FAT };

  private static Map<CaloricLevel, List<Dish>> groupDishesByCaloricLevel() {
    return menu.stream().collect(
        groupingBy(dish -> {
          if (dish.getCalories() <= 400) {
            return CaloricLevel.DIET;
          }
          else if (dish.getCalories() <= 700) {
            return CaloricLevel.NORMAL;
          }
          else {
            return CaloricLevel.FAT;
          }
        })
    );
  }
```

#### 그룹화된 요소 조작
  요소를 그룹화 한 다음에 각 결과 그룹의 요소를 조작하는 방법을 살펴보자. 예를 들어 500 칼로리 이상의 요리를 필터링 하면서 모든 키를 유지하는 방법을 알아보자. 이렇게 하려면 groupingBy 팩토리 메서드를 오버로드해 두 번째 인수에 *filtering* 메서드를 넘겨주면 된다. 이 *filtering* 메소드는 팩토리 메서드로 프레디케이트를 인수로 받는다.
```java
  private static Map<Dish.Type, List<Dish>> groupCaloricDishesByType() {
    //return menu.stream().filter(dish -> dish.getCalories() > 500).collect(groupingBy(Dish::getType));
    return menu.stream().collect(
        groupingBy(Dish::getType,
            filtering(dish -> dish.getCalories() > 500, toList())));
  }
```

#### 다수준 그룹화
  *Collectors.groupingBy*를 이용하면 항목을 다수준으로 그룹화할 수 있다. 그러므로 groupingBy를 중첩해서 사용하면 외부, 내부 맵을 가진 다수준 그룹화가 가능하다. 아래 코드로 살펴보자:
```java
  private static Map<Dish.Type, Map<CaloricLevel, List<Dish>>> groupDishedByTypeAndCaloricLevel() {
    return menu.stream().collect(
        groupingBy(Dish::getType,
            groupingBy((Dish dish) -> {
              if (dish.getCalories() <= 400) {
                return CaloricLevel.DIET;
              }
              else if (dish.getCalories() <= 700) {
                return CaloricLevel.NORMAL;
              }
              else {
                return CaloricLevel.FAT;
              }
            })
        )
    );
  }
```

### 분할
  분할은 **분할 함수(partitioning function)**라 불리는 프레디케이트를 분류 함수로 사용한다. 분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 분할의 결과 맵은 참 아니면 거짓 두개의 그룹으로 분류된다.

  채식 요리와 채식이 아닌 요리로 나누는 코드로 살펴보자:
```java
  private static Map<Boolean, List<Dish>> partitionByVegeterian() {
    return menu.stream().collect(partitioningBy(Dish::isVegetarian));
  }
```
  위 코드의 리턴값은 아래와 같다:
```
  {false=[pork, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}
```
  이렇게 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 **분할의 장점**이다.

### Collector 인터페이스
  Collector 인터페이스에 있는 toList가 어떻게 구현되었는지 살펴보면서 내부적으로 컬렉트 메서드가 toList가 반환하는 함수를 어떻게 활용했는지 살펴보자. 다음 코드는 *Collector Interface*의 시그니처와 다섯개의 메서드 정의이다:
```java
  public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> chracteristics();
  }
```
  - T는 수집될 스트림 항목의 제네릭 형식이다.
  - A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식이다.
  - R은 수집 연산 결과 객체의 형식(대개 컬렉션)이다.
  
  Stream<T>의 모든 요소를 List<T>로 수집하는 ToListCollector<T>라는 클래스를 통해서 위 항목들을 자세히 알아보자.
```java
  public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
``
#### supplier 메서드: 새로운 결과 컨테이너 만들기
  supplier 메서드는 빈 결과로 이루어진 Supplier를 반환해야 한다. 즉, supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수다.
```java
  public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
  }
```

#### accumulator 메서드 : 결과 컨테이너에 요소 추가
  accumulator 메서드는 리듀싱 연산을 수행하는 함수를 반환한다. 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부 상태가 바뀌므로 함수의 리턴값은 void이다. ToListCollector에서 accumulator가 반환하는 함수는 이미 탐색한 항목을 포함하는 리스트에 현재 항목을 추가하는 연산을 수행한다:
```java
  public BiConsumer<List<T>, T> accumulator {
    return (list, item) -> list.add(item);
  }
```

#### finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용
  