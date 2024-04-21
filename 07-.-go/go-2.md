# 클로저 변수와 go 구문

## 클로저 변수와 go 구문



클로저는 단순함을 추구하는 Go 치고 왜 만들어졌는지 의문이 드는 기능이긴 합니다. 그런 의문은 잠시 접어두고, 클로저 변수란 무엇인지 알아봅시다.

{% hint style="info" %}
go 에서 **클로저 변수**란 함수 안에 이름이 없는 익명함수가 부모 함수(자신이 정의된 환경)의 지역변수를 사용할 때 이 지역변수를 클로저 변수라고 합니다.
{% endhint %}

<예제 1>

```
func main() {  
    for i := 0; i < 20; i++ {  
       go func() {  
          fmt.Print(i, " ")  
       }()  
    }  
  
    time.Sleep(1 * time.Second)  
    fmt.Println()  
}
```

<실행 결과 2>

```
0 1 6 3 4 5 8 7 9 10 2 14 11 12 13 16 15 17 18 19
```

<예제 2>

```
func main() {  
    for i := 0; i < 20; i++ {  
       i := i  
       go func() {  
          fmt.Print(i, " ")  
       }()  
    }  
  
    time.Sleep(1 * time.Second)  
    fmt.Println()  
}
```

<실행 결과 2>

```
1 3 0 4 8 5 6 7 11 9 10 12 13 14 2 15 16 17 18 19
```

이후 CGO를 설치하고 예제 1, 예제 2에 대해서 경쟁 상태를 체크해보겠습니다.

\<Closer.bat>

```
set CGO_ENABLED=1  
go run -race main.go
```

```
..\mastering-go-example\2024-04-20 closer\cmd>set CGO_ENABLED=1 

..\mastering-go-example\2024-04-20 closer\cmd>go run -race main.go 
0 16 1 5 6 8 9 10 7 2 12 11 13 4 14 15 17 3 18 19 
```



두 예제 모두 위의 결과와 같이, Go 1.22.1 버전에서는 클로저 변수에 대해 자동으로 고루틴에 사용할 변수의 인스턴스를 새로 생성하도록 설정 되었음을 알 수 있습니다.

