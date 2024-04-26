# 공유 메모리와 공유 변수

## 개요

`공유 메모리(shared memory)`와 `공유 변수(shared variable)`는 동시성 프로그래밍의 가장 큰 주제이며 유닉스 스레드 간의 통신을 위해 사용되는 가장 흔한 방법 중 하나입니다. 같은 원리가 Go와 고루틴에도 적용되며 해당 절에서 이에 대해 알아보도록 하겠습니다. :face\_with\_monocle:

`뮤텍스(mutex)` 변수는 `상호 배제(mutual exclusion)` 변수의 줄임말로 스레드를 동기화하거나 여러 스레드가 동시에 쓸 수 없는 공유 데이터를 보호하기 위한 목적으로 주로 사용됩니다. 뮤텍스는 마치 크기가 1인 버퍼 채널처럼 동작하며, 하나의 고루틴만이 공유 변수에 접근할 수 있습니다.

Go 언어에서는 `sync.Mutex`와 `sync.RWMutex` 데이터 타입을 통해 지원하고 있습니다.

### 임계 영역(critical section)

임계 영역(critical section)이란 동시성 프로그램 코드 중에서 여러 프로세스, 스레드, 고루틴 등에서 동시에 실행할 수 없는 영역을 의미합니다. 해당 영역은 뮤텍스를 통해 반드시 보호해야합니다.

## sync.Mutex 타입

sync.Mutex는 Go에서 뮤텍스를 구현한 구현체입니다. sync.Mutex 타입의 정의는 sync 패키지 내부의 [mutex.go ](https://github.com/golang/go/blob/master/src/sync/mutex.go)코드에서 찾을 수 있습니다.

```go
type Mutex struct {
	state int32
	sema  uint32
}
```

#### sync.Mutex 사용예시&#x20;

{% code title="mutex.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"os"
	"strconv"
	"sync"
	"time"
)

var m sync.Mutex
var v1 int

func change(i int) {
	m.Lock()
	time.Sleep(time.Second)
	v1 = v1 + 1
	if v1 == 10 {
		v1 = 0
		fmt.Print("* ")
	}
	m.Unlock()
}

func read() int {
	m.Lock()
	a := v1
	m.Unlock()
	return a
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Please give me an integer!")
		return
	}

	numGR, err := strconv.Atoi(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}
	var wg sync.WaitGroup

	fmt.Printf("%d ", read())
	for i := 0; i < numGR; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			change(i)
			fmt.Printf("-> %d", read())
		}(i)
	}

	wg.Wait()
	fmt.Printf("-> %d\n", read())
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>mutex.go 실행 결과</p></figcaption></figure>

### 뮤텍스 해제를 잊어버릴 경우

만약 뮤텍스를 사용하고 해제 않을 경우에는 매우 간단한 프로그램일지라도 패닉이 발생할 수 있습니다. 이는 sync.RWMutex 도 동일합니다.(이는 다음 sync.RWMutex 부분에서 확인)

#### 뮤텍스 해제를 잊은 경우 예시

{% code title="forgetMutex.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"sync"
)

var m sync.Mutex
var w sync.WaitGroup

func function() {
	m.Lock()
	fmt.Println("Locked!")
}

func main() {
	w.Add(1)
	go func() {
		defer w.Done()
		function()
	}()

	w.Add(1)
	go func() {
		defer w.Done()
		function()
	}()

	w.Wait()
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>forgetMutex.go 실행 결과 화면</p></figcaption></figure>

위 코드의 경우 Lock 이후 해제를 해주지 않아 다음 고루틴에서 다시 뮤텍스를 Lock 하려고 하여 발생한 오류입니다. 만약 Lock이 되어 있는 뮤텍스를 다시 Lock하게 되면 교착 상태에 빠지게 됩니다.

## sync.RWMutex 타입

`sync.RWMutex`는 `sync.Mutex`의 발전 된 버전으로 Go 표준 라이브러리 `sync` 패키지 디렉터리의 [rwmutex.go](https://github.com/golang/go/blob/master/src/sync/rwmutex.go) 파일에 정의 되어 있습니다.

```go
type RWMutex struct {
	w           Mutex  
	writerSem   uint32
	readerSem   uint32
	readerCount atomic.Int32
	readerWait  atomic.Int32
}
```

차이를 비교해 보면 `sync.RWMutex`는 `sync.Mutex`에서 몇 가지 사항을 추가하고 개선한 것입니다.

바로 **읽기 잠금과 쓰기 잠금을 별도로 분리**하여 읽기 연산을 수행하는 함수를 여러 개 동작할 수 있게 하여 수행 속도를 높일 수 있게 개선하였습니다 !&#x20;

#### sync.RWMutex 사용 예시

{% code title="rwMutex.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var Password *secret
var wg sync.WaitGroup

type secret struct {
	RWM      sync.RWMutex
	password string
}

func Change(pass string) {
	fmt.Println("Change() function")
	Password.RWM.Lock()
	fmt.Println("Change() Locked")
	time.Sleep(4 * time.Second)
	Password.password = pass
	Password.RWM.Unlock()
	fmt.Println("Change() UnLocked")
}

func show() {
	defer wg.Done()
	Password.RWM.RLock()
	fmt.Println("Show function locked!")
	time.Sleep(2 * time.Second)
	fmt.Println("Pass value:", Password.password)
	defer Password.RWM.RUnlock()
}

func main() {
	Password = &secret{password: "myPass"}
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go show()
	}

	wg.Add(1)
	go func() {
		defer wg.Done()
		Change("123456")
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		Change("54321")
	}()

	wg.Wait()

	// Direct access to Password.password
	fmt.Println("Current password value:", Password.password)
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>rwMutex.go 프로그램 결과 화면</p></figcaption></figure>
