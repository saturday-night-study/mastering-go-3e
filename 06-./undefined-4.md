# 파일 쓰기

#### fmt.Fprintf()

{% embed url="https://gist.github.com/kineo2k/fa89ed674dc7a494c46f1fac7ef459cb" %}

#### File.WriteString()

{% embed url="https://gist.github.com/kineo2k/2fe1614f5923a2352664b6b8fb1a9568" %}

#### bufio.NewWriter().WriteString()

{% embed url="https://gist.github.com/kineo2k/6e307bb15da325f264de6f37d0afd695" %}

#### io.WriteString()

{% embed url="https://gist.github.com/kineo2k/bf7518174ced5fdf73f52d63dc2a6922" %}

#### os.OpenFile()

`os.OpenFile()` 함수는 파일을 열때 사용 가능한 몇 가지 옵션과 파일 권한 설정 기능을 제공합니다.

```go
const (
	// 아래 플레그는 세 가지 중 하나만 설정해야 합니다.
	O_RDONLY int = syscall.O_RDONLY // 읽기 전용으로 열기
	O_WRONLY int = syscall.O_WRONLY // 쓰기 전용으로 열기
	O_RDWR   int = syscall.O_RDWR   // 읽기/쓰기 모두 가능하게 열기
	
	// 아래 플래그는 중복으로 적용이됩니다.
	O_APPEND int = syscall.O_APPEND // 파일이 존재할 경우 파일 내용을 유지하고 끝에서부터 쓰기
	O_CREATE int = syscall.O_CREAT  // 파일이 존재하지 않을 경우 파일을 생성
	O_EXCL   int = syscall.O_EXCL   // 파일이 존재할 경우 파일 열기 실패 처리
	O_SYNC   int = syscall.O_SYNC   // 동기식 I/O로 파일을 열기
	O_TRUNC  int = syscall.O_TRUNC  // 파일이 존재할 경우 파일 내용을 지우고 열기
)
```

`os.Open()`과 `os.Create()` 함수는 각각 아래와 같이 정의되어 있습니다.

```go
func Open(name string) (*File, error) {
	return OpenFile(name, O_RDONLY, 0)
}

func Create(name string) (*File, error) {
	return OpenFile(name, O_RDWR|O_CREATE|O_TRUNC, 0666)
}
```

쓰기 전용 모드에서 이어쓰기로 파일을 생성하는 예제입니다.

{% embed url="https://gist.github.com/kineo2k/8fa5383c196d0d7e9d17d10238d53850" %}
