# 파일 입출력

#### io.Reader와 io.Writer 인터페이스

`io.Reader`는 파일을 읽게 해줍니다.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

`Reader` 인터페이스를 만족하려면 `Read()` 메서드를 구현해야 합니다. `Read()` 메서드는 읽어들인 값을 저장할 바이트 슬라이스를 매개변수로 받고, 읽은 바이트 수와 읽을때 발생한 에러를 반환합니다.

`io.Writer`는 파일을 쓸 수 있게 해줍니다.

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

`Writer` 인터페이스를 만족하려면 `Write()` 메서드를 구현해야 합니다. `Write()` 메서드는 쓰고자하는 값을 담은 바이트 슬라이스를 매개변수로 받고, 쓴 바이트 수와 쓸때 발생한 에러를 반환합니다.

#### io.Reader와 io.Writer의 사용과 오용

{% embed url="https://gist.github.com/kineo2k/c90902acb57976933a49a9d76c5be7ff" %}

#### 버퍼를 이용한 파일 입출력과 버퍼를 이용하지 않는 파일 입출력

**버퍼(Buffer)**를 이용한 파일 입력과 출력은 데이터를 읽거나 쓰기 전에 잠시 버퍼에 저장합니다. 이는 파일이나 소켓에 데이터를 읽거나 쓸때 시스템 콜 호출 수를 줄여서 **성능을 크게 개선**할 수 있습니다. `bufio` 패키지는 버퍼를 사용한 입출력을 지원합니다. 내부적으로는 `bufio.Reader`와 `bufio.Writer` 인터페이스를 구현하는데 이 인터페이스들은 `io.Reader`와 `io.Writer` 객체를 감싼 인터페이스입니다.
