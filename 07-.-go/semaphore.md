# semaphore 패키지

세마포어는 공유 자원의 접근을 제한하고 제어하는 자료구조입니다. 타 언어에서는 스레드끼리의 경합을 제어하는데 사용되지만, 이번에 소개할 패키지는 고루틴끼리 경합을 제어하는데 사용됩니다.

<설치>

```sh
go get golang.org/x/sync/semaphore
```

<소스>

```go
var Workers = 4  
var sem = semaphore.NewWeighted(int64(Workers))   
  
func worker(n int) int {  
    square := n * n  
    time.Sleep(time.Second)  
    return square  
}  
  
func main() {  
  
    nJobs := 10  
  
    var results = make([]int, nJobs)  
  
    ctx := context.TODO()  
  
    for i := range results {  
       err := sem.Acquire(ctx, 1)  
       if err != nil {  
          fmt.Println("Cannot acquire semaphore:", err)  
          break  
       }  
  
       go func(i int) {  
          log.Println("Starting job", i)  
          defer sem.Release(1)  
          temp := worker(i)  
          results[i] = temp  
       }(i)  
    }  
  
    if err := sem.Acquire(ctx, int64(Workers)); err != nil {  
       fmt.Println(err)  
    }  
  
    for k, v := range results {  
       fmt.Println(k, "->", v)  
    }  
}
```

<출력>

```

2024/05/03 17:11:40 Starting job 0
2024/05/03 17:11:40 Starting job 3
2024/05/03 17:11:40 Starting job 1
2024/05/03 17:11:40 Starting job 2
2024/05/03 17:11:41 Starting job 4
2024/05/03 17:11:41 Starting job 5
2024/05/03 17:11:41 Starting job 6
2024/05/03 17:11:41 Starting job 7
2024/05/03 17:11:42 Starting job 8
2024/05/03 17:11:42 Starting job 9

0 -> 0
1 -> 1
2 -> 4
3 -> 9
4 -> 16
5 -> 25
6 -> 36
7 -> 49
8 -> 64
9 -> 81
```

위의 소스코드를 실행 해 보면, 4개까지 실행이 되도록 쉽게 제약을 걸 수 있습니다.
