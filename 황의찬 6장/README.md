# Chapter 6 -  스트림으로 데이터 수집
## 이전 내용의 복습
Java8의 스트림은 데이터 집합을 멋지게 처리하는 게으른 반복자라고 설명할 수 있습니다.  
중간 연산(filter, map 같은)은 스트림의 요소를 소비하지 않습니다. 반면, 최종 연산은 스트림의 요소를 소비해서  
최종 결과를 도출합니다(예를 들어 가장 큰 값 반환).
  
최종 연산은 스트림 파이프라인을 최적화하면서 계산 과정을 짧게 생략하기도 합니다.  
  
이 장에서는 reduce가 그랬던 것처럼 collect 역시 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는  
리듀싱 연산을 수행할 수 있음을 설명합니다. 다양한 요소 누적 방식은 Collector 인터페이스에 정의되어 있습니다.  
  
## collect VS Collector VS Collectors
- collect : 스트림의 최종 연산 메서드 중 하나입니다.  
- Collector : collect에서 필요한 메서드를 정의해놓은 인터페이스입니다.  
- Collectors : 복수형이 잘 나타내듯이 `Collector`를 구현한 클래스들을 제공합니다.  
  
다양한 요소 누적 방식은 Collector 인터페이스에 정의되어 있습니다.  
  
Collectors.toList()는 Collector를 반환하기 때문에 `collect(Collectors.toList())`를 하면 `collect` 메서드에 `collector` 인터페이스가 들어가는 것입니다.  
쉽게 설명하면, stream.collect(요소 누적 방식); 과 같고 요소 누적 방식은 Collector 인터페이스에 정의되어 있습니다.  
Collector 인터페이스들을 구현한 클래스가 Collectors입니다.  
## 6.1 컬렉터란 무엇인가?
함수형 프로그래밍에서는 '무엇'을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없습니다.  
이전 예제에서 collect 메서드로 Collector 인터페이스 구현을 전달했습니다.  
Collector 인터페이스 구현은 `스트림의 요소를 어떤 식으로 도출할지 지정합니다`.  
  
이전까지는 toList를 Collector 인터페이스의 구현으로 사용했습니다.  
함수형 프로그래밍에서는 필요한 컬렉터를 쉽게 추가할 수 있습니다.  

### 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터
훌륭하게 설계된 함수형 API의 또 다른 장점으로는 `높은 수준의 조합성과 재사용성`을 꼽을 수 있습니다.  
collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점입니다.  
  
구체적으로 설명해서 스트림에 `collect`를 호출하면 스트림의 요소에(컬렉터로 파라미터화된) 리듀싱 연산이 수행됩니다.  
collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리합니다.  
  
Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정됩니다.  
Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공합니다.  
  
```java
List<Transaction> transactions = transactionStream.collect(Collectors.toList());
```
### 6.1.2 미리 정의된 컬렉터
Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있습니다.  
- 스트림 요소를 `하나의 값으로 리듀스하고 요약`    
- 요소 `그룹화`    
- 요소 `분할`  

추가적으로, 커스텀 컬렉터를 만들어서 Collectors 클래스의 팩토리 메서드로 해결할 수 없는 작업을 해결하는 방법을 설명합니다.  

## 6.2 리듀싱과 요약 
Collector 로 스트림의 항목을 컬렉션으로 재구성할 수 있습니다.  
좀 더 일반적으로 말해 컬렉터로 스트림의 모든 항목을 `하나의 결과로 합칠 수 있습니다.`  
```java
long howManyDishes = menu.stream()
			.collect(Collectors.counting());

// 불필요한 과정 생략 가능
long howManyDishesByCount = menu.stream().count();
```
counting 컬렉터는 다른 컬렉터와 함께 사용할 때 위력을 발휘합니다.  
  
### 6.2.1 스트림에서 최대값과 최솟값 검색
`Collectors.maxBy, Collectors.minBy` 두 개의 메서드를 이용해서 스트림의 최댓값과 최솟값을 계산할 수 있습니다.  
두 컬렉터는 스트림의 요소를 비교하는 데 사용할 Comparator를 인수로 받습니다.  
```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```
menu가 비어있다면 그 어떤 요리도 반환되지 않을 수 있기 때문에 Optional로 감싸서 반환합니다.  
  
또한 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용됩니다. 이러한 연산을 **요약(Summarization)**연산이라고 부릅니다.  
  
### 6.2.2 요약 연산
Collectors 클래스는 Collectors.summingInt 라는 특별한 요약 팩토리 메서드를 제공합니다.  
summingInt는 객체를 int로 매핑하는 함수를 인수로 받습니다. summingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환합니다.  
```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
칼로리로 매핑된 각 요리의 값을 탐색하면서 초깃값(여기서는 0)으로 설정되어 있는 누적자에 칼로리를 더합니다.  
  
이러한 단순 합계 외에 평균값 계산 등의 연산도 요약 기능으로 제공됩니다.  
Collectors.averagingInt, averagingDouble 등으로 다양한 형식으로 이루어진 숫자 집합의 평균을 계산할 수 있습니다.  
```java
private static double calculateAverageCalories() {
		return menu.stream().collect(averagingInt(Dish::getCalories));
	}
```
종종 이들 중 두 개 이상의 연산을 한 번에 수행해야 할 때도 있습니다.  
이럴 때는 팩토리 메서드 `summarizingInt` 가 반환하는 컬렉터를 사용할 수 있습니다.  
예를 들어 다음은 하나의 요약 연산으로 메뉴에 있는 요소 수, 요소의 칼로리 합계, 평균, 최댓값, 최솟값 등을 계산하는 코드입니다.  
  
```java
private static void printAllInformationOfDish() {
		IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
		System.out.println(menuStatistics);
}
```

### 6.2.3 문자열 연결
컬렉터에 joining 팩토리 메서드를 이용하면 스트림의 각 객체에 toString 메서드를 호출해서  
추출한 모든 문자열을 하나의 문자열로 연결해서 반환합니다.  
  
```java
private static String getShortMenu() {
		return menu.stream().map(Dish::getName).collect(joining());
}
```
joining 메서드는 내부적으로 StringBuilder 이용해서 문자열을 하나로 만듭니다.  
Dish 클래스가 toString 메서드를 포함하고 있다면 map으로 각 요리의 이름을 추출하는 과정을 생략할 수 있습니다.  
```java
  menu.stream().collect(joining());
```
오버로드 된 joining 팩토리 메서드를 이용해서 리스트를 콤마로 구분할 수 있습니다.  
```java
private static String getShortMenuCommaSeparated() {
		return menu.stream().map(Dish::getName).collect(joining(", "));
}
```

다음 절에서는 Collectors.reducing 메서드가 제공하는 범용 리듀싱 컬렉터로도 지금까지 살펴본 모든 리듀싱을 재현할 수 있음을 알아봅니다.  
  
### 6.2.4 범용 리듀싱 요약 연산
지금까지 살펴본 모든 컬렉터는 reducing 팩터리 메서드로도 정의할 수 있습니다.  
즉, `Collectors.reducing`으로도 구현할 수 있습니다. 그럼에도 이전 예제에서 특화된 컬렉터를 사용한 이유는 프로그래밍적 편의성 때문입니다.  
예를 들어 reducing 메서드로 만들어진 컬렉터로도 메뉴의 모든 칼로리 합계를 계산할 수 있습니다.  
```java
private static int calculateTotalCalories() {
		return menu.stream().collect(reducing(0, Dish::getCalories, (Integer i, Integer j) -> i + j));
}
```
Collectors.reducing은 인수 세 개를 받습니다.  
- 첫 번째 인수는 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값입니다(숫자 합계에서는 인수가 없을 때 반환값으로 0이 적합합니다.)  
- 두 번째 인수는 요리를 칼로리 정수로 변환할 때 사용할 변환 함수입니다.  
- 세 번째 인수는 같은 종류의 두 항목을 하나의 합으로 더하는 BinaryOperator입니다.  
  
한 개의 인수를 갖는 reducing은 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않는 상황이 벌어집니다.  
따라서 `Optional<Dish>`를 반환합니다.  
```java
	private static Optional<Dish> calculateTotalCaloriesByOneArgument() {
		return menu.stream()
			.collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
	}
```
collect 메서드는 도출하는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드인 반면 reduce는 두 값을 하나로  
도출하는 불변형 연산이라는 점에서 의미론적인 문제가 일어납니다.  
  
여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져버리므로 리듀싱 연산을 병렬로 수행할 수 없다는 점도 문제입니다.  
이 문제를 해결하려면 매번 새로운 리스트를 할당해야 하고 따라서 객체를 할당하느라 성능이 저하될 것입니다.  
`가변 컨테이너 관련 작업이면서 병렬성을 확보하려면` collect 메서드로 리듀싱 연산을 구현하는 것이 바람직합니다.  
  
Integer의 sum을 참조하면 코드를 좀 더 단순화할 수 있습니다.  
```java
	private static int calculateTotalCaloriesWithMethodReference() {
		return menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum)); // 초깃값, 변환 함수, 합계 함수 
	}
```
counting() 컬렉터도 세 개의 인수를 갖는 reducing 팩터리 메서드를 이용해서 구현할 수 있습니다.  
```java
public static <T> Collector<T, ?, Long> counting() {
		return reducing(0L, e -> 1L, Long::sum);
}
```
다음처럼 스트림의 Long 객체 형식 요소를 1로 변환한 다음에 모두 더할 수 있습니다.  
사용은 다음처럼 할 수 있습니다.  
```java
private static long countMenusByCustomCollector() {
		return menu.stream().collect(counting());
}
```
5장에서는 다음과 같은 연산을 수행할 수 있음을 살펴봤습니다.  
```java
private static int calculateTotalCaloriesWithoutCollectors() {
		return menu.stream().map(Dish::getCalories).reduce(Integer::sum).get();
}
```
`Optional<Integer>`를 반환하는데, 일반적으로는 orElse, orElseGet을 이용하는 것이 좋습니다.  
```java
private static int calculateTotalCaloriesUsingSum() {
		return menu.stream().mapToInt(Dish::getCalories).sum();
	}
```
스트림을 IntStream으로 매핑한 다음에 sum 메서드를 호출하는 방법으로도 결과를 얻을 수 있습니다.  
```java
String shortMenu1 = menu.stream().map(Dish::getName).collect(joining());
String shortMenu2 = menu.stream().collect(reducing((d1, d2) -> d1.getName() + d2.getName())).get();
String shortMenu3 = menu.stream().collect(reducing("", Dish::getName, (s1, s2) -> s1 + s2));
```
실무에서는 joining을 사용하는 것이 가독성과 성능에 좋습니다.  

## 6.3 그룹화
Java8의 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있습니다.  
이번에는 메뉴를 그룹화한다고 가정해봅니다. 예를 들어 고기를 포함하는 그룹, 생선을 포함하는 그룹, 나머지 그룹으로 메뉴를 그룹화할 수 있습니다.  
다음처럼 Collectors.groupingBy 를 이용해서 쉽게 메뉴를 그룹화할 수 있습니다.  
  
```java
Map<Type, List<Dish>> dishesByType = 
		menu.stream().collect(groupingBy(Dish::getType));
```
스트림의 각 요리에서 Dish.Type과 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달했습니다.  
이 함수를 기준으로 스트림이 그룹화되므로 이를 분류 함수라고 부릅니다.  
  
그룹화 연산의 결과로 그룹화 함수가 반환하는 키 그리고 각 키에 대응하는 스트림의 모든 항목 리스트를 값으로 갖는 맵이 반환됩니다.  
  
단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 분류 함수로 사용할 수 없습니다.  
예를 들어 400칼로리 이하는 'diet', 400~700 칼로리를 'normal', 700칼로리 초과를 'fat'요리로 분류한다고 가정해봅니다.  
따라서 람다 표현식으로 필요한 로직을 구현할 수 있습니다.  
```java
enum CaloricLevel {DIET, NORMAL, FAT};

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
			groupingBy(dish -> {
				if (dish.getCalories() <= 400) return CaloricLevel.DIET;
				else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
				else return CaloricLevel.FAT;
			}));
```
그렇다면 요리 종류와 칼로리 두 가지 기준으로 `동시에` 그룹화할 수 있을까요?  
  
### 6.3.1 그룹화된 요소 조작 
요소를 그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요합니다.  
예를 들어 500칼로리가 넘는 요리만 필터한다고 가정해봅니다. 다음 코드처럼 그룹화를 하기 전에 프레디케이트로 필터를 적용해 문제를 해결할 수 있다고 생각할 것입니다.  
```java
Map<Type, List<Dish>> caloricDishesByType = menu.stream().filter(dish -> dish.getCalories() > 500)
			.collect(groupingBy(Dish::getType));
```
단점으로는, 필터 프레디케이트를 만족하는 FISH 종류 요리는 없어 맵에서 해당 키 자체가 사라집니다.  
```java
Map<Type, List<Dish>> caloricDishesByTypeRight = menu.stream()
			.collect(groupingBy(Dish::getType, filtering(dish -> dish.getCalories() > 500, toList())));
```
목록이 비어있는 FISH 항목도 제공됩니다.  
매핑 함수를 이용해 요소를 변환할 수 있습니다.  
```java
Map<Type, List<String>> dishNamesByType = menu.stream()
			.collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
System.out.println(dishNamesByType);
```

flatMapping 컬렉터를 이용하면 각 형식의 요리의 태그를 간편하게 출력할 수 있습니다.  
```java
Map<Type, Set<String>> dishNamesByTag = menu.stream()
			.collect(groupingBy(Dish::getType,
				flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));
```
Set으로 그룹화해서 중복 태그를 제거했습니다.

### 6.3.2 다수준 그룹화
스트림의 항목을 분류할 두 번째 기준을 정의하는 내부 groupingBy를 전달해서 두 수준으로 스트림의 항목을 그룹화할 수 있습니다.  
```java
Map<Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
			groupingBy(Dish::getType, // 첫 번째 수준의 분류 함수
				groupingBy(dish -> { // 두 번째 수준의 분류 함수
					if (dish.getCalories() <= 400)
						return CaloricLevel.DIET;
					else if (dish.getCalories() <= 700)
						return CaloricLevel.NORMAL;
					else
						return CaloricLevel.FAT;
				}))
		);
```
### 6.3.3 서브그룹으로 데이터 수집
요리의 수를 종류별로 계산할 수 있습니다.  
```java
Map<Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));
System.out.println(typesCount);
```
분류 함수 한 개의 인수를 갖는 groupingBy(f) 는 사실 groupingBy(f, toList())의 축약형입니다.  
  
요리의 종류를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 프로그램도 다시 구현할 수 있습니다.  
```java
Map<Type, Optional<Dish>> mostCaloricByType = menu.stream()
			.collect(groupingBy(Dish::getType, maxBy(comparingInt(Dish::getCalories))));
```

#### 컬렉터 결과를 다른 형식에 적용하기
마지막 그룹화 연산에서 맵의 모든 값을 Optional 로 감쌀 필요가 없으므로 Optional을 삭제할 수 있습니다.  
즉, 다음처럼 Collectors.collectingAndThen으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있습니다.  
```java
Map<Type, Dish> mstCaloricByType = menu.stream()
			.collect(groupingBy(Dish::getType,
				collectingAndThen(
					maxBy(comparingInt(Dish::getCalories)), // 감싸인 컬렉터
					Optional::get))); // 변환 함수
```
collectingAndThen은 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환합니다.  
반환되는 컬렉터는 기존 컬렉터의 래퍼 역할을 하며 collect의 마지막 과정에서 변환 함수로 자신이 반환하는 값을 매핑합니다.  
  
Collector 인터페이스 `Collector<T, A, R>` T(요소)를 A에 누적한 다음, 결과 R로 변환해 반환합니다.  












