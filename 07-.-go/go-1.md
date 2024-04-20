# Go 채널 다시보기

## 개요

지금 까지는 채널의 기본적인 사용 방법에 대해서 알아보았습니다. 이번 절에서는 `버퍼 채널(Buffered Channel)`,  `닐 채널(Nil Channel)`, `시그널 채널(Signal Channel)` 총 3가지에 대해서 심화 학습을 진행해보도록 하겠습니다.

### 참고 사항

채널 타입에서 `제로 값(zero value)`로 사용되는 값은 `nil`입니다. 또한 **닫힌 채널에 메시지를 보내면 패닉이 발생**하는데, **닫힌 채널에서 값을 읽을 때에는 해당 채널에서 지정한 타입의 제로 값을 받게 됩니다**.

따라서 채널을 닫은 뒤에 쓰기 연산은 수행할 수 없지만 읽기 연산은 수행이 가능합니다. :open\_mouth:

## 버퍼 채널(Buffered Channel)

버퍼  채널은 버퍼를 사용하는 채널로 버퍼 채널을 통해 더 많은 요청을 처리할 수 있도록 작업을 큐에 넣을 수 있습니다. 또한 어플리케이션의 처리량을 제한하고자 할 때 버퍼 채널을 `세마포어(Semaphore)`와 같은 형태로 사용할 수 있습니다.

### chan.go 파일 분석

{% embed url="https://go.dev/src/runtime/chan.go" %}
go runtime package 중 chan.go 파일
{% endembed %}

위 코드를 확인해보면, 실제 채널은 [원형 큐(circular queue)](https://velog.io/@mcc919/Data-Structure-%EC%9B%90%ED%98%95-%ED%81%90Circular-Queue-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)를 사용하여 구현했음을 확인할 수 있습니다. 큐가 있기 때문에 버퍼의 크기에 따라서 블록을 할 수 있고 이를 `select`로 활용해 다양한 방식의 프로그램을 구현할 수 있습니다.

### 버퍼 채널의 예시

{% code title="bufChannel.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
)

func main() {
	numbers := make(chan int, 5) // 최대 5개 크기의 int 채널
	counter := 10

	for i := 0; i < counter; i++ {
		select {
		case numbers <- i * i:
			fmt.Println("About to process", i)
		default:
			fmt.Print("No space for ", i, " ")
		}
	}
	fmt.Println()

	for {
		select {
		case num := <-numbers:
			fmt.Print("*", num, " ")
		default:
			fmt.Println("Nothing left to read!")
			return
		}
	}
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>bufChannel.go의 예시 결과 화면</p></figcaption></figure>

여기서 버퍼 채널이 실제로 동작하는 방식이 어떤 방식인지 조금 더 자세히 알고 싶어 `go runtime` 패키지 내부의 `chan.go`를 확인해 보았습니다.

### 워커 풀 (Worker Pool)

워커 풀(worker pool)은 할당 된 작업을 처리하려는 스레드의 집합입니다. 다양한 웹 서버와 Go 기본 패키지인 net/http도 워커 풀을 사용합니다.

메인 프로세스는 들어온 모든 요청을 모두 워커 프로세스로 전달하여 처리를 진행하며, 워커 프로세스가 작업을 모두 마치면 새로운 클라이언트를 받아들일 준비를 합니다. (Go의 경우 고루틴을 활용)

하지만 일반 적인 프로그램에서 스레드를 한번 생성한 뒤 삭제 후 재 생성하면 비용이 많이 들기 때문에 이를 생성 한 뒤 재활용하는 방식으로 사용하고 있습니다. 아래 예시는 이를 버퍼 채널을 통해 구현 된 워커 풀 예시입니다. 이렇게 제작하면 동시에 실행되는 고루틴의 수를 제한할 수 있습니다.

#### 워커 풀 예시

{% code title="" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"strconv"
	"sync"
	"time"
)

type Client struct {
	id      int
	integer int
}

type Result struct {
	job    Client
	square int
}

var size = runtime.GOMAXPROCS(0)
var clients = make(chan Client, size)
var data = make(chan Result, size)

func worker(wg *sync.WaitGroup) {
	for c := range clients {
		square := c.integer * c.integer
		output := Result{c, square}
		data <- output
		time.Sleep(time.Second)
	}
	wg.Done()
}

func create(n int) {
	for i := 0; i < n; i++ {
		c := Client{i, i}
		clients <- c
	}
	close(clients)
}

func main() {
	if len(os.Args) != 3 {
		fmt.Println("Need #jobs and #workers!")
		return
	}

	nJobs, err := strconv.Atoi(os.Args[1])
	if err != nil {
		fmt.Println(err)
		return
	}

	nWorkers, err := strconv.Atoi(os.Args[2])
	if err != nil {
		fmt.Println(err)
		return
	}

	go create(nJobs)

	finished := make(chan interface{})
	go func() {
		for d := range data {
			fmt.Printf("Client ID: %d\tint: ", d.job.id)
			fmt.Printf("%d\tsquare: %d\n", d.job.integer, d.square)
		}
		finished <- true
	}()

	var wg sync.WaitGroup
	for i := 0; i < nWorkers; i++ {
		wg.Add(1)
		go worker(&wg)
	}
	wg.Wait()
	close(data)

	fmt.Printf("Finished: %v\n", <-finished)
}
```
{% endcode %}

#### 예시 결과 화면

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>wPools.go 예시 실행 화면</p></figcaption></figure>

## 닐 채널(Nil Channel)

`nil` 채널은 항상 블록됩니다. 다시 말해 `nil 채널`은 읽거나 쓰면 블록이 됩니다. 채널의 이러한 속성은 `select` 구문에서 채널 변수에 `nil` 값을 할당하여 해당 `case`를 선택하지 못하게 할 수 있습니다.&#x20;

또한 <mark style="color:red;">**`nil 채널`**</mark><mark style="color:red;">**을 닫으려고 하면 프로그램에서  패닉이 발생**</mark>하게 됩니다. 그렇기에 **`nil 채널`은 의도적으로 블록이 되고 싶은 경우에만 사용 되어져야 합니다**.

### nil 채널 닫기 예시

```go
package main

func main() {
	var c chan string
	close(c)
}
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>nil 채널을 닫았을 때 오류가 발생한 모습</p></figcaption></figure>

### nil 채널의 사용예시

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var wg sync.WaitGroup

func add(c chan int) {
	sum := 0
	t := time.NewTimer(time.Second)

	for {
		select {
		case input := <-c:
			sum = sum + input
		case <-t.C:
			c = nil
			fmt.Println(sum)
			wg.Done()
		}
	}
}

func send(c chan int) {
	for {
		c <- rand.Intn(10)
	}
}

func main() {
	c := make(chan int)
	rand.Seed(time.Now().Unix())

	wg.Add(1)
	go add(c)
	go send(c)
	wg.Wait()
}
```

위 `nil 채널` 예시를 실행하면 첫 번째 브랜치가 실행 되는 횟수를 지정하지 않았기 때문에 결과 값은 실행 시 마다 달라집니다.

## 시그널 채널(Signal Channel)

`시그널 채널(Signal Channel)`은 신호를 보내는 용도로만 사용되는 채널입니다. 쉽게 말해 다른 고루틴에게 신호를 알려주기 위해 사용되는 채널입니다. **시그널 채널은 데이터를 전송할 수 없으며**, 시그널 채널을 활용하면 **고루틴의 실행 순서를 지정**할 수 있습니다.

### 고루틴의 실행 순서 지정

해당 절에서는 시그널 채널을 이용해 고루틴의 실행 순서를 지정하는 방법에 대해서 알아보도록 하겠습니다. 이 기법을 사용할 때 주의할 사항으로는 반드시 작은 수의 고루틴을 다룰 때에만 사용해야 한다는 점입니다.&#x20;

그 이유는 여러개의 고루틴을 사용하게 되면 **실행 순서의 복잡도가 증가하게 되고 이로 인해 교착 상태에 빠져버릴 수 있기 때문**입니다.

#### 고루틴 실행 순서 지정 예시

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func A(a, b chan struct{}) {
	<-a
	fmt.Println("A()!")
	time.Sleep(time.Second)
	close(b)
}

func B(a, b chan struct{}) {
	<-a
	fmt.Println("B()!")
	time.Sleep(3 * time.Second)
	close(b)
}

func C(a, b chan struct{}) {
	<-a
	fmt.Println("C()!")
	close(b)
}

func D(a chan struct{}) {
	<-a
	fmt.Println("D()!")
	wg.Done()
}

func main() {
	x := make(chan struct{})
	y := make(chan struct{})
	z := make(chan struct{})
	w := make(chan struct{})

	wg.Add(1)
	go func() {
		D(w)
	}()

	wg.Add(1)
	go func() {
		D(w)
	}()

	go A(x, y)

	wg.Add(1)
	go func() {
		D(w)
	}()

	go C(z, w)
	go B(y, z)

	wg.Add(1)
	go func() {
		D(w)
	}()

	// This triggers the process
	close(x)
	wg.Wait()
}
```

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>defineOrder.go 예시 결과 화면</p></figcaption></figure>

결과 화면에서 볼 수 있듯이 `A()`, `B()`, `C()`, `D()`가 각각 실행 되고 이후 `D()`가 호출 된 횟수만큼 실행 됨을 확인할 수 있었습니다.&#x20;
