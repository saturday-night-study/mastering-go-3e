# 인터페이스

<mark style="color:orange;">**인터페이스(Interface)**</mark>는 타입 메서드들을 이용해 구현합니다. 인터페이스는 특정 행위(Behavior)를 정의하며, 같은 행위를 하는 여러 데이터 타입을 다룰때 코드를 간결하게 해줍니다. Go 언어에서는 명시적(Explicit)으로 인터페이스를 구현하는 것이 아니라 인터페이스가 정의하는 타입 메서드를 구현하면 인터페이스 구현을 자동으로 만족(Implicit)하게 됩니다.

이런 특징을 <mark style="color:orange;">**덕 타이핑(Duck typing)**</mark>이라고도 부릅니다. 명시적 인터페이스 구현 대비 덕 타이핑이 갖는 문제는 런타임에 오류가 검출된다는 것입니다. 하지만 **Go 언어의 덕 타이핑은 컴파일 타임에 이루어지기 때문에 문제가 발생하지 않습니다.** 그래서 [**Structual typing**](https://blog.carbonfive.com/structural-typing-compile-time-duck-typing/)이라고 부릅니다.

Go 인터페이스 타입은 구현 해야 할 메서드 집합을 명시하는 방식으로 정의합니다. 특정 타입이 인터페이스를 만족하려면 인터페이스가 정의한 모든 타입 메서드를 구현해야 합니다. 즉, 인터페이스는 메서드를 구현하는 <mark style="color:orange;">**추상 타입(Abstract type)**</mark>이고 인터페이스를 구현한 타입은 인터페이스의 인스턴스로 볼 수 있습니다.

인터페이스를 매개변수로 정의하는 함수에는 인터페이스를 구현한 데이터 타입을 인자로 전달할 수 있습니다. 이를 통해 매개변수의 타입별로 별개의 함수를 정의할 필요가 없어집니다. 최근에는 <mark style="color:orange;">**제네릭(Generics)**</mark>이 추가되면서 인터페이스 외에도 추가적인 선택지가 생겼습니다.

인터페이스를 이용해 객체지향 프로그래밍의 <mark style="color:orange;">**다형성(Polymorphism)**</mark>을 제공할 수 있습니다. 또한 인터페이스를 <mark style="color:orange;">**합성(Composition)**</mark>하여 새로운 인터페이스를 만들 수 있습니다. 물론 인터페이스에 너무 많은 메서드가 존재할 경우 재사용하기 어려우므로 사용 시 주의가 필요합니다.

### sort.Interface 인터페이스

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

### 빈 인터페이스

<mark style="color:orange;">**빈 인터페이스**</mark>는 `interface{}`로 정의하는데 아무런 정의도 없으므로 모든 타입이 이미 구현했다고 말할 수 있습니다. 이때문에 어떤 값인지 알수 없는 경우를 처리하는데 이용됩니다. 함수에서 `interface{}` 매개변수를 사용할때는 반드시 데이터 타입을 검증한 후 사용해야합니다. 그렇지 않으면 프로그램이 패닉될 수 있습니다.

{% embed url="https://gist.github.com/kineo2k/99d742df0b92d88d12d0f051c0bacb59" %}

`interface{}`를 이용해 모든 타입을 매개변수로 받을 수 있고, 모든 타입을 반환할 수 있도록 정의할 수 있습니다. 하지만 실제 들어오는 타입을 잘 알아야 하므로 주의가 필요합니다.

### 타입 단언과 타입 스위치

<mark style="color:orange;">**타입 단언(Type assertion)**</mark>은 **인터페이스의 값을 특정 타입으로 사용할 수 있게해주는 기능**입니다. 인터페이스는 타입의 행위만 정의할 뿐 값을 정의하지 않기 때문에 **특정 타입의 값을 사용**하고자 할때 필요합니다.

타입 단언은 `x.(T)` 형식으로 표기하며 x는 인터페이스 타입, T는 특정 타입을 나타냅니다. 타입 단언을 수행하면 x의 **실제 값과 `bool` 타입의 성공 여부를 반환**합니다. 성공 여부가 `true`인 경우만 값을 활용해야하며, `false`인 경우 실제 값을 추출하지 못했다는 것을 의미합니다.

{% embed url="https://gist.github.com/kineo2k/957f7b0fa35801c268d4ebf297c0f045" %}

출력 결과를 보면 `Entry` 타입이 아닌 타입에 대해서도 `T`는 **main.Entry** 타입으로 변환이되고 값을 추출하는데 실패했으므로 제로값으로 초기화되어 있습니다. 성공적으로 값을 추출하는데 실패했으므로 `T`값을 사용해서는 안됩니다.

그런데 만약 어떤 타입인지 모르는 상황에서 여러 타입에 대한 지원을 해야한다면 어떻게 해야할까요? 이럴땐 <mark style="color:orange;">**타입 스위치(Type switch)**</mark>를 이용해서 타입을 구분하여 사용할 수 있습니다. 타입 스위치는 `switch` 구문과 타입 단언을 사용하여 표현합니다.

{% embed url="https://gist.github.com/kineo2k/de37ddad8638666c8af58451390dc0a6" %}

타입 단언을 이용해서 값을 가져오는 다음 3가지 경우에 대한 예제를 살펴보겠습니다.

* 타입 단언을 이용해서 올바르게 값을 가져오는 경우
* 타입 단언이 우연히 성공하는 경우
* 타입 단언이 실패하는 경우

{% embed url="https://gist.github.com/kineo2k/c1fa9aa4f1e17f3676172975c9d35058" %}

### map\[string]interface{} 맵

사전에 정의되지 않은 JSON 처럼 형태를 알 수 없는 입력을 처리할때 `map[string]interface{}`를 유용하게 이용할 수 있습니다. 예제 코드를 살펴보겠습니다.

{% embed url="https://gist.github.com/kineo2k/07f6ebe8e7808eb505e42271872313ff" %}

`typeSwitch()` 함수는 타입 스위치를 이용해서 맵 값들을 구분합니다. 값들 중 map이 있다면 `typeSwitch()` 함수를 재귀적으로 호출하여 값을 조사합니다.

### 에러 타입

<mark style="color:orange;">**error 타입**</mark>도 아래와 같이 인터페이스로 정의합니다.

```go
type error interface {
    Error() stirng
}
```

`Error() string` 타입 메서드를 구현한다면 `error` 인터페이스를 만족하는 것이며, `error` 인터페이스를 구현하면 Go 언어에서 에러를 사용 하는 방식을 만족할 수 있습니다.

Go 언어에서는 파일에 더 이상 읽을 데이터가 없을때 `io.EOF` 에러를 반환합니다. 만약 특정 상황에서 파일을 전부 읽어서 파일 읽기를 실패한 경우와 빈 파일이라서 읽기에 실패한 경우를 구분하려고 한다면 어떻게 해야할까요? `error` 인터페이스를 이용해서 해결해 보겠습니다.

{% embed url="https://gist.github.com/kineo2k/0bdd935c707a368fd862e1e5b11c497f" %}

### 나만의 인터페이스 만들기

커스텀 인터페이스는 아래와 같이 정의합니다.

```go
type Shape2D interface {
    Perimeter() float64
}
```

* `Shape2D`라는 이름을 갖는다.
* `Perimeter() float64` 메서드를 구현하면 `Shape2D` 인터페이스를 만족한다.

### Go 인터페이스 사용

`Shape2D` 인터페이스를 사용하는 간단한 예제를 살펴보겠습니다.

{% embed url="https://gist.github.com/kineo2k/cfaa79b4b97f70962b7daf5a5d9f9852" %}

`shapes`가 `Shape2D` 인터페이스 타입인지를 검사하여 면적을 계산합니다. 58번째 라인에서 `c` 인스턴스가 `Shape2D` 인터페이스를 구현하는지 확인하기 위해서 `interface{}(c).(Shape2D)` 표현식을 사용했습니다.&#x20;

### 3차원 모형 데이터에서 sort.Interface 구현

3차원 모형과 모협의 부피를 계산하는 인터페이스를 정의해보겠습니다. 인터페이스를 이용해서 하나의 슬라이스에 인터페이스를 만족하는 모든 종류의 구조체 인스턴스를 저장해 보겠습니다. 또한 `sort.Interface`를 이용해서 저장된 모형의 부피를 정렬해보도록 하겠습니다.

{% embed url="https://gist.github.com/kineo2k/826d845457dc64f7ba8f4e92389291fa" %}
