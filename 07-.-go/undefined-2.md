# 채널

채널이란 고루틴끼리 데이터를 주고 받을때 사용하는 통신 매커니즘입니다. 각 채널마다 '원소타입' 으로만 데이터를 교환하고. 채널로 보낼 송신자 수신자가 있어야합니다. 이때 '파이프라인' 처럼 고루틴 끼리 지속적으로 데이터를 주고 받는것도 가능합니다.

### 채널에 데이터 읽고 쓰기

아래 예제는 고루틴에서 채널에 값을 쓰고, 닫은 후 메인 고루틴에서 채널의 상태를 확인하는 예제입니다.

<소스>

```go
func main() { 
	// 버퍼 채널 (크기 1)
    c := make(chan int)  
  
    var wg sync.WaitGroup  
    wg.Add(1)  
  
    go func(c chan int) {  
       defer wg.Done()  
  
       writeToCh(c, 10)  
       fmt.Println("Exiting goroutine")  
    }(c)  
    
    // 값을 가져올때 까지 대기, 위의 고루틴에서 값을 입력하지 않았다면 데드락이 된다.
    fmt.Println("Read:", <-c)
  
    // 채널이 닫혔는지 확인  
    value, ok := <-c  
    if ok {  
       fmt.Println("Channel is open")  
    } else {  
       fmt.Println("Channel is closed")  
    }
  
    wg.Wait()  
  
    fmt.Printf("ch value is %d\n", value)  
  
}  
  
func writeToCh(c chan int, x int) {  
    c <- x  
    close(c)  
}
```

<출력>

```
Exiting goroutine
Read: 10
Channel is closed
ch value is 0
```

아래 예제는 고루틴이 모두 받아오지 않았을때 어떻게 되는지 확인하는 예제입니다. 총 5개의 채널이 사용되었지만, 강제로 채널을 닫게되면 프로그램은 종료되지만 채널의 모든값을 받아오지 않아 패닉 상태에 빠지게 됩니다.

<소스>

```go
func main() {  
    var ch = make(chan bool)  
  
    for i := 0; i < 5; i++ {  
       go printer(ch)  
    }  
  
    n := 0  
  
    for i := range ch {  
       println(i)  
  
       if i == true {  
          n++  
       }  
  
       if n > 2 {  
          fmt.Println("n:", n)  
          close(ch)  
          break  
       }  
    }  
  
    for i := 0; i < 5; i++ {  
       fmt.Println(<-ch)  
    }  
}  
  
func printer(ch chan bool) {  
    ch <- true  
}
```

<출력>

```
true
true
true
n: 3
false
false
false
false
false
panic: send on closed channel
```

### 닫힌 채널에서 데이터 받기

닫힌 채널에서 데이터를 읽으면 데이터 타입의 제로 값을 반환하게 되는데요, 이때 닫힌 채널에 데이터를 쓰려고 하면 충돌이 일어납니다.

<소>

```go
func main() {  
    willClose := make(chan complex64, 10)  
  
    // 채널에 2개의 값을 발송후 닫음  
    willClose <- -1  
    willClose <- 1i  
    close(willClose)  
  
    // 채널을 닫았기 때문에 더 이상 값을 발송 / 송신이 불가능 
    read := <-willClose  
    fmt.Println(read)  
}
```

<출력>

```
(-1+0i) - 데이터 타입의 제로값...
```

결론적으로 닫힌 채널에서는 값을 줄 수도 받을수도 없기 때문에 타이밍에 주의해야한다는것을 알게 되었습니다.

### 함수 매개변수로 지정한 채널

채널의 방향을 통해 함수에서 읽기, 쓰기 용도를 지정할 수 있습니다. f2 함수를 통해 읽기 쓰기 방향을 기억하면 좋을것 같네요.

함수를 통해 채널의 방향을 설정하면, 읽기 전용. 쓰기 전용으로 기능들을 사용할수 있고. 이는 프로그램의 방향성을 만드는데 도움이 됩니다.

```go
func main() {  
    c := make(chan int)  
    go writeToChannel(c, 10)  
    time.Sleep(1 * time.Second)  
    fmt.Println("Read:", <-c)  
    time.Sleep(1 * time.Second)  
    close(c)  
  
    c1 := make(chan int, 1)  
    c2 := make(chan int, 1)  
  
    // Write to channel  
    c1 <- 5  
    f2(c1, c2)  
    // Read from channel  
    fmt.Println("Read (main):", <-c2)  
}  
  
// 쓰기 전용  
func printer(ch chan<- bool) {  
    ch <- true  
}  
  
// 읽기 전용  
func writeToChannel(c chan<- int, x int) {  
    fmt.Println("1", x)  
    c <- x  
    fmt.Println("2", x)  
}  
  
// OUT 읽기 전용, IN 쓰기 전용  
func f2(out <-chan int, in chan<- int) {  
    x := <-out  
    fmt.Println("Read (f2):", x)  
    in <- x  
    return  
}
```

```
1 10
Read: 10
2 10
Read (f2): 5
Read (main): 5
```
