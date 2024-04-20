# 경쟁 상태

## 개요

`데이터 경쟁 상태(data race condition)`는 스레드 또는 고루틴과 같은 요소가 여러 개 생성 된 프로그램 내부에서 **동일한 변수 또는 공유 리소스에 서로 접근하려고 경쟁하는 상태**를 의미합니다.

조금 더 상세하게 표현한다면, 데이터 경쟁은 두 개 이상의 `명령(Instruction)`이 동일한 메모리 주소에 접근하는 상황에서 `쓰기(Write)` 연산을 수행할 때 발생 됩니다. 또한 `읽기(Read)` 연산에서는  경쟁 상태가 발생하지 않습니다.

만약 **경쟁 상태가 일어난다면 이는 작성 된 프로그램이 특정 명령을 수행했을 때 동일한 결과를 낼 수 없음**을 의미하며, 이는 큰 오류를 발생 시킬 수 있는 부분입니다.

## Go 경쟁 상태 감지기

앞서 이야기한 것 처럼 경쟁 상태가 발생 되면 프로그램에 오류가 발생할 수 있기에 Go에서는 `경쟁 상태 감지기(race detector)`를 지원합니다. Go 소스 파일을 빌드하거나 실행할 때 `-race` 플래그를 지정하면 작동하며, 공유 메모리에 접근하는 모든 경우 `sync.Mutex`나 `sync.WaitGroup`에 대한 호출을 비롯한 모든 **동기화 이벤트를 기록하고 자동으로 보고서를 출력**해 발생할 수 있는 경쟁 상태를 사전에 확인할 수 있게 하여 문제를 사전 차단할 수 있게 도와줍니다. :clap:

#### 실습

실제로 `-race` 플래그를 활용해서 `channels.go` 파일을 실행한다면, 다음과 같은 결과를 확인할 수 있습니다.

{% code title="channels.go" lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"sync"
)

func writeToChannel(c chan int, x int) {
	c <- x
	close(c)
}

func printer(ch chan bool) {
	ch <- true
}

func main() {
	c := make(chan int, 1)

	var waitGroup sync.WaitGroup
	waitGroup.Add(1)
	go func(c chan int) {
		defer waitGroup.Done()
		writeToChannel(c, 10)
		fmt.Println("Exit.")
	}(c)

	fmt.Println("Read:", <-c)
	_, ok := <-c
	if ok {
		fmt.Println("Channel is open!")
	} else {
		fmt.Println("Channel is closed!")
	}

	waitGroup.Wait()

	var ch chan bool = make(chan bool)
	for i := 0; i < 5; i++ {
		go printer(ch)
	}

	n := 0
	for i := range ch {
		fmt.Println(i)
		if i == true {
			n++
		}
		if n > 2 {
			fmt.Println("n:", n)
			close(ch)			// channel 닫음
			break
		}
	}

	for i := 0; i < 5; i++ {
		fmt.Println(<-ch)
	}
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>-race 플래그와 함께 실행한 channels.go 예시 프로그램 출력 화면</p></figcaption></figure>



위 코드를 실행하면 분명 소스코드 상의 문제는 없어 보이나 실제 `channels.go` 54번 줄의 채널 닫음과 `channels.go` 14번째 줄의 채널 데이터 입력이 순서가 보장되지 않아 여러 번 실행했을 때 `send on closed channel` 과 같은 패닉 오류가 발생할 수 있습니다.

이를 방지하기 위해서는 반드시 채널을 사용할 때에는 경쟁 상태를 방지하는 것이 중요합니다. 다음 장인 [select.md](select.md "mention")에서 이를 방지하기 위한 방법을 알아보도록 하겠습니다.
