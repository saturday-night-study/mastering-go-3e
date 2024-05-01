# context 패키지

### context란?

context 는 여러 언어에서 중요하게 다루는 동시성을 지원하기 위한 개념 중 하나입니다. 보통 프로그램이 실행될 때의 환경이나 상황에 대한 정보를 담게 되는데요. 각 언어들에서 주로 사용하는 context 들을 살펴보면서 비교한 후 go에서 의미하는 context란 무엇인지 알아보겠습니다.

* JAVA
  * 스레드 컨택스트 : 스레드 스택, 실행상태, 우선순위
  * 의존성 주입 컨택스트 : 어플리케이션 정보
  * 서블릿 컨택스트 : 어플리케이션 전역 정보
* C#
  * DB Context : 데이터 베이스의 정보
  * Http Context : 현재 Http 의 정보
  * 스레드 컨택스트 : 스레드 관련 정보

위의 대표 언어들을 보면 스레드란 '현재 실행중인 범위'를 나타내는 경우로 쓰입니다. 그렇다면 동시성이 필요할 때는 '동시에 실행중인 다수 중 하나의 범위에서 환경이나 상황에 대한 정보'를 제공하는 역할을 하고 있다는것을 알 수 있습니다.

### go의 context

그렇다면 Go에서 Context란 무엇일까요? 단순하게 보자면 아래의 인터페이스 입니다.

```go
type Context interface {  
    Deadline() (deadline time.Time, ok bool)  
    Done() <-chan struct{}  
    Err() error  
    Value(key any) any  
}
```

1. Deadline() : 작업을 수행한 시간을 반환하고, 기한이 없는지 여부를 판단합니다.
2. Done() : 종료 신호를 줄 수 있는 채널을 반환합니다.
3. Err() : 컨텍스트가 종료된 이유 등을 반환할 수 있습니다.
4. Value() : 컨텍스트에 저장된 값을 반환합니다.

위의 인터페이스의 메서드를 보면 다른 언어들과 같이 동시성을 지원하기위해 컨텍스트는 API / 고루틴 간에 취소 신호 및 기타 요청 값을 전달하는 맥락의 객체를 전달합니다. 일반적으로 아래와 같이 생성하고 사용합니다.

<생성 방법>

1. context.Background() : 빈 Context. 요청에 대한 최상위 컨택스트. 취소되지 않습니다.
2. context.TODO() : 어떤일을 시작할지 예상이 되지 않을때.

<자주 사용하는 함수>

1. context.WithValue() : 값을 전달하는 용도로 사용할 때
2. context.WithCancel() : 취소 신호를 보낼수 있는 채널을 만듭니다.
3. context.WithTimeout() : 타임아웃이 있는 채널을 포함하여 만듭니다.

이를 이해하기위해서 가장 자주 사용하는 타임아웃을 보면

```go

// 메인 함수
func main() {  
	ctx,cancel:= context.WithTimeout(context.Background(), time.Second*10)  
	defer cancel()  
	fmt.Println(ctx)
}

// 부모 컨택스트 객체
type emptyCtx struct{}  
  
func (emptyCtx) Deadline() (deadline time.Time, ok bool) {  
    return  
}  
  
func (emptyCtx) Done() <-chan struct{} {  
    return nil  
}  
  
func (emptyCtx) Err() error {  
    return nil  
}  
  
func (emptyCtx) Value(key any) any {  
    return nil  
}  
  
type backgroundCtx struct{ emptyCtx }  
  
func (backgroundCtx) String() string {  
    return "context.Background"  
}

// 타임아웃 구현현
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {  
    return WithDeadline(parent, time.Now().Add(timeout))  
}

func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {  
    return WithDeadlineCause(parent, d, nil)  
}

func WithDeadlineCause(parent Context, d time.Time, cause error) (Context, CancelFunc) {  
    if parent == nil {  
       panic("cannot create context from nil parent")  
    }  
    if cur, ok := parent.Deadline(); ok && cur.Before(d) {  
       // The current deadline is already sooner than the new one.  
       return WithCancel(parent)  
    }  
    c := &timerCtx{  
       deadline: d,  
    }  
    c.cancelCtx.propagateCancel(parent, c)  
    dur := time.Until(d)  
    if dur <= 0 {  
       c.cancel(true, DeadlineExceeded, cause) // deadline has already passed  
       return c, func() { c.cancel(false, Canceled, nil) }  
    }  
    c.mu.Lock()  
    defer c.mu.Unlock()  
    if c.err == nil {  
       c.timer = time.AfterFunc(dur, func() {  
          c.cancel(true, DeadlineExceeded, cause)  
       })  
    }  
    return c, func() { c.cancel(true, Canceled, nil) }  
}

// 타이머 컨텍스트 
type timerCtx struct {  
    cancelCtx  
    timer *time.Timer // Under cancelCtx.mu.  
  
    deadline time.Time  
}  
  
func (c *timerCtx) Deadline() (deadline time.Time, ok bool) {  
    return c.deadline, true  
}  
  
func (c *timerCtx) String() string {  
    return contextName(c.cancelCtx.Context) + ".WithDeadline(" +  
       c.deadline.String() + " [" +  
       time.Until(c.deadline).String() + "])"  
}  
  
func (c *timerCtx) cancel(removeFromParent bool, err, cause error) {  
    c.cancelCtx.cancel(false, err, cause)  
    if removeFromParent {  
       // Remove this timerCtx from its parent cancelCtx's children.  
       removeChild(c.cancelCtx.Context, c)  
    }  
    c.mu.Lock()  
    if c.timer != nil {  
       c.timer.Stop()  
       c.timer = nil  
    }  
    c.mu.Unlock()  
}

```

타임아웃이 될때 취소할 수 있도록 구현하고 있네요, 락킹으로 낭비되는 자원을 막기 위해서는 타임아웃이 되지 않았다면 적절한 시점에 cancel을 반드시 호출 해 주어야겠죠?

참고 자료

* https://pkg.go.dev/context
* https://medium.com/@sdl182975/golang%EC%97%90%EC%84%9C%EC%9D%98-context-81c25630f673
* https://learn.microsoft.com/ko-kr/dotnet/api/system.threading.executioncontext?view=netframework-2.0

### context 패키지의 활용

#### 타임아웃

**context.WithCancel()**

<소스>

```go
func main() {  
    f1(1)  
    f1(2)  
    f1(3)  
    f1(4)  
    f1(5)  
}  
  
func f1(t int) {  
    c1 := context.Background()  
    c1, cancel := context.WithCancel(c1)  
    defer cancel()  
  
    go func() {  
       time.Sleep(4 * time.Second)  
       cancel()  
    }()  
  
    select {  
    case <-c1.Done():  
       fmt.Println("f1(): Done", c1.Err())  
       return  
    case r := <-time.After(time.Duration(t) * time.Second):  
       fmt.Println("f1():", r)  
    }  
    return  
}
```

<출력>

```
f1(): 2024-05-01 20:13:24.6081146 +0900 KST m=+1.010791901
f1(): 2024-05-01 20:13:26.6219237 +0900 KST m=+3.024601001
f1(): 2024-05-01 20:13:29.634386 +0900 KST m=+6.037063301
f1(): 2024-05-01 20:13:33.6485651 +0900 KST m=+10.051242401
f1(): Done context canceled
```

* context.Done() 은 cancel 함수가 동작하였을 때 select 됩니다. (즉 t값이 5부터는 확실하게 동작 하겠죠)

**context.WithTimeout()**

<소스>

```go

func main() {  
    f2(1)  
    f2(2)  
    f2(3)  
    f2(4)  
    f2(5)  
}  
  
func f2(t int) {  
    c2 := context.Background()  
    c2, cancel := context.WithTimeout(c2, time.Duration(t)*time.Second)  
    defer cancel()  
  
    go func() {  
       time.Sleep(4 * time.Second)  
       cancel()  
    }()  
  
    select {  
    case <-c2.Done():  
       fmt.Println("f2(): Done", c2.Err())  
       return  
    case r := <-time.After(time.Duration(t) * time.Second):  
       fmt.Println("f2():", r)  
    }  
  
    return  
}
```

<출력>

```
f2(): 2024-05-01 20:28:55.0499741 +0900 KST m=+1.001358301
f2(): 2024-05-01 20:28:57.0719454 +0900 KST m=+3.023329601
f2(): 2024-05-01 20:29:00.0857554 +0900 KST m=+6.037139601
f2(): 2024-05-01 20:29:04.0891249 +0900 KST m=+10.040509101
f2(): Done context canceled
```

**context.WithDeadline()**

<소스>

```go
func main() {  
    f3(1)  
    f3(2)  
    f3(3)  
    f3(4)  
    f3(5)  
}  
  
func f3(t int) {  
    c3 := context.Background()  
    deadline := time.Now().Add(time.Duration(2*t) * time.Second)  
    c3, cancel := context.WithDeadline(c3, deadline)  
    defer cancel()  
  
    go func() {  
       time.Sleep(4 * time.Second)  
       cancel()  
    }()  
  
    select {  
    case <-c3.Done():  
       fmt.Println("f3(): Done", c3.Err())  
       return  
    case r := <-time.After(time.Duration(t) * time.Second):  
       fmt.Println("f3():", r)  
    }  
  
    return  
}
```

<결과>

```
f3(): 2024-05-01 20:40:07.4685589 +0900 KST m=+1.005514901
f3(): 2024-05-01 20:40:09.4969263 +0900 KST m=+3.033882301
f3(): 2024-05-01 20:40:12.501628 +0900 KST m=+6.038584001
f3(): 2024-05-01 20:40:16.5053476 +0900 KST m=+10.042303601
f3(): Done context canceled
```

#### 키/값 저장소로 활용하기

```go

type aKey string  
  
func searchKey(ctx context.Context, k aKey) {  
    v := ctx.Value(k)  
    if v != nil {  
       fmt.Println("found value:", v)  
    } else {  
       fmt.Println("k not found:", k)  
    }  
}  
  
func main() {  
    myKey := aKey("mySecretValue")  
    ctx := context.WithValue(context.Background(), myKey, "myValue")  
    searchKey(ctx, myKey)  
    searchKey(ctx, aKey("notMyKey"))  
  
    emptyCtx := context.TODO()  
    searchKey(emptyCtx, myKey)  
}
```

```
found value: myValue
k not found: notMyKey
k not found: mySecretValue
```
