# Go의 객체지향 프로그래밍



## 객체 지향과 Go



* Go 에서는 일부 객체 지향 개념을 일부만 사용합니다.&#x20;



### 다형성

아래와 같이 인터페이스로 추상 적인 데이터 타입을 정의할 수 있습니다.

```go
type IExample interface {
    DoSomething() (string, error)
}

type Example struct {

}

func (e *Example) DoSomething() (string, error) {
	return "something", nil
}

```

\*\* (다형성 개념 확인 필요)

### 캡슐화



아래와  같이 구조체 단위의 캡슐화를 사용할 수 있습니다.

```go
type Example struct {

}

func (e *Example) DoSomething() (string, error) {
	return "something", nil
}

// Go 컴파일러는 아래와 같이 해석합니다. (왜?)
// func (e *Example) DoSomething(e *Example) (string, error) {
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





