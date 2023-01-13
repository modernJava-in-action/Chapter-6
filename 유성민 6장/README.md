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
  