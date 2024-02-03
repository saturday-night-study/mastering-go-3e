# Go의 객체지향 프로그래밍



## 객체 지향과 Go



객체 지향은 클래스를 기반으로 아래 개념들을 이용하여 코드의 변경을 최소화하고 유지보수를 돕는데요

* 추상화
* 다형성
* 캡슐화
* 상속



Go는 절차지향  언어지만 **다형성**, **캡슐화**, **상속(합성)** 개념을 사용하고 있습니다. 특이하게도  일반적인 객체지향 언어와는 다르게 아래와 같은 이유로 객체지향의 개념을  간소화하고 있습니다. (Go의 목적성에 따라 설계된 부분입니다.)



* 단순함, 가독성&#x20;
  * &#x20;객체 지향의 복잡성, 추상화를 줄여 개발자가 코드를 이해하기 쉽게 만들려고 합니다.성능, 메모리&#x20;
* 안정성, 성능
  * &#x20;객체 지향은 프로그램의 성능을 느리게 하는 불필요하는 요소를 만들수 있습니다. Go의 주 타겟인 CLI 프로그램(유틸리티)과, 서버 소프트웨어 에서는 객체지향의 일부 개념을 제거 하는것만으로도 성능을 향상 시킬 수 있습니다.

> &#x20;([위키](https://ko.wikipedia.org/wiki/Go\_\(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D\_%EC%96%B8%EC%96%B4\)), [기사](https://www.ciokorea.com/news/250552),  GPT 에서 일부 정보를 조합하여 낸 결과, 정확하지 않을 수 있음)



위의 예시를 들면, 클래스 대신 구조체와 메소드 리시버를 이용한 부분이 아래와 같이 성능적 이득을 볼 수 있다고 하네요.

> **성능**:
>
> * 덜 추상적인 구조체와 메서드 설계는 메모리 효율성과 성능 측면에서 이점을 제공합니다. 객체 지향 프로그래밍에서는 종종 불필요한 객체 생성과 메모리 할당이 발생할 수 있습니다. 이로 인해 가비지 컬렉션 작업이 자주 발생하고, 성능이 저하될 수 있습니다.
> * Go의 구조체와 메서드는 명확하게 메모리를 할당하고 관리할 수 있으며, 필요한 경우 메모리를 직접 관리할 수도 있습니다. 이러한 특징은 성능에 도움을 줄 뿐만 아니라 메모리 안정성도 제공합니다.
>
> (GPT)



대략적인 설계 목적과 객체지향을  일부만 지원하는 이유를  확인했으니 이제  코드에서의 구체적인 예시를확인 해 보겠습니다.

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



Go 는 아래와  같이 메소드 리시버를  이용해  캡슐화를 사용할 수 있습니다.

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



