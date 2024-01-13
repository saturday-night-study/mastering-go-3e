# 인터페이스

<mark style="color:orange;">**인터페이스(Interface)**</mark>는 타입 메서드들을 이용해 구현합니다. 인터페이스는 특정 행위(Behavior)를 정의하며, 같은 행위를 하는 여러 데이터 타입을 다룰때 코드를 간결하게 해줍니다. Go 언어에서는 명시적(Explicit)으로 인터페이스를 구현하는 것이 아니라 인터페이스가 정의하는 타입 메서드를 구현하면 인터페이스 구현을 자동으로 만족(Implicit)하게 됩니다.

이런 특징을 <mark style="color:orange;">**덕 타이핑(Duck typing)**</mark>이라고도 부릅니다. 명시적 인터페이스 구현 대비 덕 타이핑이 갖는 문제는 런타임에 오류가 검출된다는 것입니다. 하지만 **Go 언어의 덕 타이핑은 컴파일 타임에 이루어지기 때문에 문제가 발생하지 않습니다.** 그래서 [**Structual typing**](https://blog.carbonfive.com/structural-typing-compile-time-duck-typing/)이라고 부릅니다.

Go 인터페이스 타입은 구현 해야 할 메서드 집합을 명시하는 방식으로 정의합니다. 특정 타입이 인터페이스를 만족하려면 인터페이스가 정의한 모든 타입 메서드를 구현해야 합니다. 즉, 인터페이스는 메서드를 구현하는 <mark style="color:orange;">**추상 타입(Abstract type)**</mark>이고 인터페이스를 구현한 타입은 인터페이스의 인스턴스로 볼 수 있습니다.

인터페이스를 매개변수로 정의하는 함수에는 인터페이스를 구현한 데이터 타입을 인자로 전달할 수 있습니다. 이를 통해 매개변수의 타입별로 별개의 함수를 정의할 필요가 없어집니다. 최근에는 <mark style="color:orange;">**제네릭(Generics)**</mark>이 추가되면서 인터페이스 외에도 추가적인 선택지가 생겼습니다.

인터페이스를 이용해 객체지향 프로그래밍의 <mark style="color:orange;">**다형성(Polymorphism)**</mark>을 제공할 수 있습니다. 또한 인터페이스를 <mark style="color:orange;">**합성(Composition)**</mark>하여 새로운 인터페이스를 만들 수 있습니다. 물론 인터페이스에 너무 많은 메서드가 존재할 경우 재사용하기 어려우므로 사용 시 주의가 필요합니다.

#### sort.Interface 인터페이스

<mark style="color:orange;">**sort**</mark> 패키지에 <mark style="color:orange;">**sort.Interface**</mark>라는 인터페이스가 정의되어 있습니다.

```go
type Interface interface {
	// 컬렉션의 원소 개수 조회
	// 컬렉션의 모든 원소에 대해서 Less 메서드를 호출하는데 사용
	Len() int

	// i와 j번째 인덱스 원소를 서로 비교하여 어떤게 정렬할지 판단
	// true면 i번째 원소가 j번째 원소보다 앞으로 정렬
	Less(i, j int) bool

	// i와 j번째 인덱스 원소를 교환하여 정렬
	Swap(i, j int)
}
```

sort.Interface를 구현하려면 세 가지 메서드를 구현해야하는데, 이 메서드들은 정렬을 구현하기 위해서 필수적인 기능이라 할 수 있습니다. 블랙핑크 멤버들을 나이순으로 정렬하는 예제입니다.

{% embed url="https://gist.github.com/kineo2k/086a5a43c5ac747f1d95e8f3b21f8d9c" %}

#### 빈 인터페이스

<mark style="color:orange;">**빈 인터페이스**</mark>는 <mark style="color:blue;">**interface{}**</mark>로 정의하는데 아무런 정의도 없으므로 모든 타입이 이미 구현했다고 말할 수 있습니다. 이때문에 어떤 값인지 알수 없는 경우를 처리하는데 이용됩니다. 함수에서 <mark style="color:blue;">**interface{}**</mark> 매개변수를 사용할때는 반드시 데이터 타입을 검증한 후 사용해야합니다. 그렇지 않으면 프로그램이 패닉될 수 있습니다.

{% embed url="https://gist.github.com/kineo2k/99d742df0b92d88d12d0f051c0bacb59" %}

<mark style="color:blue;">**interface{}**</mark>를 이용해 모든 타입을 매개변수로 받을 수 있고, 모든 타입을 반환할 수 있도록 정의할 수 있습니다. 하지만 실제 들어오는 타입을 잘 알아야 하므로 주의가 필요합니다.
