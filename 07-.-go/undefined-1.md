# 고루틴

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

고루틴이란?

Go 런타임에 의해 관리되는 경량 스레드입니다.

### 고루틴 생성

고루틴을 정의하고 실행하는 방법은 go 키워드 뒤에 '일반 함수', '익명 함수' 를 실행하면 되는데요, 이때 그 함수는 즉시 반환되고 실제 동작은 실행한 고루틴이 아닌 다른 고루틴을 통해 진행하게 됩니다.

이때 메인 고루틴이 종료되면, 다른 고루틴은 실행이 되지 않고 종료 되므로 아래 두개 예제는 time.Sleep 을 이용하여 지연해 주었습니다.

#### 익명 함수의 고루틴 생성

<소스>

```
func main() {  
    go func(x int) {  
       fmt.Printf("x: %d\n", x)  
    }(10)  
  
    time.Sleep(1 * time.Second)  
}
```

<출력>

```
x: 10
```

#### 함수의 고루틴 생성

<소스>

```
func main() {  
    go printNumber(10)  
    time.Sleep(1 * time.Second)  
}  
  
func printNumber(x int) {  
    fmt.Printf("x: %d\n", x)  
}
```

<출력>

```
x: 10
```

### 다수의 고루틴 생성

다음으로 for 루프를 통해 다수의 고루틴을 생성해 보겠습니다.

<소스>

```
func main() {  
    for i := 0; i < 20; i++ {  
       go func(x int) {  
          fmt.Printf("%d ", x)  
       }(i)  
    }  
  
    time.Sleep(1 * time.Second)  
}
```

<출력>

```
5 1 7 6 8 9 0 3 4 10 2 11 12 13 14 19 15 16 17 18
```

위를 통해 고루틴이 생성되는 순서와 실행되는 순서가 일정하지 않고, 스케줄러가 서로 다른 CPU 자원을 활용했기 때문에 실행 순서가 일관되지 않음을 확인할 수 있습니다.

고루틴의 스케줄링 전략을 잘 보여주는 예시가 되겠네요.

### 고루틴이 끝날때 까지 기다리기

생성한 고루틴의 작업이 언제쯤 끝날지 시간으로 알 수는 없습니다. 당연히 go 런타임은 대기 방법을 가지고 있습니다.

<소스>

```
func main() {  
    var wg sync.WaitGroup  
  
    count := 10  
    wg.Add(count)  
  
    for i := 0; i < count; i++ {  
       go func(x int) {  
          defer wg.Done()  
          fmt.Printf("%d ", x)  
       }(i)  
    }  
  
    wg.Wait()  
}
```

<출력>

```
1 9 3 0 4 6 7 5 8 2
```

생성한 고루틴 만큼을 wg.Add 함수를 통해 구조체에 전달하고, wg.Wait 함수를 통해 메인 고루틴에서 대기할 수 있도록 소스코드를 작성하였습니다.

실행된 고 루틴에서는 작업이 종료되면 wg.Done() 을 통해 호출하고요.

닷넷의 async, await 에 비하면 좀 카운터로 제어한다는 것이 좀 문제가 발생할것 같이 보이시지 않으시나요? 실제로 문제가 발생합니다.

<소스>

```
func main() {  
    var wg sync.WaitGroup  
  
    count := 10  
    wg.Add(count)  
  
    for i := 0; i < count; i++ {  
       go func(x int) {  
          //defer wg.Done()  
          fmt.Printf("%d ", x)  
       }(i)  
    }  
  
    wg.Wait()  
}
```

<츨력>

```
9 1 5 3 4 7 6 8 0 2 fatal error: all goroutines are asleep - deadlock!

goroutine 1 [semacquire]:
sync.runtime_Semacquire(0xc000008120?)
        C:/Users/prtra/sdk/go1.22.1/src/runtime/sema.go:62 +0x25
sync.(*WaitGroup).Wait(0xc000066058?)
        C:/Users/prtra/sdk/go1.22.1/src/sync/waitgroup.go:116 +0x48
main.main()
        C:/Users/prtra/OneDrive/문서/GitHub/mastering-go-example/2024-04-27 goroutine/cmd/wait/main.go:21 +0x7d

```

<소스>

```
func main() {  
    var wg sync.WaitGroup  
  
    count := 10  
    wg.Add(count)  
  
    for i := 0; i < count; i++ {  
       go func(x int) {  
          if x != 0 {  
             defer wg.Done()  
          }  
  
          fmt.Printf("%d ", x)  
       }(i)  
    }  
  
    wg.Wait()  
}
```

<출력>

```
9 1 5 3 4 7 6 8 2 0 fatal error: all goroutines are asleep - deadlock!

goroutine 1 [semacquire]:
sync.runtime_Semacquire(0xc000008120?)
        C:/Users/prtra/sdk/go1.22.1/src/runtime/sema.go:62 +0x25
sync.(*WaitGroup).Wait(0x0?)
        C:/Users/prtra/sdk/go1.22.1/src/sync/waitgroup.go:116 +0x48
main.main()
        C:/Users/prtra/OneDrive/문서/GitHub/mastering-go-example/2024-04-27 goroutine/cmd/wait/main.go:24 +0xe5
```

wg에 입력되었던 값과, 고루틴 종료가 일치하지 않을 때, 데드락임을 패닉을 일으키게 됩니다.

<소스>

```
func main() {  
    var wg sync.WaitGroup  
  
    count := 10  
    wg.Add(count)  
  
    for i := 0; i < count; i++ {  
       go func(x int) {  
          if x != 0 {  
             defer wg.Done()  
          }  
  
          fmt.Printf("%d ", x)  
       }(i)  
    }  
  
    wg.Add(1)  
    go func() {  
       time.Sleep(1 * time.Hour)  
    }()  
  
    wg.Wait()  
}
```

<출력>

```
2 1 6 3 7 4 8 5 9 0
... 무한정 대기
```

또한 이런 케이스와 같은 경우에는 인지하지 못하고 무한정 대기 할 수도 있죠. 불완전한 대기방법으로 보이네요.



