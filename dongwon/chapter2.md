# Chapter 2. 동작 파라미터화 코드 전달하기


동작 파라미터화 - 아직 어떻게 실행할지 정해지지 않은 코드 블록(코드 블록은 나중에 프로그램에서 호출되어 실행이 미뤄져, 나중에 실행될 메서드의 인스로 코드 블록을 전달할수 있음)

동작 파라미터화를 통해서 원하는 동작을 메서드의 인수로 전달할수 있지만 쓸데없는 코드가 늘어난다

첫번째 시도 : 녹색 사과 필터링

```java
사과 색을 정의하는 Color num

euum Color {Red, GREEN}

public static List<Apple> filterGreenAppples(List<Apple> inventory) {
		List<Apple> result = new ArrayList<>(); <- 사과 누적 리스트
		for(Apple apple : inventory) {
				if (**GREEN.equals(apple.getColor**()) { <- 녹색 사과만 선택
						result.add(apple);
				}
		}
		return result;
}
```

**`GREEN.equals(apple.getColor**()` 녹색사과 필요한 조건에서 만약 빨간 사과도 필터링한다면 새로운 filterRedApple 메서드를 만들어서 해야한다

두번째 시도 : 색 파라미터화

filterGreenApples 코드 반복하지 않고 filterRedApple 구현하기 위해서 색을 파라미터에 추가

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, **Color color**)
{
		List<Apple> result = new ArrayList<>();
		for (Apple apple : inventory) {
				if (**apple.getColor().equals(color)** ) {
						result.add(apple);
				}
		}
		return result;
}

다음과 같이 메서드 호출 가능해짐

List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

여기에 더해 더 많은 조건이 추가 된다면 메서드에 파라미터를 추가하면 되겠지만 각 필터링 코드가 중복이 되어진다. 그리하여 코드를 뜯어고치게 된다면 비용소모가 클 것이다

필터링 코드를 하나로 묶어 filter 메서드로 합치게 된다면 어떤 필터 기준으로 구분하는지 플래그 추가 할 수 있을거다(but 이 방법은 추천하지 않는다)

세번째 시도 : 가능한 모든 속성 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, booean flag) {
		List<Apple> result = new ArrayList<>();
		for (Apple apple : inventory) {
				if (**(flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > we                ight)) { // <- 색과 무게를 선택하는 방법**
						result.add(apple);
				}
		}
		return result;
}

메서드를 호출할때는

List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);

각각의 값들이 어떤것을 가리키는지 알수없으며, 바뀌어질 요구사항에 유연하게 대처할수가 없다
문제가 잘 정의되어 있다면 좋겠지만 요구사항은 바뀔수다 있다는것 또한 어떠한 기준으로 필터링 할 것인지 효과적으로 전달하면 더 좋은 코드가 될것이다
```

### 동작 파라미터화를 통한 유연성 얻기

ex) 선택조건을 사과의 어떠한 속성에 따라 boolean값을 반환할수 있을것

이러한 참, 거짓을 반환하는 함수를 프레디케이트(Predicate) 라 함

선택조건을 결정하는 인터페이스

```java
public interface ApplePredicate {
		boolean test (Apple apple);
}

위와 같이 다양한 선택 조건 ApplePredicate 정의할 수 있을 거다
-------------------------------------------------------------------------------------
public class AppleHeav_WeightPredicate implement ApplePredicate { <- 무거운 사과 조건
		public boolean test(Apple apple) {
				return apple.getWeight() > 150;
		}
}

public class AppleGreen_CcolorPredicate implements ApplePredicate { <- 녹색사과만
		public boolean test(Apple apple) {
				return GREEN.equals(apple.getColor());
		}
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/732d0c28-4004-4116-bbe1-7197465a697b/Untitled.png)

조건에 따라 filter 메서드가 다르게 동작하게 되는 것이다. 이를 **디자인 패턴** 이라 부른다

(알고리즘 패밀리를 정의 해둔뒤 런타임에 알고리즘을 선택하는 기법 `Apple_Predicate`알고리즘 패밀리, `Apple_HeavyWeightPredicate` , `Apple_GreenColorPredicate` 전략)

`Apple_Predicate`가  다양한 동작을 수행하기 위해 filter_Apples 에서 `Apple_Predicate` 객체를 받아 애플의 조건을 검사하도록 메서드 고쳐야 함

이것을 **동작파라미터화** 라고 부르며 풀어서 메서드가 다양한 동작을 **받아서** 내부적으로 다양한 동작을 **수행**하는 것이다. 위의 예에서는

컬랙션을 반복하는 로직 / 컬렉션의 각 요소에 적용할 동작(Predicate) 을 분리할수 있다

네번째 시도 : 추상적 조건 필터링

```java
Apple_Predicate 이용한 필터 메서드

public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
		List<Apple> result = new ArrayList<>();
		for(Apple apple: inventory) {
				if(**p.test(apple)**) { <- Predicate 객체로 사과 검사 조건을 캡슐화
						result.add(apple);
				}
		}
		return result;
}

유연하고 가독성이 좋아짐
```

(조건 150g 넘는 빨간 사과 검색)

```java
ApplePredicate 구현하는 클래스 만들기(Apple의 모든 속성 변화에 대응이 가능해짐)

public class AppleRed_HeavyPredicate implements ApplePredicate {
		public boolean test(Apple apple) {
				return RED.equals(apple.getColor()) && apple.getWeight() > 150;
		}
}

List<Apple> red_HeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());

ApplePredicate 객체에 의해 filterApples 메서드의 동작이 결정됨
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b54dcb82-3bac-4476-83ca-241a12099ec8/Untitled.png)

핵심은 test 메서드인데 ApplePredicate 객체로 감싸서 전달해야 해서 코드를 전달하는 것과 같다

**동작 파라미터화**는 이와같이 각 항목에 적용할 동작을 분리할 수 있다는 것이 강점이다, 그래서 메서드가 다른 동작을 다시 활용할수 있다(→ 유연한 API를 만들 때 동작 파라미터화가 중요한 역할을 함)

**복잡한 과정 간소화**

> 아 이제는 여러 클래스 구현해서 인스턴스화 과정도 거추장스럽다. 어떻게 개선 가능한가?
> 

filterApples 메서드로 새로운 동작 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스 정의해서 인스턴스화 해야한다

```java
public class { 무거운 사과 선택 클래스}
public class { 녹색 사과 선택 클래스}

List<Apple> heavyApples = filterApples(무거운 사과 선택 결과)->리스트는 155 사과한개 포함
List<Apple> heavyApples = filterApples(녹색 사과 선택 결과)->리스트는 녹색 사과두개 포함
```

개선하기 위해서 클래스의 선언과 동시에 인스턴스화를 할수 있는 **익명 클래스**가 있다

즉, 즉석에서 필요한 것을 바로 만들어 사용이 가능하다

다섯 번째 시도 : 익명 클래스 사용

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() { <- filterApples 메서드의 동작 직접 파라미터화

	public boolean test(Apple apple) {
				return RED.equals(apple.getColor());
	}
});	

GUI 애플리케이션 이벤트 핸들러 객체 구현할 때 익명 클래스 종종 사용함
하지만 익명 클래스는 여전히 많은 공간을 차지함(노란 글씨)

**List<Apple> redApples = filterApples(inventory, new ApplePredicaate() {
		public boolean test(Apple a)** {
				return RED.equals(a.getColor());
		}
});

**button.setOnAction(new EventHandler<ActionEvent>() {
		public void handle(ActionEvent event)** {
				System.out.println("Whoooo a click!!"));
		}
}
```

여섯 번째 시도 : 람다 표현식 사용

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));

매우 간결하면서 유연해짐
```

일곱 번째 시도 : 리스트 형식으로 추상화

```java
public interface Preicate<T> {
		boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) { <- 형식 파타미터 T
		List<T> result = new ArrayList<>();
		for(T e: list) {
				if(p.test(e)) {
						result.add(e);
				}
		}
		return result;
}

람다식으로
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));

List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

<aside>
❓ 동작 파라미터화 패턴은 동작을 캡슐화(한 조각의 코드)해서 메서드로 전달해 동작을 파라미터화 한다. 자바 API의 많은 메서드를 다양한 동작으로 파라미터화 할 수 있을 거다

</aside>

**Comparator로 정렬**

sort 동작을 파라미터화

```java
public interface Comparator<T> {
		int compare(T o1, T o2);
}

Comparator를 구현해서 sort 메서드의 동작 다양화 가능

****익명클래스 이용
inventory.sort(new Comparator<Apple>() {
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a2.getWeight());
		}
});

람다 이용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

Runnable로 코드 블록 실행

자바 스레드를 이용해 병렬로 코드 블록 실행가능, 나중에 실행할 코드 구현 방법 필요(자바 8까지 Thread 생성자에 객체만 전달가능해서 결과를 반환하지 않는 void 메소드 포함 익명 클래스가 Runnable 인터페이스 구현하는 것이 일반적)

```java
// java.lang.Runnable
public interface Runnable {
		void run();
}

Runnable이용
Thread t = new Thread(new Runnable() {
		public void run() {
				System.out.println("Hello world");
		}
});

람다로 표현
Thread t = new Thread(() -> System.out.println("Hello world"));
```

 

Callable 결과로 반환

자바 5부터 있는 실행서비스(ExecutorService)는 태스크 제출과 실행 과정의 연관성 끊어줌ExecutorService를 이용하면 태스크를 스레드 풀로 보내고 결과를 Future로 저장 가능. 이 점이 Runnable과 다르다. 

```java
Callable 인터페이스를 이용한 결과 반환 태스크

// java.util.concurrent.Callable
public interface Callable<V> {
		V call();
}

(실행서비스)
ExecutorService service = Executors.newCached_ThreadPool();
Future<String> threadName = service.submit(new Callable<String>() {
		@Override
				public String call() throws Exception {
				return Thread.currentThread().getName();
		}
}); // 실행결과 태스크 실행하는 스레드의 이름을 반환

람다 이용
Future<String> threadName = service.submit( () -> Thread.currentThread().getName());
```

GUI이벤틑 처리

사용자의 전송 버튼 클릭 → 동작 로그 파일 저장, 팝업표시

**변화에 대응할수 있는 유연한 코드 필요(모든 동작에 반응)**

ex ) 자바 FX → setOnAction 메서드에 EventHandler 전달해 이벤트에 반응

```java
Button button = new Button("Send");
button.setOnAction(new EventHandler<ActionEvent>() {
		public void handle(ActionEvent event) {
				label.setText("Sent!!");
		}
});

EventHandler는 setOnAction 메서드의 동작을 파라미터화 함
람다 표현
button.setOnAction((ActionEvent event) -> label.setText("Sent!!"));
```

정리

- **동작 파라미터화**는 메서드 내부적으로 다양한 동작을 위한 코드를 **메서드 인수로 전달**
- 변화하는 요구사항에 유연한 대처를해 비용줄임
- 코드 전달 기법을 통해 익명 클래스 사용가능
- (정렬, 스레드, GUI 처리) 자바 API 는 다양한 동작으로 파라미터화 가능
