# 리플렉션

리플렉션은 Go의 고급 기능입니다. 리플렉션(Reflection)을 이용하면 실행 시점(Run-time)에 구조체의 정보를 알아낼 수 있습니다. <mark style="color:orange;">**reflect**</mark> 패키지로 리플렉션을 제공합니다.

리플렉션은 강력한 기능이지만 읽기 좋은 코드를 만들어주지는 않으므로 꼭 필요한 경우에만 사용할 수 있도록 해야합니다.

### 왜 Go에 리플렉션이 있을까?

동적으로 오브젝트 구조에 대한 정보를 알수 있기 때문입니다. `fmt.Println()` 함수는 매개변수의 데이터 타입에 따라 다르게 동작하는데, 이를 위해서 <mark style="color:orange;">**fmt**</mark> 패키지에서 리플렉션을 사용합니다.

{% embed url="https://gist.github.com/kineo2k/bf845fb1e926bdf7be8f26a193fda6fe" %}

### 리플렉션은 언제 사용하는가?

컴파일 타임(Compile-time)에 존재하지 않지만 런타임에 존재하는 데이터 타입을 처리할 수 있게해줍니다. 예를 들어 REST API나 설정 파일로부터 JSON 문자열을 읽어온후 구조체 오브젝트로 바인딩하는 경우에 리플렉션을 활용할 수 잇습니다.

#### 리플렉션에서 Go 타입과 값

<mark style="color:orange;">**reflect**</mark> 패키지에서는 `reflect.Type`과 `reflect.Value`를 주로 사용합니다.

* reflect.Type
  * 오브젝트의 타입을 표현
  * 특히 Go `interface{}`에는 어떤 타입이든 담을 수 있으므로 실제로 어떤 값이 있는지 확인할때 필요
  * `reflect.TypeOf(i any)`는 reflect.Type을 반환, <mark style="color:blue;">**i**</mark>에 `nil`을 넘기면 `nil`을 반환
* reflect.Value
  * 타입의 값을 저장
  * `reflect.ValueOf(i any)`는 reflect.Value를 반환, <mark style="color:blue;">**i**</mark>에 <mark style="color:blue;">**nil**</mark>을 넘기면 <mark style="color:blue;">**nil**</mark>을 반환

구조체의 필드 정보도 제공합니다.

* `reflect.NumField()` : 구조체의 필드 개수를 반환
* `reflect.Field()` : 구조체의 특정 필드의 reflect.Value값을 조회

`reflect.Kind`라는 타입이 있는데 reflect.Type이 나타내는 타입이 어떤 자료형으로 이루어졌는지 표현합니다.

{% embed url="https://gist.github.com/kineo2k/247734af4f85f82f0e5e979b8f857d59" %}

{% embed url="https://gist.github.com/kineo2k/da0b0b5648e1f8948bc228fedbe51639" %}

리플렉션을 이용해 문자열, 정수, 함수, 구조체, 포인터 유형의 값을 출력하는 예제입니다. default 구문에서 출력하는 Kind와 Type의 타이를 확인해보시기 바랍니다.

### Go 구조체의 내부

다음 예제는 리플렉션으로 Go 구조체 내부 구조와 필드 정보를 조회하는 방법을 보여줍니다.

{% embed url="https://gist.github.com/kineo2k/2cf8a9566e24e8752ee47f88f70316e3" %}

`main.Record, main.Secret`은 구조체의 완전한 이름으로 패키지의 이름은 main이고 구조체의 이름은 Record와 Secret입니다. Go에서 서로 다른 패키지에 동일한 이름의 구조체가 있어서 서로 구분이되는 것은 이와 같이 이름을 정하기 때문입니다.

위 예제에서는 구조체의 정보만 조회할 뿐 아무것도 변경하지 않습니다. 다음 절에서 구조체의 필드 값을 변경해 보겠습니다.

### 리플렉션을 이용해 구조체 값 바꾸기

리플렉션으로 구조체의 정보를 파악하는 것도 유용하지만, 값을 변경하는 것도 가능합니다.

{% embed url="https://gist.github.com/kineo2k/9b974cc89cce25af11a2bc68133fe430" %}

오브젝트 A의 값을 런타임에 동적으로 변경했습니다. 이처럼 리플렉션을 이용해 오브젝트의 값을 동적으로 변경할 수 있습니다.

### 리플렉션의 세 가지 단점

1. 리플렉션은 코드를 이해하기 어렵게 만들어 관리를 힘들게 합니다. 문서화로 어느정도 해소할 수 있으나 좋은 문서를 작성하고 유지보수 하는 것 또한 어려운 일입니다.
2. 리플렉션은 실행 속도를 느리게합니다. 구체적으로 타입이 지정된 코드가 리플렉션을 이용해 타입을 동적으로 다루는 코드보다 훨씬 빠릅니다.
3. 리플렉션과 관련된 에러는 런타임에 패닉(Panic)이 되어야 확인할 수 있습니다. 컴파일 타임에 검출할 수 없으므로 안정성이 떨어지고 리팩터링이나 코드 정적 분석의 지원을 받기 힘듭니다.

