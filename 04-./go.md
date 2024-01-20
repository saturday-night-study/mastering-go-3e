# Go의 객체지향 프로그래밍



## 객체 지향과 Go



* Go 에서는 일부 객체 지향 개념을 일부만 사용합니다.&#x20;



### 다형성

아래와 같이 인터페이스로 추상 적인 데이터 타입을 정의할 수 있습니다.

<pre class="language-go"><code class="lang-go"><strong>// 추상 타입 정의
</strong><strong>type IExample interface {
</strong>    DoSomething() (string, error)
}

// 구현체 정의
type Example struct {

}

func (e *Example) DoSomething() (string, error) {
	return "something", nil
}

</code></pre>

* 다형성이란 '서로 다른 유형의 객체가 **동일한 메시지**(인터페이스로 구현한 메서드)에 대해 다르게 반응 하는것' 이라고 설명 하게 되는데요,&#x20;
* 인터페이스를 작성하는 것을 '추상화(추상 타입 정의)' ,  메서드를 구현하는것을 '구현체 정의' 라고 합니다.
* Go 에서는 단순함을 위해'덕  타이핑'  (="만약 어떤 새가 오리처럼 걷고, 오리처럼 소리를 낸다면, 그 새는 오리일 것이다"라는 논리) 으로 다형성을 실현하는데요, 그 객체가 어떤 메서드를 가지고 있는지, 어떻게 행동하는지에 초점을 맞추기 때문에. 인터페이스와 구현체가 서로에 대해 알 필요가 없고, 알 수 없다는 것입니다.&#x20;



### 캡슐화



Go 는아래와  같이 메소드 리시버를  이용해  캡슐화를 사용할 수 있습니다.

```go
type Example struct {

}

// (e Example) -> 메소드 리시버

// Value 리시버, E의 필드에 영향을 주지 않음.
// 메서드 호출이 객체의 복사본을 만드는 데 큰 비용이 들지 않을 때 유용
func (e Example) DoSomething() (string, error) {
	return "something", nil
}

// Pointer 리시버
// 객체의 상태를 변경하거나, 구조체가 클 때 메모리 복사를 방지하기 위해 사용
func (e *Example) DoSomething() (string, error) {
	return "something", nil
}

// 책에서Go 컴파일러는 아래와 같이 변환하도록 내부 구현이 되어있다.
// func ExampleDoSomething(e *Example) (string, error) {
// 	return "something", nil
// }

```

\*\* 캡슐화 : 변수와, 메서드를 묶어 하나의 단위로 묶어 외부에서 직접 접근하지 못하도록 하는 행위



### 합성

###

아래와 같이 인터페이스를 합성 할 수 있습니다.

```go
// 패키지 A
type IExample01 interface {
    DoSomething01() (string, error)
}

// 패키지 B
type IExample02 interface {
    DoSomething02() (string, error)
}

// 패키지 C
type IExampleMerged interface {
    IExample01
    IExample02
}

// 인터페이스를 구현
type Example struct {

}

func (e *Example) DoSomething01() (string, error) {
	return "something1", nil
}

func (e *Example) DoSomething02() (string, error) {
	return "something2", nil
}
```

* 딱히 와닿는 느낌이 없습니다. 반드시 필요한지... 모듈성 재사용성 증대, 유연성 확장성 향상이라고 하는데. 덕 타이핑이 있는 상태에서필수적으로 사용할 만큼 유용할까요?



## 참고자료

* 추상화 와 다형성 :  [https://bugoverdose.github.io/development/abstraction-and-polymorphism-in-go/](https://bugoverdose.github.io/development/abstraction-and-polymorphism-in-go/)
*

