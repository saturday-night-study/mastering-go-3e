# 모듈

## Go 모듈

Go 모듈은 버전을 표시한 패키지라고 표현되고 있습니다. Go 모듈은 버전 기입을 할 때 `시맨틱 버저닝(semantic versioning)`을 채택해서 사용하고 있습니다.

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="349"><figcaption></figcaption></figure>

시멘틱 버저닝은 3자리 숫자로 구성되어 있습니다. 각각 `메이저(Major)`, `마이너(Minor)`, `패치(Patch)` 버전으로 구성되어 있습니다.

그 외 [캘린더 버저닝(Calandar Versioning)](https://calver.org/), 시퀀셜 버저닝(Sequential Versioning), [롤링 릴리즈(Rolling Release)](https://ko.wikipedia.org/wiki/%EB%A1%A4%EB%A7%81\_%EB%A6%B4%EB%A6%AC%EC%8A%A4), [빌드 번호(Build Numbers)](https://brunch.co.kr/@joonwonlee/19) 등 [다양한 방식의 버전 관리 방법이 있습니다.](#user-content-fn-1)[^1]

**Go 모듈**에서는 **시멘틱 버저닝 구조에서 v를 붙여 사용**되며, 예를 들자면 v1.0.5, v.2.0.21 과 같은 형태로 사용할 수 있습니다.

{% hint style="info" %}
Go 모듈은 1.11 버전에서 도입되었고 1.13 버전에서 기본 기능으로 채택 됨
{% endhint %}

## Go 모듈 구성 방안

### 좋은 Go 모듈 개발 및  배포

Go 모듈을 작성하고 발행할 때에는 아래 6가지 팁을 통해 본인의 모듈이 진짜 **"모듈"**로써 가치가 있는지에 대해서 꼼꼼히 설계하고 작성 해야합니다.

1. **모듈 개발 및 발행 워크플로**\
   시간에 따라 모듈을 개발하고 새로운 버전으로 발행하는 과정이 반드시 만들어져야함
2. **설계 및 개발 관행**\
   사용자가 모듈을 이해하고 안정적으로 새 버전으로 업그레이드할 수 있도록 해야함
3. **분산 발행 시스템**\
   &#x20;저장소를 통해 모듈을 사용할 수 있도록 제공하고 버전 번호와 함께 발행해야함
4. **패키지 검색 엔진 및 문서 브라우저**\
   개발자가 모듈을 찾을 수 있는 플랫폼([pkg.go.dev](https://pkg.go.dev))의 활용
5. **모듈 버전 관리의 필요성**\
   안정성과 호환성을 보장할 수 있는 모듈 버전 관리가 필요함
6. **의존성 관리를 위한 Go 도구의 사용**\
   다른 개발자가 모듈의 소스를 관리하고 업그레이드하는 데 도움을 주는 도구(go mod)를 활용한 [의존성 관리](https://go.dev/doc/modules/managing-dependencies) 필요&#x20;

#### 참고 내용&#x20;

_"Developing and publishing modules"_라는 Go 언어 공식 블로그의 아티클을 참고하여 작성하였습니다.

{% embed url="https://go.dev/doc/modules/developing" %}
Developing and publishing modules, Official Golang Blog
{% endembed %}

### Go 모듈 작성 방법

Go 모듈은 `go mod` 명령어를 통해서 관리 됩니다. 아래 5가지 `go mod`의 기능을 통해서 Go 모듈을 직접 관리할 수 있습니다.

#### 새로운 Go 모듈 생성하기

```
// 폴더 구조 

example.com
 ㄴ hello
    ㄴ hello_test.go
    ㄴ hello.go
```

```go
// hello.go
package hello

func Hello() string {
    return "Hello, Go Mod !"
} 
```

```go
// hello_test.go
package hello

import "testing"

func TestHello(t *testing T) {
    want := "Hello, Go Mod !"
    if got := Hello(); got != want {
        t.Errorf("Hello() = %q, want %q", got, want)
    }
} 
```

위 새로운 2가지 파일을 구성한 뒤에 먼저 `go`의 `test` 명령어를 통해서 `test`가 정상적으로 통과하는지 확인해야 합니다. (반드시 go 모듈은 테스팅이 통과하고 출시  해야함)

```go
// go test 결과 화면
$ go test
PASS
ok      _/home/gopher/hello 0.020s
```

만약 정상적으로 test가 성공했다면, init 명령어를 통해 새로운 모듈을 생성해줍니다.

```go
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello

$ go test
PASS
ok      example.com/hello   0.020s
```

이후 모듈을 생성하고 실행해보면 실제 실행 경로가 모듈의 경로로 변경 됨을 확인할 수 있고 해당 경로에 `go.mod` 파일이 생성됩니다.

```
// 신규 모듈 구성 이후 폴더 구조

example.com
 ㄴ hello
    ㄴ hello_test.go
    ㄴ hello.go
    ㄴ go.mod
```

#### 의존성 추가하기

만약 새로운 의존성을 추가하고 싶다면, 실제 소스코드에서 새로운 패키지를 import하고 `test` 명령어를 입력해주기만 하면 됩니다.

해당 파트에서는 예시로 `rsc.io/quote` 모듈을 추가해보겠습니다.

```go
// hello.go
package hello

import "rsc.io/quote"

func Hello() string {
    return quote.Hello()
}
```

이후 `go test` 명령어를 입력하면 아래와 같이 `rsc.io/quote` 모듈이 자동으로 의존성 추가가 됩니다.

```go
$ go test
go: finding rsc.io/quote v1.5.2
go: downloading rsc.io/quote v1.5.2
go: extracting rsc.io/quote v1.5.2
go: finding rsc.io/sampler v1.3.0
go: finding golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: downloading rsc.io/sampler v1.3.0
go: extracting rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: extracting golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
PASS
ok      example.com/hello   0.023s
```

해당 모듈 의존성이 정상적으로 추가 됨을 확인하려면 `go list -m all` 명령어를 통해 확인할 수 있습니다. 또한 의존성이 추가가 되면 `go.sum` 파일이 자동으로 생성되며, 해당 파일을 확인해 의존성을 확인할 수 있습니다.

```go
// go list 명령어를 통한 의존성 확인
$ go list -m all
example.com/hello
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0

// go.sum 파일 확인
$ cat go.sum
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZO...
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:Nq...
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3...
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPX...
rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/Q...
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9...
```

#### 의존성 업데이트하기

의존성 업데이트를 하기 위해서는 그냥  go get 명령어만 호출하면 자동으로 업데이트가 됩니다.

```go
$ go get golang.org/x/text
go: finding golang.org/x/text v0.3.0
go: downloading golang.org/x/text v0.3.0
go: extracting golang.org/x/text v0.3.0

$ go test
PASS
ok      example.com/hello   0.013s
```

이후 `go.mod` 파일을 확인해 보면 `require` 부분에 버전과 함께 의존성 모듈이 기록 되어 있음을 확인할 수 있습니다.

```go
// go 모듈의 의존성 모듈 확인
$ go list -m all
example.com/hello
golang.org/x/text v0.3.0
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0

// go.mod 확인
$ cat go.mod
module example.com/hello

go 1.22

require (
    golang.org/x/text v0.3.0 // indirect
    rsc.io/quote v1.5.2
)
```

그런데 모듈을 최신으로 받다 보면 특정 버전의 모듈을 사용하고 싶을 때가 있습니다. 그럴 때는 2번의 과정만 거치면 손쉽게 진행할 수 있습니다.

1. 의존성 모듈의 버전 확인하기
2. `go get` 명령어에 **@**를 사용해 특정 버전으로 업데이트하기

```go
// rsc.io/sampler 버전 확인
$ go list -m -versions rsc.io/sampler
rsc.io/sampler v1.0.0 v1.2.0 v1.2.1 v1.3.0 v1.3.1 v1.99.99

// 1.3.1 버전으로 get
$ go get rsc.io/sampler@v1.3.1
go: finding rsc.io/sampler v1.3.1
go: downloading rsc.io/sampler v1.3.1
go: extracting rsc.io/sampler v1.3.1

// go test 진행하여 정상 동작 확인
$ go test
PASS
ok      example.com/hello   0.022s
```

#### 새로운 메이저 버전을 의존성 추가하기

새로운 메이저 버전의 의존성 모듈을 추가하고 싶다면 매우 간단하게 import에서 메이저 버전을 명시해주기만 하면 됩니다.

```go
// hello.go
package hello

import (
    "rsc.io/quote"
    quoteV3 "rsc.io/quote/v3"
)

func Hello() string {
    return quote.Hello()
}

func Proverb() string {
    return quoteV3.Concurrency()
}
```

```go
// hello_test.go
func TestProverb(t *testing.T) {
    want := "Concurrency is not parallelism."
    if got := Proverb(); got != want {
        t.Errorf("Proverb() = %q, want %q", got, want)
    }
}
```

위와 같이 소스 코드와  테스트 코드를 작성 한 뒤 `go test`를 입력하고 의존성 목록을 조회하면 새로운 메이저 버전의 모듈이 추가 됨을 확인할 수 있습니다.

#### 의존성 관리하기

사용하지 않거나 새로운 의존성 추가 및 업데이트는 `go mod tidy` 명령어를 통해 관리할 수 있습니다.

```go
// go 모듈 목록 조회
$ go list -m all
example.com/hello
golang.org/x/text v0.3.0
rsc.io/quote v1.5.2
rsc.io/quote/v3 v3.1.0
rsc.io/sampler v1.3.1

// go.mod 파일의 확인
$ cat go.mod
module example.com/hello

go 1.12

require (
    golang.org/x/text v0.3.0 // indirect
    rsc.io/quote v1.5.2
    rsc.io/quote/v3 v3.0.0
    rsc.io/sampler v1.3.1 // indirect
)
```

이후 `rsc.io/quote` 모듈을 `import`에서 제거하고 `go mod tidy`를 입력하여 의존성 패키지를 업데이트 하면 아래와 같은 결과 값을 얻을 수 있습니다.

```go
// go mod tidy 명령어를 통해 의존성 업데이트
$ go mod tidy

// go 모듈 목록 조회 (사용하지 않는 의존성이 삭제 됨을 확인)
$ go list -m all
example.com/hello
golang.org/x/text v0.3.0
rsc.io/quote/v3 v3.1.0
rsc.io/sampler v1.3.1

// go.mod 파일의 확인
$ cat go.mod
module example.com/hello

go 1.12

require (
    golang.org/x/text v0.3.0 // indirect
    rsc.io/quote/v3 v3.1.0
    rsc.io/sampler v1.3.1 // indirect
)

$ go test
PASS
ok      example.com/hello   0.020s
```

#### 참고 내용&#x20;

_"Using Go Modules"_ 라는 Go 언어 공식 블로그의 아티클을 참고하여 작성하였습니다.

{% embed url="https://blog.golang.org/using-go-modules" %}
Using Go Modules, Official Golang Blog
{% endembed %}

[^1]: 각 프로그램, 패키지, 라이브러리 등에 따라서 고유한 체계가 있으며 약간의 차이가 존재
