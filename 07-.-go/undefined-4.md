# 고루틴 타임아웃

## 개요

가끔은 고루틴이 종료되는 시간이 예상보다 길어질 수 있는데, 이런 상황에서는 고루틴의 `타임아웃(timeout)`을 통해서 블록 상태를 풀 수 있습니다.&#x20;

고루틴의 타임아웃을 주기 위해서는 2가지 방법이 존재하는데, 이에 대해서 알아 보도록 하겠습니다.

## main() 내부에서 고루틴 타임아웃

### 예시

{% code title="timeOut1.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c1 := make(chan string)
	go func() {
		time.Sleep(3 * time.Second)
		c1 <- "c1 OK"
	}()

	select {
	case res := <-c1:
		fmt.Println(res)
	case <-time.After(time.Second):
		fmt.Println("timeout c1")
	}

	c2 := make(chan string)
	go func() {
		time.Sleep(3 * time.Second)
		c2 <- "c2 OK"
	}()

	select {
	case res := <-c2:
		fmt.Println(res)
	case <-time.After(4 * time.Second):
		fmt.Println("timeout c2")
	}
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>timeOut1.go의 실행 예시 화면</p></figcaption></figure>

위 예시와 같이 main() 내부에서 호출할 때에는 단순히 `time.After`를 활용해 구현할 수 있습니다.

## main() 바깥에서 고루틴 타임아웃

### 예시

{% code title="timeOut2.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"os"
	"strconv"
	"time"
)

var result = make(chan bool)

func timeout(t time.Duration) {
	temp := make(chan int)
	go func() {
		time.Sleep(5 * time.Second)
		defer close(temp)
	}()

	select {
	case <-temp:
		result <- false
	case <-time.After(t):
		result <- true
	}
}

func main() {
	arguments := os.Args
	if len(arguments) != 2 {
		fmt.Println("Please provide a time duration in milliseconds!")
		return
	}

	t, err := strconv.Atoi(arguments[1])
	if err != nil {
		fmt.Println(err)
		return
	}

	duration := time.Duration(int32(t)) * time.Millisecond
	fmt.Printf("Timeout period is %s\n", duration)

	go timeout(duration)

	val := <-result
	if val {
		fmt.Println("Time out!")
	} else {
		fmt.Println("OK")
	}
}
```
{% endcode %}

#### 타임아웃 주기를 100ms로 했을 때 결과 화면

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption><p>타임아웃이 100ms 인 경우 결과 화면</p></figcaption></figure>

#### 타임아웃 주기를 5500ms로 했을 때 결과 화면

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption><p>타임아웃이 5500ms 인 경우 결과 화면</p></figcaption></figure>

위 2가지 결과를 통해 볼 수 있듯이 `main()`함수 외부에서 채널을 구성하고 이를 `select`로 다룸으로써 타임아웃을 구현하는 방식에 대해서 알아보았습니다.
