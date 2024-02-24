# 함수

`함수(Function)`는 `패키지(Package)`를 이루는 주요 요소입니다.&#x20;

이때 `함수(Function)`는 상호 독립적이며, 단 한 가지의 일만 수행할 수 있도록 정의되어야 합니다. 각 `함수(Function)`간의 역할을 명확하게 구분하고 나눌 수록 의존성이 낮아지고 재 사용성이 높아지기 때문입니다.

## 개요

`함수(Function)`에는 여러가지 타입의 함수가 존재합니다.

가장 사용이 많이 되는 `함수(Function)`는 모든 언어들과 비슷하게 `main()` 함수가 가장 많이 사용됩니다. `main()` 함수의 경우 어떠한 매개변수도 없고 어떠한 값도 반환하지 않습니다.

다만 모든 Go 프로그램의 **시작 점으로 호출 되는** `함수(Function)`입니다. 또한 `main()` 함수가 끝이 나면 프로그램이 종료됩니다.

그 외 **초기화(init) 함수, 익명 함수**, **여러 값을 반환하는 함수**, **함수를 매개변수로 받는 함수**, **함수를 반환하는 함수**, **가변 인수 함수**와 같은 여러가지의 형태로 구현될 수 있습니다.

## Go 코드의  실행 순서

Go 언어에서는 내부적으로 코드를 실행할 때 아래와 같은 순서를 통해 실행하게 됩니다.&#x20;

<figure><img src="../.gitbook/assets/code_flow (1).png" alt="" width="294"><figcaption><p>Go 언어에서의 코드 실행 순서</p></figcaption></figure>

**모든 패키지는 위 규칙을 따르기 때문에 반드시 코드를 작성할 때는 위 순서에 알맞게 코드가 작성되고 본인이 의도한 대로 구성되어 있는지 확인**이 필요합니다.

## 초기화(init) 함수

모든 Go의 패키지는 선택사항으로 `init()` 이라는 Private 함수를 가질 수 있습니다. 이 함수는 패캐지를 실행할 때 자동으로 호출되며, 아래와 같은 특징을 가지고 있습니다.

### 초기화 함수의 특징

* 인수를 가지고 있지 않음
* 어떠한 값도 반환하지 않음
* 사용(선언)하지 않아도 됨
* 내부적으로 자동 호출 함
* `main` 패키지에서 또한 `init()` 함수를 사용할 수 있으며, 이 경우 `main()` 함수를 호출하기 전에 `init()` 함수가 먼저 호출 됨
* 소스 파일 내에는 여러 개의 `init()` 함수를 선언할 수 있고, 이들은 호출 순서대로 실행 됨
* 패키지를 여러 번 import 해도 해당 패키지의 `init()` 함수는 한번만 호출 됨
* Go 패키지는 여러 파일을 가질 수 있고 각 소스 파일에는 하나 이상의 `init()` 함수가 있을 수 있음

{% hint style="warning" %}
`init()` 함수는 Private 함수로 설계 되었기 때문에 `init()`**함수가 속한 패키지가 아닌 곳에서는 호출을 할 수 없습**니다. 또한 패키지 사용자는 `init()` **함수를 직접 제어할 수 없습**니다.&#x20;
{% endhint %}

## 익명 함수

`익명 함수(Anonymous Function)`는 함수 안에서 이름 없이 정의하며, 짧은 코드로 구현할 수 있는 대상을 표현할 때 사용됩니다. 또한 값으로 반환하거나 매개변수로도 활용할 수 있습니다. (변수와의 연결도 가능)

```go
package main

import "fmt"

func main() {
	af := func(param int) int {
		return param * param
	}

	var a = af(2)

	fmt.Println(a)
}
```

```
// 예시 출력
4
```

`익명 함수`는 값으로 반환하거나 매개변수로 사용할 수 도 있습니다. 또한 변수에 연결도 가능합니다.

{% hint style="warning" %}
하지만 익명 함수의 경우 함수 내부의 간단한 처리에만 사용 되어져야 하며, 절 때 남용해서는 안된다. 그 이유는 **"익명"** 이기 때문에 코드 관리가 어려워지고 코드를 재 사용 할 수 없어 좋지 않은 코드가 생길 수 있기 때문이다.
{% endhint %}

## 여러 값을 반환하는 함수

Go 함수는 여러 개의 반환 값을 갖는 함수를 정의할 수 있습니다. 아래 예시에서 볼 수 있듯이 a와 b값을 받아 더한 값, 뺀 값을 반환할 수 있습니다.

```go
package main

import "fmt"

func SumAndSubType1(a int, b int) (int, int) {
	return a+b, a-b
}

func SumAndSubType2(a int, b int) (sum int, sub int) {
	sum = a + b
	sub = a - b
	return
}

func main() {
	var sum1, sub1 = SumAndSubType1(4, 3)
	fmt.Println(sum1, sub1)

	var sum2, sub2 = SumAndSubType2(4, 3)
	fmt.Println(sum2, sub2)
}
```

```
// 예시 출력
7 1
7 1
```

또한 반환하는 값의 순서를 지정하여 활용할 수도 있습니다.

```go
package main

import "fmt"

func BigAndSmall(a int, b int) (int, int) {
	if a > b {
		return a, b
	}
	return b, a
}

func main() {
	var big1, small1 = BigAndSmall(4, 3)
	fmt.Println(big1, small1)

	var big2, small2 = BigAndSmall(3, 4)
	fmt.Println(big2, small2)
}
```

```
// 예시 출력
4 3
4 3
```

## 함수를 매개변수로 받는 함수

Go에서 함수는 다른 함수를 매개변수로 받아올 수 있습니다. 가장 좋은 예시로는 `sort` 패키지가 있습니다. `sort.Slice()` 함수에서는 정렬하는 방법을 명시한 함수를 인수로 받습니다.

`sort.Slice()`의 시그니처는 아래와 같습니다.

```go
func Slice(slice interface{}, less fun(i, j int) bool){
    //...
}
```

위 내용을 간단하게 요약하면,

* sort.Slice()는 아무 데이터도 반환하지 않는다
* sort.Slice()는 인수로 interface{} 타입 슬라이스 및 또 다른 함수가 필요하다
* sort.Slice()의 매개변수로 들어오는 함수의 이름은 less이고, func(i, j int) bool의 시그니처를 가져야한다. (여기서는 매개변수이기 때문에 이름을 가짐)
* less의 i와 j는 slice 매개변수의 인덱스를 의미한다

실제 동작 코드를 확인해 보면,

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	people := []struct {
		Name string
		Age  int
	}{
		{"Gopher", 7},
		{"Alice", 55},
		{"Vera", 24},
		{"Bob", 75},
	}
	sort.Slice(people, func(i, j int) bool { return people[i].Name < people[j].Name })
	fmt.Println("By name:", people)

	sort.Slice(people, func(i, j int) bool { return people[i].Age < people[j].Age })
	fmt.Println("By age:", people)
}
```

```
// 예시 출력
By name: [{Alice 55} {Bob 75} {Gopher 7} {Vera 24}]
By age: [{Gopher 7} {Vera 24} {Alice 55} {Bob 75}]
```

name을 기준으로 비교한 people 값은 name 순으로, age를 기준으로 비교한 people 값은 age 순으로 정렬 됨을 확인할 수 있습니다.

## 함수를 반환하는 함수

말 그대로 반환 값에 함수를 만들어서 반환하는 함수입니다 😵‍💫

글로 표현하니 되게 복잡한 것 같지만 단순히 반환 값에 함수가 들어가 있는 것을 의미합니다. 아래 예시  코드를 통해 조금 더 자세하게 활용 형태를 소개하겠습니다.

```go
package main

import "fmt"

func funcRet(i int) func(int) int {
	if i < 0 {
		return func(k int) int {
			k = -k
			return k + k
		}
	}

	return func(k int) int {
		return k * k
	}
}

func main() {
	funcA := funcRet(5)
	funcB := funcRet(-5)

	fmt.Println(funcA(10))
	fmt.Printf("funcA's Sig: %T, Addr: %v \n", funcA, funcA)

	fmt.Println(funcB(10))
	fmt.Printf("funcB's Sig: %T, Addr: %v \n", funcB, funcB)
}
```

```
// 예시 출력
100
funcA's Sig: func(int) int, Addr: 0x47c680 
-20
funcB's Sig: func(int) int, Addr: 0x47c6a0 
```

위 예시 결과를 볼 수 있듯이 `funcA` 함수와 `funcB` 함수의 각각 메모리 참조 주소가 다름을 볼 수 있고 결과 값 또한 처음 `funcRet` 함수에 넣었던 매개변수에 따라 다른 결과물이 호출 되었음을 확인할 수 있었습니다.

## 가변 인수 함수

`가변 인수 함수(Variadic Function)`는 **인수의 개수가 정해지지 않은 함수**를 의미합니다. 대표적인 예시로 `fmt.Println()`이나 `append()`와 같이 널리 사용되는 함수가 있습니다.

### **가변 인수 함수의 일반적인 개념과 규칙**

* 가변 인수 함수는 `팩 연산자(Pack Operator)`를 사용한다. 팩 연산자는 데이터 타입 앞에 ... 연산자가 위치한 것을 의미하며, 가변 길이의 int 값을 매개변수로 받고 싶다면, int 값 앞에 ... 연산자를 붙인 ... int 와 같은 형태로 활용해야 한다
* 팩 연산자는 주어진 함수에서 딱 한번만 사용할 수 있다
* 팩 연산자로 데이터를 받는 변수는 Slice 형태이다
* 팩 연산자로 선언 된 매개 변수는 반드시 매개변수 목록의 마지막에 위치해야한다
* 팩 연산자는 빈 인터페이스(interface {}) 형태도 지원한다 \
  (fmt 라이브러리에 많은 예제가 있음)

위 내용을 아래 예시 코드를 통해서 조금 더 친근하게 접근해보도록 하겠습니다.

```go
package main

import "fmt"

func addFloats(message string, s ...float64) float64 {
	fmt.Println(message)
	sum := float64(0)
	for _, a := range s {
		sum = sum + a
	}
	return sum
}

func main() {
	sumFloat := addFloats("adding floats", 1.1, 1.2, 3.4, 5.7)

	fmt.Println(sumFloat)
}
```

```
// 예시 출력
adding floats
11.399999999999999
```

## Defer 키워드

`defer` 키워드는 **파일 입출력**, **출력 연산**, **네트워크 통신** 등과 같은 연산 작업에 많이 사용 됩니다. 해당 작업이 완료가 되는 시점을 신경 쓸 필요가 없어지기 때문에 매우 편리한 키워드입니다.

`defer`를 통해 `실행이 미뤄진 함수(Deferred Function)`를 호출하는 순서는 `LIFO(Last In First Out)`입니다.&#x20;

f1(), f2(), f3() 순서로 `defer`문을 실행했다면 f3(), f2(), f1() 순서로 실행이 됩니다.

```go
package main

import "fmt"

func PrintNumberOne() {
	fmt.Println("1")
}

func PrintNumberTwo() {
	fmt.Println("2")
}

func PrintNumberThree() {
	fmt.Println("3")
}

func main() {
	defer PrintNumberOne()
	defer PrintNumberTwo()
	defer PrintNumberThree()
}
```

```
// 예시 출력
3
2
1
```

### defer 키워드 주의 사항

defer 키워드를 사용할 때에 반드시 주의해야하는 점이 있습니다. 바로 익명 함수와 함께 무분별하게 사용 되는 경우입니다.

아래 예제 코드를 통해 발생 할 수 있는 오류를 확인해보도록 하겠습니다.

```go
package main

import "fmt"

func d1() {
	for i := 3; i > 0; i-- {
		defer fmt.Print(i, " ")
	}
}

func d2() {
	for i := 3; i > 0; i-- {
		defer func() {
			fmt.Print(i, " ")
		}()
	}
	fmt.Println()
}

func d3() {
	for i := 3; i > 0; i-- {
		defer func(n int) {
			fmt.Print(n, " ")
		}(i)
	}
}

func main() {
	d1()
	d2()
	fmt.Println()
	d3()
	fmt.Println()
}
```

```
// 예시 출력
1 2 3 
0 0 0
1 2 3
```

여기서 보면 `d2()` 함수의 경우 출력 값이 0로 되어있는데, 이는 **익명 함수의 경우 외부 변수인 i를 호출하는 시점에는 이미 for 반복문이 완료**되어 i 값이 0으로 고정되어 있는 상태이기에 3번 호출 된 출력 결과에서 모두 0을 보여주고 있습니다.
