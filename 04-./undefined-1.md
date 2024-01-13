# 타입 메서드

<mark style="color:orange;">**타입 메서드(Type method)**</mark>는 특정 데이터 타입과 연결되는 함수를 말합니다.

타입에 연결된 함수를 통해서 Go에서 어느정도 객체지향 프로그래밍을 흉내낼 수 있습니다. 또한 타입에 인터페이스를 사용하려면 타입 메서드가 필요합니다.

### 타입 메서드 생성

2 x 2 행렬들을 계산하는 타입을 정의하고 행렬 계산을 수행한다고 가정하겠습니다.  <mark style="color:blue;">**ar2x2**</mark>라는 데이터 타입이 있을때 아래와 같은 형태로 타입 메서드를 정의할 수 있습니다.

```go
func (a ar2x2) FunctionName(parameters) <return values> {
    ...
}

var ar ar2x2 = ...
ar.FunctionName(...)
```

함수를 정의할때 <mark style="color:blue;">**(a ar2x2)**</mark>라고 작성하면 <mark style="color:blue;">**ar2x2**</mark> 타입과 연결된 타입 메서드가 됩니다. <mark style="color:blue;">**(a ar2x2)**</mark>는 <mark style="color:orange;">**리시버(Receiver)**</mark>라고 부릅니다. <mark style="color:blue;">**ar2x2**</mark> 타입의 변수에 <mark style="color:orange;">**점(.)**</mark>을 이용하여 타입 메서드를 호출할 수 있으며 이를 <mark style="color:orange;">**선택자(Selector, .)**</mark>라고 부릅니다.

&#x20;<mark style="color:blue;">**FunctionName()**</mark>은 타입 메서드는 말 그대로 메서드이기 때문에 다른 타입이나 일반 함수 처럼 사용할 수 없습니다. 반면, 다른 타입이나 일반 함수로 <mark style="color:blue;">**FunctionName()**</mark>을 정의하는 것은 가능합니다.

```go
func FunctionName(a ar2x2, parameters...) <return value> {
    ...
}
```

컴파일 시점에 Go 컴파일러는 **타입 메서드를 자신의 타입이 첫번째 매개변수로 사용하는 일반 함수로 변경**합니다.

### 타입 메서드 사용
