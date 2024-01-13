# 구조체

## 개요

Go 언어에서 사용되는 `구조체(Struct)`는 매우 강력하다. 그 이유는 여러 타입의 데이터를 같은 이름을 사용하여 모으고 조직화 할 수 있기 때문이다.

`구조체(Struct)`는 다른 데이터 타입과는 다르게 조금  더 유연하며, 함수와의 연결이 가능하다. 이를 `메서드(method)`라고 명칭하고 있다.

{% hint style="info" %}
사용자 정의 타입인 `구조체(Struct)`는 `main()` 함수 또는 패키지 함수 밖에 정의 되는 것이 좋다. (함수 외부에 구현 시 재 사용성을 높일 수 있음)
{% endhint %}

## 구조체 정의 및 사용방법

### 구조체 변수 정의

* `구조체(Struct)`를 정의하면 여러 값의 집합을 하나의 데이터 타입으로 묶어 하나의 `객체(Object)` 처럼 주고 받을 수 있음
* `구조체(Struct)`는 `필드(field)`를 가지고 있고 각각의 `필드(field)`는 데이터의 타입을 가지고 있음
* `구조체(Struct)`는 새로운 데이터 타입이기에 선언할 때 항상 `type 이름 struct` 형태의 구조로 선언

```go
package main

import "fmt"

type Job struct {
	JobCode uint
	JobName string
}

type Stat struct {
	Str  uint
	Dex  uint
	Int  uint
	Luck uint
}

type Character struct {
	Name  string
	Level uint
	Job   Job
	Stat  Stat
}

func main() {
	aCharacter := Character{"zl존S2도적", 40, Job{1, "도적"}, Stat{4, 25, 4, 120}}
	fmt.Println(aCharacter.Name, "( Lv.", aCharacter.Level, ") - [", aCharacter.Job.JobName, "]")
}
```

```
// 출력 결과
zl존S2도적 ( Lv. 40 ) - [ 도적 ]
```

### 구조체 변수를 다루는 방식2 가지

`구조체(Struct)`를 다룰 때에는 `일반 변수(Regular Variables)`와 `포인터 변수(Pointer Variables)` 두 가지가 있다.

* `일반 변수(Regular Variables)`와 `포인터 변수(Pointer Variables)` 둘다 초기화를 위한 별도의 함수를 통해서 관리함 \
  &#x20; → 함수를 이용하면 일부 또는 모든 필드 초기화 가능\
  &#x20; → 구조체 변수 초기화 이전에 별도의 로직 구현 가능
* `일반 변수(Regular Variables)`의 경우에는 말 그대로 초기화 후 반환 값을 일반 구조체 변수로 전달, `포인터 변수(Pointer Variables)`의 경우 반환 값이 구조체 변수의 포인터 주소를 반환 함

### 구조체 변수 초기화 방식 3가지

구조체는 아래 3가지 방식으로 초기화를 지원한다.

1. 변수 선언시 Go 컴파일러가 초기화 하는 방식
2. 커스텀으로 생성하여 사용자가 임의 구성하는 방식
3. `new` 키워드를 통한 초기화 방식

{% hint style="info" %}
Go 컴파일러는 변수의 초기 값을 따로 지정하지 않으면 자동으로 해당 타입의 `초기 값(Zero Value)`으로 초기화 함
{% endhint %}

```go
// 위 예시 코드의 부분 추가

func newCharacter(Name string, Level uint) Character {
	// 조건을 통해서 Level 10이하는 초보자로 초기화
	if Level < 10 {
		return Character{Name, Level, Job{0, "초보자"}, Stat{4, 4, 4, 4}}
	}
	// 그 이외는 도적으로 초기화 가능
	return Character{Name, Level, Job{1, "도적"}, Stat{4, 25, 4, 120}}
}

func main() {
	// 변수 선언시 Go 컴파일러가 초기화 하는 방식
	aCharacter := Character{"zl존S2도적", 40, Job{1, "도적"}, Stat{4, 25, 4, 120}}
	fmt.Println(aCharacter.Name, "( Lv.", aCharacter.Level, ") - [", aCharacter.Job.JobName, "]")

	// 커스텀으로 생성하여 사용자가 임의 구성하는 방식
	var bCharacter = newCharacter("저는초보자입니다", 7)
	fmt.Println(bCharacter.Name, "( Lv.", bCharacter.Level, ") - [", bCharacter.Job.JobName, "]")

	// new 키워드를 통한 초기화 방식
	cCharacter := new(Character)
	cCharacter.Name = "뉴캐릭터"
	cCharacter.Level = 9999
	cCharacter.Job = Job{999, "GM"}
	cCharacter.Stat = Stat{9999, 9999, 9999, 9999}
	fmt.Println(cCharacter.Name, "( Lv.", cCharacter.Level, ") - [", cCharacter.Job.JobName, "]")
}
```

```
// 결과 값
zl존S2도적 ( Lv. 40 ) - [ 도적 ]
저는초보자입니다 ( Lv. 7 ) - [ 초보자 ]
뉴캐릭터 ( Lv. 9999 ) - [ GM ]
```

### 생성자(Constructor) 함수

일반적으로 Go 언어에서는 위 3가지 방식도 있지만 `생성자(Constructor) 함수`를 직접 구현하여 사용하며 반환할 때의 값은 `구조체(Struct)`의 `포인터 변수(Pointer Variables`이다.

```go
package main

import "fmt"

type job struct {
	JobCode uint
	JobName string
}

type stat struct {
	Str  uint
	Dex  uint
	Int  uint
	Luck uint
}

type character struct {
	Name  string
	Level uint
	Job   job
	Stat  stat
}

func newCharacter() *character {
	c := character{}
	c.Job = job{999, "GM"}
	c.Stat = stat{9999, 9999, 9999, 9999}
	return &c
}

func main() {
	// 생성자 함수를 통한 생성
	cCharacter := newCharacter()
	cCharacter.Name = "뉴캐릭터"
	cCharacter.Level = 9999
	fmt.Println(cCharacter.Name, "( Lv.", cCharacter.Level, ") - [", cCharacter.Job.JobName, "]")
}
```

```
// 결과 값
뉴캐릭터 ( Lv. 9999 ) - [ GM ]
```

## 구조체의 슬라이스

또한 `구조체(Sturct)`는 `슬라이스(Slice, [])` 형태로 구성하여 여러개의 `구조체(Struct)`를 하나의 변수로 묶어서도 사용할 수 있다.

```go
package main

import "fmt"

type Job struct {
	JobCode uint
	JobName string
}

type Stat struct {
	Str  uint
	Dex  uint
	Int  uint
	Luck uint
}

type Character struct {
	Name  string
	Level uint
	Job   Job
	Stat  Stat
}

func main() {
	S := []Character{}
	for i := 0; i < 10; i++ {
		S = append(S, Character{"zl존S2도적", uint(i + 1), Job{1, "도적"}, Stat{4, 25, 4, 120}})
	}

  	// range를 활용해 반복 호출 가능
	for _, k := range S {
		fmt.Println(k.Name, "( Lv.", k.Level, ") - [", k.Job.JobName, "]")
	}
}
```

```
// 결과 값
zl존S2도적 ( Lv. 1 ) - [ 도적 ]
zl존S2도적 ( Lv. 2 ) - [ 도적 ]
zl존S2도적 ( Lv. 3 ) - [ 도적 ]
zl존S2도적 ( Lv. 4 ) - [ 도적 ]
zl존S2도적 ( Lv. 5 ) - [ 도적 ]
zl존S2도적 ( Lv. 6 ) - [ 도적 ]
zl존S2도적 ( Lv. 7 ) - [ 도적 ]
zl존S2도적 ( Lv. 8 ) - [ 도적 ]
zl존S2도적 ( Lv. 9 ) - [ 도적 ]
zl존S2도적 ( Lv. 10 ) - [ 도적 ]
```
