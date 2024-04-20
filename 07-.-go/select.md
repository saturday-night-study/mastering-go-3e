# select 키워드

## 개요

`select` 키워드는 여러 채널을 동시에 기다릴 수 있게 만들어주는 키워드이기에 매우 중요합니다. `select` 블록은 `switch`문과 비슷하게 여러 `case`와 `기본 선택자(default)`와 같은 추가 요소를 사용하여 구현할 수 있습니다.

또한, 교착상태에 오랜 시간 빠지는 것을 방지하기 위한 타임아웃 옵션을 사용할 수 있습니다.&#x20;

### select 사용법

`select`는 `case`를 사용하지 않는다면 영원히 대기 상태로 만들 수 있습니다. 이를 사용해 `select`로 여러 통신 연산을 기다릴 수 있습니다. `select` 블록 하나로 여러 개의 채널을 다룰 수 있고 여러 채널에 대해서 `논블로킹(non-blocking)` 방식으로 수행할 수 있습니다.&#x20;

{% hint style="warning" %}
`select` 문은 **모든 채널을 동시에 확인**하기에 **순차 실행이 되지 않음 !!**
{% endhint %}

#### 예시&#x20;

{% code title="select.go" lineNumbers="true" fullWidth="false" %}
```go
package main

import (
	"fmt"
	"math/rand"
	"os"
	"strconv"
	"sync"
	"time"
)

func gen(min, max int, createNumber chan int, end chan bool) {
	time.Sleep(time.Second)
	for {
		select {
		case createNumber <- rand.Intn(max-min) + min:
		case <-end:
			fmt.Println("Ended!")
			// return (만약 return 시 여기서 종료 됨)
		case <-time.After(4 * time.Second):
			fmt.Println("time.After()!")
			return
		}
	}
}

func main() {
	rand.Seed(time.Now().Unix())
	createNumber := make(chan int)
	end := make(chan bool)

	if len(os.Args) != 2 {
		fmt.Println("Please give me an integer!")
		return
	}

	n, _ := strconv.Atoi(os.Args[1])
	fmt.Printf("Going to create %d random numbers.\n", n)

	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		gen(0, 2*n, createNumber, end)
		wg.Done()
	}()

	for i := 0; i < n; i++ {
		fmt.Printf("%d ", <-createNumber)
	}

	end <- true
	wg.Wait()
	fmt.Println("Exiting...")
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption><p>select.go 예시코드 실행 화면</p></figcaption></figure>

위 예시를 통해 가장 중요하게 기억에 남겨야 하는 부분은 **여러 case를 통해서 여러 개의 채널의 입력을 기다릴 수 있다는 점**입니다 :tada:
