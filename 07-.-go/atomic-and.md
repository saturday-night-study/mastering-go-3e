# atomic 패키지



go 에서는 다양한 방법의 락킹을 통해 자원의 동기화를 지원하지만, 경합이 발생하여 시스템에 문제를 발생시키는 경우가 종종 발생합니다. 이를 해결하는 방법이 원자적 연산입니다.

{% hint style="info" %}
**원자적 행위**(atomic action)의 기본적인 의미는 더 이상 나누어질 수 없는 하나의 행위입니다. 컴퓨터 과학에서는 수행 도중 중단될 수 없는 하나의 동작 단위로 표현합니다. (위키 참조)
{% endhint %}

go lang 에서는 atomic 패키지(https://pkg.go.dev/sync/atomic)에서 원자적 연산을 지원하는데요. 패키지 개요 를 살펴보면 아래와 같습니다.

> Package atomic provides low-level atomic memory primitives useful for implementing synchronization algorithms.

이 패키지는 가장 낮은 수준의 메모리 동기화 알고리즘을 제공하기 위해서 '하드웨어 레벨' 에서 제공하는 연산을 실행하는데요. x86 CPU를 예시로 들어보자면. Add연산의 경우 lock add1 에 해당하는 명령어를 사용하도록 합니다.

아래 예제를 통해 실제로 위의 설명과 같이 동작하는지 확인해 보겠습니다.

소스코드

```
package main  
  
import "sync/atomic"  
  
func main() {  
    var x int64 = 1  
    atomic.AddInt64(&x, 1)  
}
```

빌드

```
go build -gcflags -S atomic.go
```

결과

```

main.main STEXT size=56 args=0x0 locals=0x18 funcid=0x0 align=0x0
        0x0000 00000 (atomic.go:5)       TEXT    main.main(SB), ABIInternal, $24-0
        0x0000 00000 (atomic.go:5)       CMPQ    SP, 16(R14)
        0x0004 00004 (atomic.go:5)       PCDATA  $0, $-2
        0x0004 00004 (atomic.go:5)       JLS     49
        0x0006 00006 (atomic.go:5)       PCDATA  $0, $-1
        0x0006 00006 (atomic.go:5)       PUSHQ   BP
        0x0007 00007 (atomic.go:5)       MOVQ    SP, BP
        0x000a 00010 (atomic.go:5)       SUBQ    $16, SP
        0x000e 00014 (atomic.go:5)       FUNCDATA        $0, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
        0x000e 00014 (atomic.go:5)       FUNCDATA        $1, gclocals·g2BeySu+wFnoycgXfElmcg==(SB)
        0x000e 00014 (atomic.go:6)       LEAQ    type:int64(SB), AX
        0x0015 00021 (atomic.go:6)       PCDATA  $1, $0
        0x0015 00021 (atomic.go:6)       CALL    runtime.newobject(SB)      
        0x001a 00026 (atomic.go:6)       MOVQ    $1, (AX)
        0x0021 00033 (atomic.go:7)       MOVL    $1, CX
        0x0026 00038 (atomic.go:7)       LOCK
        0x0027 00039 (atomic.go:7)       XADDQ   CX, (AX)
        0x002b 00043 (atomic.go:8)       ADDQ    $16, SP
        0x002f 00047 (atomic.go:8)       POPQ    BP
        0x0030 00048 (atomic.go:8)       RET
        0x0031 00049 (atomic.go:8)       NOP
        0x0031 00049 (atomic.go:5)       PCDATA  $1, $-1
        0x0031 00049 (atomic.go:5)       PCDATA  $0, $-2
        0x0031 00049 (atomic.go:5)       CALL    runtime.morestack_noctxt(SB
```

7번 라인을 확인하면 예상한 결과와 유사하게 어셈블리를 사용하고 있다는것을 확인할 수 있습니다.

* MOV.. : 데이터 이동
* LOCK : 락킹
* ADD : 덧셈연산
* 참고 자료
  * https://go.dev/doc/asm



위의 자료를 통해 단일 메모리 주소 하나를 락을 통해 제어하는 것을 확인했다면, 이것이 동시성이 필요한 상황에서 경쟁 상태를 잘 피하는지, 이 시간이 얼마나 걸리는지 확인이 필요한데요.

아래 책의 예제를 실행해 보겠습니다.

<예제>

```
package main  
  
import (  
    "fmt"  
    "log"    "sync"    "sync/atomic"    "time")  
  
type atomCounter struct {  
    value int64  
}  
  
func (c *atomCounter) AddWithLock(i int64) {  
    atomic.AddInt64(&c.value, i)  
}  
  
func (c *atomCounter) AddWithNonLock(i int64) {  
    c.value += i  
}  
  
func (c *atomCounter) Value() int64 {  
    return atomic.LoadInt64(&c.value)  
}  
  
func main() {  
    locking()  
    nonLocking()  
}  
  
func locking() {  
    x := 100  
    y := 4  
  
    st := time.Now()  
  
    fmt.Printf("start\n")  
  
    var wg sync.WaitGroup  
    counter := atomCounter{}  
  
    for i := 0; i < x; i++ {  
       wg.Add(1)  
       go func() {  
          defer wg.Done()  
          // =+4  
          for j := 0; j < y; j++ {  
             counter.AddWithLock(1)  
          }  
       }()  
    }  
  
    wg.Wait()  
  
    fmt.Printf("%d ns\n", time.Since(st).Nanoseconds())  
  
    // expected: 4 * 100 = 400  
    log.Println("counter:", counter.Value())  
}  
  
func nonLocking() {  
    x := 100  
    y := 4  
  
    st := time.Now()  
  
    fmt.Printf("start\n")  
    counter := atomCounter{}  
  
    for i := 0; i < x; i++ {  
       for j := 0; j < y; j++ {  
          counter.AddWithLock(1)  
       }  
    }  
  
    fmt.Printf("%d ns\n", time.Since(st).Nanoseconds())  
  
    // expected: 4 * 100 = 400  
    log.Println("counter:", counter.Value())  
}

```

<실행 결과>

```
// 락킹을 건 결과
start
510200 ns
2024/04/20 05:21:21 counter: 400


// 락킹을 걸지 않은 결과
start
0 ns
2024/04/20 05:21:21 counter: 400

```

예상 한대로. 400이 정상적으로 저장 된 것을 확인할 수 있습니다. 락을 걸지 않은 것에 비해 굉장히 오랜 시간이 걸린 것을 확인할 수 있습니다.

하나의 메모리를 사용해야 한다면, 경쟁 상태를 피하는 것이 성능적으로는 더 훌륭하다는 사실을 알 수 있었네요.

