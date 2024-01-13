# 정규표현식과 패턴 매칭

해당 파트는 `정규표현식(Regular Expression)`과 `패턴 매칭(Pattern Matching)`에 대해서 설명하고 예시자료를 통해 사용하는 방법을 알아본다.

## Go의 정규표현식

Go 언어에서는 regexp 패키지를 통해서 `정규표현식(Regular Expression)`을 정의하고 `패턴 매칭(Pattern Matching)`을 수행하며, `regexp.MustCompile()` 함수를 통해서 `정규표현식(Regular Expression)`을 생성하고 `Match()` 함수를 통해 패턴 매칭 여부를 판별한다.

`regexp.MustCompile()` 함수는 `패턴 매칭(Pattern Matching)`을 할 수 있도록 `regexp.Regexp` 변수를 반환하게 되어있고 해당 변수 값을 활용될 수 있게 구현되어 있다.

{% hint style="info" %}
이점을 활용하여 커스텀한 `패턴 매칭(Pattern Matching)`함수를 만들어 재 사용이 가능한 함수 구조를 만들 수 있다.
{% endhint %}

### 정규표현식의 표현식

<table><thead><tr><th width="118">표현식</th><th>설명</th></tr></thead><tbody><tr><td>.</td><td>모든 문자를 매칭</td></tr><tr><td>*</td><td>여러 번 사용 됨 (단독 사용 불가)</td></tr><tr><td>?</td><td>0회 또는 1회 (단독 사용 불가)</td></tr><tr><td>+</td><td>1회 이상 (단독 사용 불가)</td></tr><tr><td>^</td><td>줄의 시작과 끝</td></tr><tr><td>[]</td><td>문자의 그룹화</td></tr><tr><td>[A-Z]</td><td>대문자 A에서 대문자 Z까지의 모든 문자</td></tr><tr><td>\d</td><td>0과 9사이의 모든 숫자</td></tr><tr><td>\D</td><td>숫자가 아닌 문자</td></tr><tr><td>\w</td><td>모든 단어 문자: [0-9A-Za-z_]</td></tr><tr><td>\W</td><td>단어가 아닌 모든 문자</td></tr><tr><td>\s</td><td>공백 문자</td></tr><tr><td>\S</td><td>공백이 아닌 문자</td></tr></tbody></table>

### 이름과 성 매칭 예제

정규식의 경우에는 예제를 통해 확인하는 것이 가장 쉽기 때문에 바로 예제를 통해서 사용  방법에 대해서 알아본다.

```go
// from: https://github.com/mactsouk/mastering-Go-3rd/blob/main/ch03/nameSurRE.go
package main

import (
	"fmt"
	"os"
	"regexp"
)

func matchNameSur(s string) bool {
	t := []byte(s)
	re := regexp.MustCompile(`^[A-Z][a-z]*$`)
	return re.Match(t)
}

func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Usage: <utility> string.")
		return
	}

	s := arguments[1]
	ret := matchNameSur(s)
	fmt.Println(ret)
}
```

해당 정규표현식을 해석해보자면, 대문자(\[A-Z])로 시작하고 그  다음부터는 어떠한 길이든 상관없이소문자(\[a-z]\*)로 된 문자열을 매칭한다.

```sh
// Some code
go run nameSurRE.go Z            // 시작이 대문자이기 때문에 허용
true

go run nameSurRE.go ZA           // 시작은 대문자이나 두 번째도 대문자여서 허용 x
false

go run nameSurRE.go Mihalis      // 시작 후 두 번째 부터 소문자이기에 허용
true

go run nameSurRE.go Jo+          // 시작 후 영문자가 아닌 +가 왔기에 허용 x 
false
```

### 정수 매칭 예제

```go
// from: https://github.com/mactsouk/mastering-Go-3rd/blob/main/ch03/intRE.go

package main

import (
	"fmt"
	"os"
	"regexp"
)

func matchInt(s string) bool {
	t := []byte(s)
	re := regexp.MustCompile(`^[-+]?\d+$`)
	return re.Match(t)
}

func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Usage: <utility> string.")
		return
	}

	s := arguments[1]
	ret := matchInt(s)
	fmt.Println(ret)
}
```

### 레코드 필드 매칭 예제

```go
// from: https://github.com/mactsouk/mastering-Go-3rd/blob/main/ch03/fieldsRE.go

package main

import (
	"fmt"
	"os"
	"regexp"
	"strings"
)

func matchNameSur(s string) bool {
	t := []byte(s)
	re := regexp.MustCompile(`^[A-Z][a-z]*$`)
	return re.Match(t)
}

func matchTel(s string) bool {
	t := []byte(s)
	re := regexp.MustCompile(`\d+$`)
	return re.Match(t)
}

func matchRecord(s string) bool {
	fields := strings.Split(s, ",")
	if len(fields) != 3 {
		return false
	}

	if !matchNameSur(fields[0]) {
		return false
	}

	if !matchNameSur(fields[1]) {
		return false
	}

	return matchTel(fields[2])
}

func main() {
	arguments := os.Args
	if len(arguments) == 1 {
		fmt.Println("Usage: <utility> record.")
		return
	}

	s := arguments[1]
	err := matchRecord(s)
	fmt.Println(err)
}
```
